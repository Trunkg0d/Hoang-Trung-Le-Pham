---
layout: posts
title: "Unlearning Inversion Attack"
date: 2025-04-16
published: false
description: "Một hướng dẫn tổng hợp về Machine Unlearning, Inversion Attacks, bề mặt tấn công mới sinh ra trong quá trình quên dữ liệu, và cách đánh giá an toàn thực chất."
reading_time: "18 min read"
categories: research security
permalink: /writing/unlearning-inversion-attack/
---

Machine Unlearning thường được mô tả như một “cục tẩy” cho mô hình học máy: khi một người dùng yêu cầu xóa dữ liệu, ta không chỉ xóa bản ghi khỏi cơ sở dữ liệu mà còn phải loại bỏ ảnh hưởng của bản ghi đó khỏi mô hình đã huấn luyện. Đây là một yêu cầu quan trọng đối với quyền riêng tư, nhưng nó dẫn đến một câu hỏi khó hơn:

> Một mô hình đã “quên” dữ liệu có thực sự ngừng tiết lộ thông tin về dữ liệu đó hay không?

Unlearning Inversion Attack khai thác chính sự thay đổi giữa mô hình trước và sau unlearning. Thay vì coi mô hình sau unlearning là một phiên bản an toàn hơn một cách mặc định, kẻ tấn công xem **quá trình chuyển trạng thái** như một nguồn tín hiệu mới.

Bài viết này gộp ba chủ đề vốn liên kết chặt chẽ—Machine Unlearning, Inversion Attacks và Unlearning Inversion Attacks—thành một bức tranh thống nhất.

## Machine Unlearning: mô hình “quên” như thế nào? {#machine-unlearning}

### Bài toán cơ bản

Giả sử:

- $D$ là tập dữ liệu huấn luyện ban đầu;
- $D_{forget}$ là phần dữ liệu cần loại bỏ;
- $D_{retain} = D \setminus D_{forget}$ là phần dữ liệu được giữ lại;
- $A$ là thuật toán huấn luyện;
- $M = A(D)$ là mô hình ban đầu.

Mục tiêu của thuật toán unlearning $U$ là tạo ra mô hình $M_u$ sao cho ảnh hưởng của $D_{forget}$ được loại bỏ, nhưng hiệu suất trên $D_{retain}$ vẫn được duy trì:

$$
M_u = U(M, D, D_{forget})
$$

Chuẩn tham chiếu mạnh nhất là mô hình được huấn luyện lại hoàn toàn trên dữ liệu giữ lại:

$$
M_{retrain} = A(D_{retain})
$$

Nếu phân phối đầu ra của $M_u$ gần với $M_{retrain}$, ta có cơ sở để nói quá trình unlearning đang tiến gần đến hành vi mong muốn. Tuy nhiên, “gần” về độ chính xác chưa đồng nghĩa với “an toàn” trước các cuộc tấn công suy luận.

<figure>
  <img src="{{ '/images/machine-unlearning/naive_retraining_and_machine_unlearning.png' | relative_url }}" alt="So sánh huấn luyện lại từ đầu và machine unlearning">
  <figcaption>Huấn luyện lại trên tập retain là chuẩn tham chiếu tự nhiên, nhưng thường tốn kém hơn unlearning.</figcaption>
</figure>

### Exact và Approximate Unlearning

**Exact Unlearning** hướng đến việc tạo ra một mô hình có phân phối tương đương với mô hình được huấn luyện lại từ đầu trên $D_{retain}$. Đây là mục tiêu chặt chẽ, nhưng thường đòi hỏi chi phí tính toán và lưu trữ lớn.

**Approximate Unlearning** chấp nhận một sai số nhỏ để đổi lấy tốc độ và khả năng triển khai. Mục tiêu có thể được mô tả bằng một khoảng cách phân phối:

$$
d(P_{M_u}, P_{M_{retrain}}) \leq \epsilon
$$

Trong thực tế, các phương pháp như fine-tuning trên tập retain, cập nhật dựa trên influence function, hoặc SISA thường nằm trong nhóm xấp xỉ. SISA chia dữ liệu thành nhiều shard, huấn luyện các mô hình con độc lập, rồi chỉ huấn luyện lại shard bị ảnh hưởng bởi yêu cầu xóa.

