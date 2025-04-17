---
layout: post
title: "Learn What You Want to Unlearn: Tấn công Đảo ngược Bỏ học"
date: 2025-04-16
categories: jekyll update
---
## 1. Giới thiệu
Trong kỷ nguyên số, dữ liệu cá nhân ngày càng được thu thập và xử lý nhiều hơn bao giờ hết. Đi kèm với đó là những lo ngại về quyền riêng tư và nhu cầu kiểm soát thông tin cá nhân. Các quy định như GDPR ở Châu Âu và CCPA ở California đã ra đời, trao cho người dùng "Quyền được Lãng quên" (Right to be Forgotten) - quyền yêu cầu các tổ chức xóa dữ liệu của họ.   

Việc xóa dữ liệu khỏi cơ sở dữ liệu truyền thống tương đối đơn giản. Tuy nhiên, với các mô hình Học máy (Machine Learning), đặc biệt là Mạng Nơ-ron Sâu (Deep Neural Networks - DNNs), việc này phức tạp hơn nhiều do bản chất phức tạp và ngẫu nhiên của quá trình huấn luyện. Để giải quyết thách thức này, kỹ thuật **Machine Unlearning** (nếu bạn không biết gì về Machine Unlearning thì có thể đọc blog trước tại <a href="https://trunkg0d.github.io/Hoang-Trung-Le-Pham/jekyll/update/2025/03/31/machine-unlearning.html">đây</a>) đã ra đời, nhằm mục đích loại bỏ ảnh hưởng của một số dữ liệu huấn luyện cụ thể khỏi mô hình đã được huấn luyện.   

Nghe có vẻ đây là một giải pháp hoàn hảo cho quyền riêng tư? Nhưng liệu quá trình "quên" này có thực sự an toàn? Bài báo **"Learn What You Want to Unlearn: Unlearning Inversion Attacks against Machine Unlearning"**  đã thực hiện một nghiên cứu tiên phong, chỉ ra rằng chính quá trình xóa học lại có thể mở ra một bề mặt tấn công mới, làm lộ thông tin nhạy cảm của chính dữ liệu đáng lẽ phải được "quên" đi. Bài viết này sẽ giải mã ý tưởng cốt lõi và các phương pháp tấn công được đề xuất trong nghiên cứu quan trọng này.

## 2. Mô hình mối đe doạ (Threat Model)
### 2.1. Bối cảnh và Kẻ tấn công
Để hiểu rõ lỗ hổng mà bài báo khai thác, trước tiên chúng ta cần xác định mô hình đe dọa (threat model), bao gồm bối cảnh hoạt động, mục tiêu của kẻ tấn công, và khả năng của chúng.

**Bối cảnh:** Cuộc tấn công thường diễn ra trong môi trường **Học máy như một Dịch vụ (MLaaS - Machine Learning as a Service)**. Trong kịch bản này:
- **Nhà phát triển mô hình (Model Developer):** Huấn luyện mô hình $(M)$ trên dữ liệu riêng tư (bao gồm cả dữ liệu $D_u$ sau này cần xoá) và tải mô hình lên server.
- **Nhà cung cấp dịch vụ MLaaS (Server):** Triển khai mô hình $M$ và cung cấp API cho người dùng. Khi có yêu cầu xoá dữ liệu $D_u$, nhà phát triển tạo ra mô hình đã unlearn $(M_u)$ bằng một phương pháp unlearning nào đó, rồi tải $M_u$ lên server để thay thế $M$.
- **Người dùng cuối (User):** Tương tác với mô hình (ban đầu là $M$, sau là $M_u$) thông qua API do server cung cấp.

<div align="center">
    <figure>
        <img src="{{site.baseurl}}/images/machine-unlearning/threat-model.png" alt="MU and Inversion Attacks" width="700">
        <figcaption><b>Tổng quan về unlearning inversion attacks trong môi trường MLaaS.</b></figcaption>
    </figure>
</div>

**Kẻ tấn công và Khả năng:** Bài báo xem xét hai loại kẻ tấn công chính:
- **Server MLaaS (Honest-but-Curious):**
  - **Giả định:** Server hoạt động đúng chức năng nhưng có thể tò mò và cố gắng trích xuất thông tin từ các mô hình mà nó quản lý.
  - **Khả năng:** Có quyền truy cập **White-box** vào cả hai mô hình $M$ và $M_u$ (do nhà phát triển tải lên). Nghĩa là server có thể thấy và phân tích các tham số nội bộ $(\theta \text{ và } \theta_u)$
  - **Hạn chế:** Giả định server không biết chính xác thuật toán unlearning nào đã được nhà phát triển sử dụng và cũng không có quyền truy cập vào tập dữ liệu huấn luyện gốc đầy đủ

