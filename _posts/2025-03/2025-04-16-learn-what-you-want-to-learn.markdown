---
layout: post
title: "Giải mã Learn What You Want to Unlearn: Tấn công Đảo ngược Bỏ học"
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
- **Nhà phát triển mô hình (Model Developer):** Huấn luyện mô hình $(M)$ trên dữ liệu riêng từ (bao gồm cả dữ liệu $D_u$ sau này cần xoá) và tải mô hình lên server.
- **Nhà cung cấp dịch vụ MLaaS (Server):** Triển khai mô hình $M$ và cung cấp API cho người dùng. Khi có yêu cầu xoá dữ liệu $D_u$, nhà phát triển tạo ra mô hình đã unlearn $(M_u)$ bằng một phương pháp unlearning nào đó, rồi tải $M_u$ lên server để thay thế $M$.
- **Người dùng cuối (User):** Tương tác với mô hình (ban đầu là $M$, sau là $M_u$) thông qua API do server cung cấp.

**Kẻ tấn công và Khả năng:** Bài báo xem xét hai loại kẻ tấn công chính:
- **Server MLaaS (Honest-but-Curious):**
  - *Giả định:* Server hoạt động đúng chức năng nhưng có thể tò mò và cố gắng trích xuất thông tin từ các mô hình mà nó quản lý.
  - *Khả năng:* Có quyền truy cập **White-box** vào cả hai mô hình $M$ và $M_u$ (do nhà phát triển tải lên). Nghĩa là server có thể thấy và phân tích các tham số nội bộ $(\theta \text{ và } \theta_u)$
  - *Hạn chế:* Giả định server không biết chính xác thuật toán unlearning nào đã được nhà phát triển sử dụng và cũng không có quyền truy cập vào tập dữ liệu huấn luyện gốc đầy đủ

- **Người dùng cuối (Malicious User):**
  - *Giả định:* Một người dùng thông thường của dịch vụ MLaaS nhưng có ý đồ xấu.
  - *Khả năng:* Chỉ có quyền truy cập **Black-box* thông qua API được cung cấp. Người dùng có thể gửi truy vấn đến mô hình và nhận lại kết quả dự đoán. Quan trọng là, người dùng có thể tương tác với mô hình $M$ trước khi nó bị thay thế và tương tác với mô hình $M_u$ sau khi nó được cập nhật.
  - *Hạn chế:* Không thể thấy tham số nội bộ của mô hình

### 2.2. Mục tiêu Tấn công và Tiêu chí Thành công (Attack Goals and Success Criteria)
Mục tiêu tổng quát của kẻ tấn công là trích xuất thông tin nhạy cảm về dữ liệu đã bị xóa $(D_u)$, làm vô hiệu hóa mục đích bảo vệ quyền riêng tư của việc xóa học. Mục tiêu cụ thể và cách đánh giá thành công phụ thuộc vào loại kẻ tấn công và quyền truy cập của họ:
- **Đối với Kẻ tấn công White-box (Server):**
  - **Mục tiêu:** Khôi phục Đặc trưng - Feature Recovery. Cố gắng tái tạo lại một mẫu dữ liệu $x'$ sao cho nó càng giống với mẫu dữ liệu gốc đã bị xoá $x^*$ càng tốt về mặt hình ảnh hoặc cấu trúc.
  - **Tiêu chí thành công:**
    - Đạt điểm cao trên các chỉ số như MSE, LPIPS thấp, PSNR cao
    - Quan trọng hơn, tạo ra được $x'$ mà **con người có thể nhận dạng được đối tượng hoặc thông tin nhạy cảm** chứa trong $x^*$ ban đầu, ngay cả khi $x'$ có nhiễu hoặc không hoàn hảo.
- **Đối với Kẻ tấn công Black-box (User):**
  - **Mục tiêu:** Suy luận Nhãn (Label Inference). Xác định chính xác các label (class) $C_u$ của dữ liệu đã bị xoá khỏi mô hình.
  - **Tiêu chí thành công:**
    - Các label được suy luận ra phải trùng khớp với các label thực tế của dữ liệu $D_u$ đã bị xoá.
    - Thành công của cuộc tấn công được đo bằng độ chính xác (accuracy) của việc suy luận nhãn.

### 2.3. Lỗ hổng cốt lõi
Điểm yếu cơ bản nằm ở chỗ kẻ tấn công (dù là server hay người dùng) có khả năng tiếp cận và so sánh trạng thái hoặc hành vi của mô hình trước $(M)$ và sau $(M_u)$ khi quá trình unlearning diễn ra. Chính sự khác biệt phát sinh từ việc loại bỏ dữ liệu $D_u$ là bề mặt tấn công, cho phép kẻ tấn công cô lập và suy luận thông tin về dữ liệu đã bị xoá.

## 3. Phương pháp Tấn công: Unlearning Inversion Attacks

### 3.1. Khôi phục Đặc trưng (Feature Recovery - White-box)

### 3.2. Suy luận Nhãn (Label Inference - Black-box)


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