<figure>
  <img src="{{ '/images/machine-unlearning/sisa.png' | relative_url }}" alt="Kiến trúc SISA cho machine unlearning">
  <figcaption>SISA giới hạn phạm vi phải huấn luyện lại bằng cách chia dữ liệu thành các shard độc lập.</figcaption>
</figure>

### Đánh giá unlearning không chỉ bằng accuracy

Một đánh giá có ý nghĩa cần xem xét đồng thời:

1. **Removal quality:** hành vi trên $D_{forget}$ có gần với $M_{retrain}$ không?
2. **Retained utility:** hiệu suất trên $D_{retain}$ và tập test có được bảo toàn không?
3. **Efficiency:** thời gian, bộ nhớ và tài nguyên cần cho mỗi yêu cầu xóa.
4. **Privacy leakage:** membership inference, model inversion hoặc gradient inversion còn khai thác được gì?

Điểm thứ tư là nơi Inversion Attacks trở thành một công cụ đánh giá quan trọng—và cũng là nơi xuất hiện một bề mặt tấn công mới.

## Inversion Attacks: đọc ngược dấu vết của mô hình {#inversion-attacks}

Inversion Attack là nhóm kỹ thuật dùng thông tin quan sát được từ mô hình để suy ngược về dữ liệu huấn luyện. Hai nhánh thường gặp là:

- **Model Inversion Attack:** khai thác đầu ra, confidence score, logits hoặc quyền truy cập tham số để suy ra thuộc tính hay mẫu đại diện của dữ liệu huấn luyện.
- **Gradient Inversion Attack:** dùng gradient hoặc thay đổi tham số để tái tạo đầu vào và nhãn đã tạo ra gradient đó.

Với một mô hình tuyến tính đơn giản

$$
f(x) = w_1x_1 + w_2x_2 + \dots + w_tx_t + b,
$$

nếu kẻ tấn công biết đầu ra $y$, tham số mô hình và tất cả đặc trưng trừ $x_1$, đặc trưng riêng tư có thể được suy ra trực tiếp:

$$
\hat{x}_1 = \frac{y - (w_2x_2 + \dots + w_tx_t + b)}{w_1}.
$$

Với mạng nơ-ron sâu, phép đảo không còn đơn giản, nhưng cùng một trực giác vẫn tồn tại: tối ưu một đầu vào giả sao cho hành vi hoặc gradient của nó khớp với tín hiệu quan sát được.

<figure>
  <img src="{{ '/images/machine-unlearning/mu-vs-inversion-attacks.png' | relative_url }}" alt="Mối liên hệ giữa machine unlearning và inversion attacks">
  <figcaption>Inversion attacks vừa là mối đe dọa, vừa là công cụ để kiểm tra mức độ unlearning thực sự.</figcaption>
</figure>

### Ba mối liên hệ quan trọng

Machine Unlearning và Inversion Attacks giao nhau theo ba cách:

1. Inversion attacks có thể làm lộ thông tin mà unlearning được kỳ vọng phải loại bỏ.
2. Khả năng chống inversion có thể dùng như một tín hiệu đánh giá chất lượng unlearning.
3. Một số kỹ thuật unlearning lại được dùng như cơ chế giảm thiểu rò rỉ gradient trong hệ thống phân tán.

Điểm nghịch lý nằm ở chỗ: quá trình xóa có thể giảm một loại rò rỉ, nhưng sự khác biệt giữa hai phiên bản mô hình lại tạo ra một tín hiệu mới cho kẻ tấn công.

## Threat model của Unlearning Inversion Attack {#threat-model}

Hãy xét một hệ thống Machine Learning as a Service (MLaaS): nhà cung cấp huấn luyện mô hình $M$ trên $D$, sau đó nhận yêu cầu xóa $D_{forget}$ và triển khai mô hình $M_u$.

Kẻ tấn công có thể đứng ở hai vị trí:

- **White-box attacker:** truy cập được tham số của cả $M$ và $M_u$—ví dụ máy chủ hoặc một bên nội bộ.
- **Black-box attacker:** chỉ truy vấn hai mô hình qua API và nhận confidence score hoặc logits.