- **Người dùng cuối (Malicious User):**
  - **Giả định:** Một người dùng thông thường của dịch vụ MLaaS nhưng có ý đồ xấu.
  - **Khả năng:** Chỉ có quyền truy cập *Black-box* thông qua API được cung cấp. Người dùng có thể gửi truy vấn đến mô hình và nhận lại kết quả dự đoán. Quan trọng là, người dùng có thể tương tác với mô hình $M$ trước khi nó bị thay thế và tương tác với mô hình $M_u$ sau khi nó được cập nhật.
  - **Hạn chế:** Không thể thấy tham số nội bộ của mô hình

### 2.2. Mục tiêu Tấn công và Tiêu chí Thành công (Attack Goals and Success Criteria)
Mục tiêu tổng quát của kẻ tấn công là trích xuất thông tin nhạy cảm về dữ liệu đã bị xóa $(D_u)$, làm vô hiệu hóa mục đích bảo vệ quyền riêng tư của việc xóa học. Mục tiêu cụ thể và cách đánh giá thành công phụ thuộc vào loại kẻ tấn công và quyền truy cập của họ:
- **Đối với Kẻ tấn công White-box (Server):**
  - **Mục tiêu:** Khôi phục Đặc trưng - Feature Recovery. Cố gắng tái tạo lại một mẫu dữ liệu $x'$ sao cho nó càng giống với mẫu dữ liệu gốc đã bị xoá $x^\ast$ càng tốt về mặt hình ảnh hoặc cấu trúc.
  - **Tiêu chí thành công:**
    - Đạt điểm cao trên các chỉ số như MSE, LPIPS thấp, PSNR cao
    - Quan trọng hơn, tạo ra được $x'$ mà **con người có thể nhận dạng được đối tượng hoặc thông tin nhạy cảm** chứa trong $x^\ast$ ban đầu, ngay cả khi $x'$ có nhiễu hoặc không hoàn hảo.
- **Đối với Kẻ tấn công Black-box (User):**
  - **Mục tiêu:** Suy luận Nhãn (Label Inference). Xác định chính xác các label (class) $C_u$ của dữ liệu đã bị xoá khỏi mô hình.
  - **Tiêu chí thành công:**
    - Các label được suy luận ra phải trùng khớp với các label thực tế của dữ liệu $D_u$ đã bị xoá.
    - Thành công của cuộc tấn công được đo bằng độ chính xác (accuracy) của việc suy luận nhãn.

### 2.3. Lỗ hổng cốt lõi
Điểm yếu cơ bản nằm ở chỗ kẻ tấn công (dù là server hay người dùng) có khả năng tiếp cận và so sánh trạng thái hoặc hành vi của mô hình trước $(M)$ và sau $(M_u)$ khi quá trình unlearning diễn ra. Chính sự khác biệt phát sinh từ việc loại bỏ dữ liệu $D_u$ là bề mặt tấn công, cho phép kẻ tấn công cô lập và suy luận thông tin về dữ liệu đã bị xoá.

## 3. Phương pháp Tấn công: Unlearning Inversion Attacks
Bài báo giới thiệu **Unlearning Inversion Attacks** như một phương pháp mới để chứng minh và khai thác lỗ hổng bảo mật trong quá trình unlearning. Ý tưởng trung tâm là tận dụng thông tin có được từ việc so sánh hai phiên bản mô hình: mô hình gốc $M$ (với tham số $\theta$) và mô hình đã xoá học $M_u$ (với tham số $\theta_u$). Sự khác biệt giữa chúng ẩn chứa "dấu vết" của dữ liệu đã bị xoá $(D_u)$. Phương pháp tấn công được chia thành hai kịch bản chính, tuỳ thuộc vào quyền truy cập của kẻ tấn công:

### 3.1. Khôi phục Đặc trưng (Feature Recovery - White-box)
Kịch bản này giả định kẻ tấn công là **Server MLaaS** (hoặc bất kỳ ai có quyền truy cập **white-box**), nghĩa là họ có thể xem và thao tác trực tiếp trên các tham số $\theta$ và $\theta_u$ của cả hai mô hình. Mục tiêu là tái tạo lại đặc trưng (ví dụ: hình ảnh) của một mẫu dữ liệu $x^\ast \in D_u$ đã bị xoá. Quá trình gồm hai bước chính:

- **Bước 1:** Ước lượng Gradient của Dữ liệu đã xoá $(\nabla^\ast)$

  - **Ý tưởng:** Trong quá trình huấn luyện ML thông thường, tham số mô hình được cập nhật dựa trên gradient của hàm mất mát tính trên dữ liệu huấn luyện. Công thức cập nhật cơ bản (như Stochastic Gradient Descent - SGD) là:

    $$\theta_{k+1} = \theta_k - \alpha \nabla_\theta L (\theta_k)$$

    Trong đó $\theta_k$ là tham số ở bước $k$, $\alpha$ là tốc độ học, và $\nabla_\theta L (\theta_k)$ là gradient tính trên một batch dữ liệu. Điều này có nghĩa là sự thay đổi tham số $(\theta_{k+1} - \theta_k)$ xấp xỉ với thông tin gradient (âm) của batch dữ liệu đó

  - **Áp dụng vào Unlearning:** Tương tự, sự khác biệt giữa mô hình gốc $\theta$ (đã "học" $D_u$) và mô hình đã unlearn $\theta_u$ (đã "quên" $D_u$) phải chứa thông tin gradient liên quan đến $D_u$. Kẻ tấn công ước lượng thông tin gradient này bằng cách tính hiệu đơn giản:

    $$\nabla^\ast = \theta - \theta_u$$

  - **Lưu ý:** $\nabla^\ast$ chỉ là một ước lượng cho thông tin gradient của $D_u$. Nó không phải là một gradient chính xác vì kẻ tấn công không biết chi tiết về thuật toán bỏ học cụ thể mà nhà phát triển đã sử dụng (ví dụ: tốc độ học, số epoch, các kỹ thuật xấp xỉ...).