Lỗ hổng cốt lõi không nhất thiết nằm trong riêng $M$ hay $M_u$, mà nằm trong **chênh lệch** giữa chúng. Nếu sự thay đổi này chủ yếu được tạo ra để loại bỏ $D_{forget}$, nó có thể cô lập dấu vết của chính dữ liệu cần quên.

### White-box: khôi phục đặc trưng

Trong huấn luyện bằng gradient descent, một bước cập nhật có dạng:

$$
\theta_{k+1} = \theta_k - \alpha \nabla_\theta \mathcal{L}(\theta_k).
$$

Kẻ tấn công white-box sử dụng chênh lệch tham số giữa mô hình gốc và mô hình đã unlearn như một ước lượng cho tín hiệu gradient liên quan đến dữ liệu bị xóa:

$$
\nabla^\ast \approx \theta - \theta_u.
$$

Sau đó, họ khởi tạo một đầu vào giả $x'$ và tối ưu nó sao cho gradient $\nabla'$ sinh ra bởi $x'$ có hướng gần với $\nabla^\ast$:

$$
\arg\min_{x'} -\operatorname{cos}(\nabla', \nabla^\ast) + \lambda TV(x').
$$

Trong đó $TV(x')$ là regularizer Total Variation giúp ảnh tái tạo mượt và tự nhiên hơn. Kết quả tối ưu có thể phục hồi cấu trúc hoặc thuộc tính nhận diện được của mẫu đã bị xóa—even khi mẫu gốc không còn được cung cấp trực tiếp.

> Ý nghĩa bảo mật: việc giữ đồng thời checkpoint trước và sau unlearning có thể tạo ra nhiều thông tin hơn việc chỉ lộ một checkpoint.

### Black-box: suy luận nhãn đã bị quên

Kẻ tấn công black-box không cần truy cập tham số. Thay vào đó, họ tạo các **probing samples** cho từng lớp mục tiêu và so sánh phản hồi của hai mô hình.

Quy trình khái quát:

1. Tạo một nhóm mẫu $x'$ mà mô hình gốc $M$ dự đoán thuộc lớp $y_t$ với confidence cao.
2. Lưu vector dự đoán của $M$ trên các mẫu này.
3. Gửi cùng các mẫu đến $M_u$.
4. Tính mức thay đổi confidence theo từng lớp.

Với $m$ probing samples của một lớp, chênh lệch trung bình có thể viết là:

$$
\Delta_p = \frac{1}{m}\sum_{i=1}^{m}\left(\hat{\mathcal{Y}}_i - \mathcal{Y}_i\right).
$$

Nếu confidence của một lớp giảm nổi bật sau unlearning, lớp đó trở thành ứng viên cho nhãn của dữ liệu đã bị xóa. Khi cần suy ra nhiều lớp, attacker xếp hạng các lớp theo độ giảm confidence và chọn Top-$K$.

Điều đáng chú ý là attacker không cần tái tạo được ảnh gốc. Chỉ riêng nhãn—bệnh lý, vị trí, nhóm nhân khẩu học hoặc loại hành vi—đã có thể là thông tin nhạy cảm.

## Vì sao “đã quên” vẫn chưa đủ?

Unlearning thường được tối ưu để đưa mô hình đến một trạng thái mong muốn. Nhưng bảo mật không chỉ phụ thuộc vào trạng thái cuối; nó còn phụ thuộc vào thông tin lộ ra trong quá trình chuyển trạng thái.

Có ba nguyên nhân chính:

- **Difference amplification:** lấy hiệu hai mô hình có thể loại bỏ nhiều tín hiệu chung và làm nổi bật đúng phần liên quan đến dữ liệu bị xóa.
- **Overexposed outputs:** logits hoặc confidence score chi tiết cung cấp nhiều tín hiệu hơn nhãn top-1.
- **Weak evaluation:** accuracy trên forget set giảm không chứng minh rằng thông tin nhận dạng hoặc nhãn đã biến mất.

Do đó, privacy claim của một thuật toán unlearning nên được kiểm tra trong threat model cụ thể, thay vì suy ra trực tiếp từ performance metric.

## Đánh giá và phòng thủ {#defense}

Không có một biện pháp đơn lẻ giải quyết toàn bộ bề mặt tấn công. Một thiết kế phòng thủ tốt nên kết hợp nhiều lớp.

### 1. Đánh giá theo chuẩn retraining

So sánh $M_u$ với $M_{retrain}$ trên cả utility và leakage. Nếu chỉ so với mô hình gốc, ta không biết mức chênh lệch nào là hành vi “quên đúng” và mức nào là dấu vết bất thường.

### 2. Hạn chế quan sát cặp mô hình

- Không để cùng một người dùng truy cập lâu dài cả phiên bản trước và sau unlearning.
- Hạn chế lưu hoặc chia sẻ checkpoint lịch sử không cần thiết.
- Phân tách quyền truy cập vận hành, audit và nghiên cứu.

### 3. Giảm độ chi tiết của API

Chỉ trả về thông tin cần thiết cho tác vụ. Nhãn top-1, confidence đã hiệu chỉnh, rate limiting và phát hiện truy vấn bất thường có thể làm probing attack khó hơn. Đây là biện pháp giảm thiểu, không phải bằng chứng unlearning đã an toàn.

### 4. Thêm cơ chế bảo vệ ở mức cập nhật

Clipping, noise có kiểm soát, regularization, differential privacy hoặc cập nhật theo cụm có thể giảm khả năng cô lập tín hiệu của một mẫu. Mỗi cơ chế đều tạo trade-off giữa privacy, utility và chi phí.

### 5. Dùng attack-based evaluation

Một bộ đánh giá nên bao gồm ít nhất:

- Membership Inference Attack;
- Model/Gradient Inversion Attack;
- Forgotten-label inference;
- kiểm tra với cả white-box và black-box access;
- nhiều kích thước và cấu trúc của $D_{forget}$.

Nếu một phương pháp chỉ vượt qua một attacker yếu, kết luận phù hợp là “an toàn trong thiết lập đã kiểm tra”, không phải “đã xóa hoàn toàn thông tin”.

## Checklist triển khai thực tế

Trước khi công bố một pipeline unlearning, có thể rà lại năm câu hỏi:

1. Ai có thể truy cập mô hình trước và sau khi unlearn?
2. API trả về nhãn, confidence hay toàn bộ logits?
3. Mô hình sau unlearning gần retraining baseline đến mức nào?
4. Những attack-based metrics nào đã được chạy trên forget set?
5. Hệ thống có audit log, rate limit và chính sách checkpoint rõ ràng không?

Machine Unlearning không chỉ là một bài toán tối ưu hóa; nó là một bài toán hệ thống. Thuật toán, API, quyền truy cập, lưu trữ phiên bản và quy trình đánh giá đều cùng quyết định dữ liệu có thực sự được “quên” hay không.

## Kết luận

Unlearning Inversion Attack cho thấy một trực giác quan trọng: **xóa ảnh hưởng khỏi mô hình và ngăn mô hình tiết lộ thông tin là hai mục tiêu liên quan nhưng không đồng nhất**.

Một mô hình có thể đạt accuracy tốt, gần với retraining baseline, và vẫn để lộ tín hiệu về dữ liệu bị xóa khi attacker so sánh hai phiên bản. Vì vậy, đánh giá unlearning cần dịch chuyển từ câu hỏi “mô hình còn dự đoán đúng trên forget set không?” sang câu hỏi mạnh hơn: “với quyền truy cập thực tế, attacker còn suy ra được gì?”

---

## Tài liệu tham khảo

- Hu, H., Wang, S., Dong, T., & Xue, M. (2024). [Learn What You Want to Unlearn: Unlearning Inversion Attacks against Machine Unlearning](https://arxiv.org/abs/2404.03233).
- Xu, J., Wu, Z., Wang, C., & Jia, X. (2024). [Machine Unlearning: Solutions and Challenges](https://doi.org/10.1109/TETCI.2024.3379240). *IEEE Transactions on Emerging Topics in Computational Intelligence*, 8(3), 2150–2168.
- Wang, W., Tian, Z., Zhang, C., & Yu, S. (2024). [Machine Unlearning: A Comprehensive Survey](https://arxiv.org/abs/2405.07406).
- Graves, L., Nagisetty, V., & Ganesh, V. (2020). [Amnesiac Machine Learning](https://arxiv.org/abs/2010.10981).
- [NeurIPS 2023 Machine Unlearning Challenge — Starting Kit](https://unlearning-challenge.github.io/).