- **Bước 2:** Đảo ngược Gradient Ước lượng (Gradient Inversion)
  - **Thách thức:** Làm thế nào để khôi phục lại dữ liệu gốc $x^\ast$ từ gradient ước lượng $\nabla^\ast$

  - **Giải pháp:** Lấy cảm hứng từ các kỹ thuật tấn công đảo ngược gradient đã được chứng minh là hiệu quả trong Học liên kết (Federated Learning), nơi server có thể khôi phục dữ liệu huấn luyện riêng tư của client từ gradient mà client gửi lên

  - **Điều chỉnh cho Unlearning:** Các kỹ thuật đảo ngược gradient chuẩn thường yêu cầu gradient chính xác và đôi khi cần thêm thông tin (như phân phối dữ liệu huấn luyện). Do kẻ tấn công ở đây chỉ có $\nabla^\ast$ (gradient ước lượng) và không có thông tin phụ trợ, bài báo sử dụng một phương pháp tối ưu hoá tập trung vào việc khớp *hướng* của gradient thay vì độ lớn.

  - **Quy trình tối ưu hoá:**

    1. Khởi tạo một dữ liệu giả lập (dummy input) $x'$ và nhãn giả lập (dummy label) $y'$

    2. Tính gradient $\nabla'$ của hàm mất mát $L$ đối với tham số $\theta$ của mô hình gốc $M$ dựa trên $x'$ và $y'$:

      $$\nabla' = \frac{\partial L (f_\theta (x'), y')}{\partial \theta}$$

    3. Tìm $x'$ tối ưu bằng cách giải bài toán tối ưu sau:

        $$\text{argmin}_{x'} -l(\nabla', \nabla^\ast) + \alpha TV(x')$$
        trong đó:
        
        - $l(\nabla', \nabla^\ast)$ là độ tương đồng cosine (cosine similarity) giữa gradient giả lập $\nabla'$ và gradient ước lượng $\nabla^\ast$:

          $$
          l(\nabla', \nabla^\ast) = \frac{\langle \nabla', \nabla^\ast \rangle}{\| \nabla' \|_2 \| \nabla^\ast \|_2}
          $$

          việc tối ưu hoá để *cực đại hoá* độ tương đồng cosine (tương đương cực tiểu hoá $-l$) giúp đảm bảo $\nabla'$ và $\nabla^\ast$ có cùng hướng, phù hợp với việc ước lượng hướng của $\nabla^\ast$.

        - $TV(x')$ là hàm **Tổng biến thiên (Total Variation)**, một hàm chính quy hoá (regularizer) thường dùng trong xử lý ảnh:

          $$TV(x') = \sum_{i, j \in N} ||c_i - c_j||_1$$

          $N$ là các cặp pixel hàng xóm, $c_i$ và $c_j$ là các giá trị pixel. Hàm này khuyến khích $x'$ được tái tạo mượt mà hơn, giống ảnh tự nhiên và giảm nhiễu.

        - $\alpha$ là một siêu tham số để cân bằng giữa việc khớp hướng gradient và tính mượt của ảnh tái tạo.

    - **Kết quả:** Quá trình tối ưu hoá sẽ tìm ra $x'$ mà khi đưa qua mô hình gốc $M$ sẽ tạo ra gradient $\nabla'$ có hướng gần nhất với $\nabla^\ast$. $x'$ này chính là đặc trưng được khôi phục của dữ liệu đã xoá $x^\ast$. Bài báo cho thấy phương pháp này vẫn hoạt động tốt ngay cả khi nhãn $y^\ast$ không được biết trước (coi $y'$ là một biến cần tối ưu)

### 3.2. Suy luận Nhãn (Label Inference - Black-box)

Kịch bản này giả định kẻ tấn công là Người dùng cuối (End-user) với quyền truy cập **black-box** thông qua API. Họ không thể xem tham số mô hình mà chỉ có thể gửi đầu vào và nhận đầu ra (thường là vector xác suất dự đoán cho các lớp). Mục tiêu là xác định (các) nhãn $C_u$ của dữ liệu $D_u$ đã bị xoá.

Cách thức:

- **Bước 1: Tạo các Mẫu thăm dò (Probing Samples)**

  - **Ý tưởng:** Sự thay đổi trong hành vi dự đoán của mô hình (từ $M$ sang $M_u$) trên cùng một đầu vào có thể tiết lộ thông tin về dữ liệu đã bị xoá. Để khuếch đại và đo lường sự thay đổi này một cách hiệu quả, kẻ tấn công cần tạo ra các mẫu đầu vào đặc biệt.

  - **Quy trình:**

    1. Kẻ tấn công chuẩn bị một tập dữ liệu nhỏ $D_p$ (có thể là dữ liệu ngẫu nhiên hoặc từ một nguồn khác, không nhất thiết cùng phân phối với dữ liệu huấn luyện gốc).

    2. Với mỗi lớp $y_t$ trong bài toán phân loại, kẻ tấn công chọn một mẫu $x \in D_p$ và tối ưu hoá nó thành $x'$ sao cho mô hình gốc $M$ dự đoán $x'$ thuộc lớp $y_t$ với **độ tin cậy (confidence) cực cao** (gần bằng 1)

    3. Việc tối ưu hoá này được thực hiện bằng các kỹ thuật tấn công black-box như **Zeroth Order Optimization (ZOO)**. Về mặt toán học, mục tiêu là tìm $x'$ để cực tiểu hoá hàm:

        $$g(x', y_t) = \max_{i \neq y_t} [Z(x')]_i - [Z(x')]_{y_t}$$

        Trong đó $Z(x')$ là vector logits (đầu ra trước hàm softmax) của mô hình gốc $M$. Việc cực tiểu hoá $g(x', y_t)$ buộc logit của lớp $y_t$ phải lớn hơn đáng kể so với tất cả các lớp khác, dẫn đến xác suất dự đoán cho $y_t$ rất cao.

    4. Kẻ tấn công tạo ra một số lượng nhỏ (ví dụ: $m=10$) mẫu thăm dò như vật cho mỗi lớp $y_t$. Tập hợp các mẫu $x'$ này cùng với vector dự đoán $\hat{Y}$ tương ứng từ mô hình $M$ được lưu lại thành **bộ dữ liệu thăm dò (probing dataset)** $D_p$.

- **Bước 2: Thăm dò Mô hình Đã bỏ học $(M_u)$**

  - Sau khi nhà phát triển thay thế $M$ bằng $M_u$ trên server, kẻ tấn công sử dụng chính bộ dữ liệu thăm dò $D_p$ đã tạo ở Bước 1 để truy vấn mô hình $M_u$

  - Với mỗi mẫu thăm dò $x'_i \in D_p$ (vốn được thiết kế để $M$ dự đoán là lớp $y_t$ với độ tin cậy cao), kẻ tấn công ghi lại vector dự đoán $Y_i = f_{\theta_u} (x'_i)$

- **Bước 3: Tính toán Độ thay đổi Tin cậy và Suy luận Nhãn**

  - **Ý tưởng:** Nếu dữ liệu thuốc lớp $C_\text{unlearn}$ bị xoá đi, mô hình $M_u$ sẽ trở nên "kém chắc chắn" hơn khi dự đoán các mẫu liên quan đến lớp đó. Do đó, khi truy vấn $M_u$ bằng các mẫu thăm dò vốn được $M$ dự đoán là $C_\text{unlearn}$ với độ tin cậy cao, thì độ tin cậy của $M_u$ cho lớp $C_\text{unlearn}$ sẽ giảm xuống đáng kể so với $M$.

  - **Tính toán:** Với mỗi lớp mục tiêu $y_t$ của các mẫu thăm dò, kẻ tấn công tính toán vector chênh lệch trung bình giữa dự đoán của $M$ và $M_u$:

    $$\Delta_p = \frac{1}{m} \sum_{i=1}^m (\hat{Y}_i - Y_i)$$

    Sau đó, trích xuất giá trị chênh lệch tại chỉ số tương ứng với lớp $y_t$:

    $$\beta_{y_t} = \Delta_p[y_t]$$

    Giá trị $\beta_{y_t}$ này thể hiện mức độ thay đổi (tăng hoặc giảm) trung bình về độ tin cậy của mô hình đối với lớp $y_t$ sau khi bỏ học, được đo trên các mẫu thăm dò được thiết kế cho lớp $y_t$.

  - **Suy luận:** Kẻ tấn công tìm lớp $y_t$ có giá trị $\beta_{y_t}$ **âm nhất** (tức là độ tin cậy giảm nhiều nhất). Lớp này được suy luận là nhãn $C_u$ của dữ liệu đã bị xoá:

    $$C_u = \text{argmin}_{y_t} \beta_{y_t}$$

  - **Trường hợp nhiều lớp:** Nếu nghi ngờ nhiều lớp bị xoá cùng lúc (ví dụ $K$ lớp), kẻ tấn công có thể chọn ra $K$ lớp có giá trị $\beta_{y_t}$ âm nhất (top $K$) làm các nhãn bị suy luận.

## 4. Kết quả thực nghiệm của bài báo

## 5. Hậu quả và cách Phòng thủ

## 6. Kết luận

<br>
<hr>
<br>

## Tài liệu tham khảo
- <a id="p2025"></a> **K. P., M. S., Nicolazzo, S., Nocera, A., & P, V. (2025).** "How Secure is Forgetting? Linking Machine Unlearning to Machine Learning Attacks." arXiv preprint arXiv:2503.20257. [https://arxiv.org/abs/2503.20257](https://arxiv.org/abs/2503.20257)

- <a id="xu2024"></a> **Xu, J., Wu, Z., Wang, C., & Jia, X. (2024).** "Machine Unlearning: Solutions and Challenges." IEEE Transactions on Emerging Topics in Computational Intelligence*, 8(3), 2150–2168. DOI: [10.1109/TETCI.2024.3379240](http://dx.doi.org/10.1109/TETCI.2024.3379240)
 
- <a id="unlearning-challenge-2023"></a> **Unlearning Challenge (NeurIPS 2023).** "Starting Kit: Machine Unlearning on CIFAR-10." [Google Colab Notebook](https://colab.research.google.com/github/unlearning-challenge/starting-kit/blob/main/unlearning-CIFAR10.ipynb)

- <a id="wang2024"></a> **Wang, W., Tian, Z., Zhang, C., & Yu, S. (2024).** "Machine Unlearning: A Comprehensive Survey." 
  arXiv preprint arXiv:2405.07406. [https://arxiv.org/abs/2405.07406](https://arxiv.org/abs/2405.07406)

- <a id="hu2024"></a> **Hu, H., Wang, S., Dong, T., & Xue, M. (2024).** "Learn What You Want to Unlearn: Unlearning Inversion Attacks against Machine Unlearning." arXiv preprint arXiv:2404.03233. [https://arxiv.org/abs/2404.03233](https://arxiv.org/abs/2404.03233)

- <a id="graves2020"></a> **Graves, L., Nagisetty, V., & Ganesh, V. (2020).** "Amnesiac Machine Learning." arXiv preprint arXiv:2010.10981. [https://arxiv.org/abs/2010.10981](https://arxiv.org/abs/2010.10981)

- <a id="GAO2024"></a> **Gao, K., Zhu, T., Ye, D., & Zhou, W. (2024).** Defending against gradient inversion attacks in federated learning via statistical machine unlearning. *Knowledge-Based Systems*, *299*, 111983. [https://doi.org/10.1016/j.knosys.2024.111983](https://doi.org/10.1016/j.knosys.2024.111983)