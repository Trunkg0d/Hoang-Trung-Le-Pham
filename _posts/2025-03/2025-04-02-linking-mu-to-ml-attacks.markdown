---
layout: post
title: Machine Unlearning trong ngữ cảnh Machine Learning Attacks
date: 2025-04-02
categories: jekyll update
---

## 1. Giới thiệu

## 2. Machine Unlearning và Security Attacks
### 2.1. Backdoor Attacks

#### a. Backdoor Attack  với Machine Unlearning

#### b. Backdoor Defense dựa trên Machine Unlearning

#### c. Đánh giá MU Framework thông qua Backdoor-based metrics

#### d. Backdoor in MU Verification frameworks

### 2.2. Membership Inference Attack

#### a. Membership Inference Attack với MU

#### b. Đánh giá MU Framework thông qua MIA-based metrics

#### c. Evaluating MU Verification framework through MIA-based metrics

### 2.3. Adversarial Attack
#### a. Adversarial Attacks với Machine Unlearning
#### b. Defense Adversarial Attacks dựa trên Machine Unlearning


### 2.4. Inversion Attacks
Inversion Attacks, hay còn gọi là Tấn công Đảo Ngược, là một trong những mối đe dọa đáng lo ngại đối với quyền riêng tư trong lĩnh vực Trí tuệ Nhân Tạo. Chúng ta có thể chia chúng thành hai loại chính:

* **Model Inversion Attacks:** Những cuộc tấn công "lật mặt" mô hình.
* **Gradient Inversion Attacks:** Những cuộc tấn công "lật mặt" thông qua độ dốc.

Trong đó, Model Inversion Attacks (MIAs) đặc biệt nguy hiểm vì chúng có thể "mò mẫm" và tái tạo lại những thông tin nhạy cảm mà mô hình AI đã học được từ dữ liệu huấn luyện. Điều này giống như việc cố gắng đoán xem một người đã học những gì dựa trên cách họ trả lời các câu hỏi. Vấn đề càng trở nên nghiêm trọng khi những thông tin này lại liên quan đến các chi tiết cá nhân riêng tư.

Nói một cách chi tiết hơn, MIA lợi dụng mối quan hệ mật thiết giữa dữ liệu mà AI được "học" và kết quả mà nó đưa ra để dựng lại những thuộc tính quan trọng của dữ liệu ban đầu. Các chuyên gia thường xem việc "lật mặt" này như một bài toán tối ưu hóa phức tạp. Mục tiêu là tìm ra các giá trị đặc trưng nhạy cảm có khả năng cao nhất dẫn đến kết quả của mô hình AI.

**Công thức "lật mặt":**

* Cho $x = (x_1, x_2, ..., x_t)$ là "bộ mặt" đầy đủ của một cá nhân, trong đó $x_1$ là những chi tiết riêng tư, còn $x_2$ đến $x_t$ là những thông tin bên ngoài.
* $y = f(x)$ là kết quả mà mô hình AI $f$ đưa ra khi nhìn vào "bộ mặt" $x$.
* Kẻ tấn công có quyền "nhìn trộm" mô hình $f$ và những thông tin phụ trợ $side(x, y) = (x_2, ..., x_t, y)$, tức là biết hết thông tin "bên ngoài" của cá nhân và kết quả mà AI đưa ra.

Khi đó, cuộc tấn công "lật mặt" này sẽ tìm cách "đoán" ra chi tiết riêng tư $x_1$ bằng công thức:

$$
\hat{x}_1 = argmax_{x_1} P(x_1 | x_2, \dots , x_t, y, f)
$$

(Đừng lo lắng về công thức này nếu bạn không quen thuộc với toán học! Nó chỉ là cách viết ngắn gọn cho việc tìm giá trị có khả năng cao nhất.)

**"Lật mặt" mô hình tuyến tính:**

Nếu mô hình AI chỉ đơn giản như một phép tính đường thẳng $f(x) = w_1x_1 + w_2x_2 + \dots + w_tx_t + b$, kẻ tấn công có thể dễ dàng "lật mặt" $x_1$:

$$
\hat{x}_1 = \frac{y - (w_2x_2 + \dots + w_tx_t + b)}{w_1}
$$

**Các bước "lật mặt":**

1.  **Chuẩn bị:** Kẻ tấn công có quyền "nhìn thấu" mô hình AI và biết tất cả mọi thông tin đầu vào, trừ những chi tiết riêng tư mà họ muốn "lật mặt".
2.  **Suy luận:** Bằng cách "mổ xẻ" các con số bên trong mô hình và những thông tin "bên ngoài" sẵn có, kẻ tấn công cố gắng "đoán" ra thông tin riêng tư.
3. **Chiến lược "lật mặt":**
- Nếu mô hình AI đơn giản như một "đường thẳng" (mô hình tuyến tính), việc "đoán" ra thông tin riêng tư ($x_1$) chỉ cần thực hiện bằng một vài phép tính
- Nhưng đối với những mô hình AI phức tạp hơn, kẻ tấn công cần sử dụng các phương pháp tối ưu hoá hoặc các thuật toán dựa trên độ dốc (gradient-based methods) để "lật mặt"

Trong khi các cuộc tấn công MIA truyền thống tập trung vào việc "lật mặt" dữ liệu trước khi AI được unlearning, thì hiện nay, mối lo ngại đặt ra là **liệu có thể "lật mặt" thông tin về dữ liệu đã được "quên" hay không**. Tức là, liệu sau khi AI cố gắng "xóa" một số thông tin, có còn "tàn dư" nào trong mô hình mà kẻ tấn công có thể lợi dụng để "đọc trộm" không? Bên cạnh đó, **các cuộc tấn công MIA cũng được sử dụng như một "bài kiểm tra" để đánh giá hiệu quả của quá trình unlearning**. Nếu mô hình càng khó bị "lật mặt", điều đó có nghĩa là quá trình unlearning diễn ra càng tốt.

Ngoài Model Inversion Attacks, chúng ta còn có Gradient Inversion Attacks (GIAs). Các cuộc tấn công GIA lợi dụng việc các mô hình chia sẻ các "con số nội bộ" (trọng số mô hình) và "hướng dẫn học tập" (độ dốc) để "rò rỉ" dữ liệu riêng tư. Bằng cách tái cấu trúc dữ liệu hoặc nhãn từ việc giảm thiểu sự khác biệt giữa các "hướng dẫn học tập" quan sát được và "hướng dẫn học tập" giả mạo, kẻ tấn công có thể "xây dựng lại" thông tin cá nhân. Các phương pháp này có thể dựa trên lặp lại hoặc đệ quy.

Nói tóm lại, cả Model Inversion Attacks và Gradient Inversion Attacks đều là những mối đe dọa đến quyền riêng tư, tìm cách lấy cắp thông tin nhạy cảm từ các mô hình học máy:
- Các cuộc tấn công MIA sử dụng kết quả đầu ra của mô hình và thông tin "bên ngoài" để suy ra các thuộc tính bị thiếu, thường nhắm vào các mô hình đã được triển khai, bất kể kẻ tấn công có quyền "nhìn thấu" mô hình (white-box) hay chỉ có quyền "sử dụng" mô hình (black-box).
- Ngược lại, các cuộc tấn công GIA khai thác các "hướng dẫn học tập" được chia sẻ trong các hệ thống học phân tán, chẳng hạn như Federated Learning, để tái tạo lại toàn bộ dữ liệu đầu vào.

Sau khi nghiên cứu các tài liệu liên quan đến Machine Unlearning và Model Inversion Attacks, ta có thể phân loại như sau:
- Model Inversion Attack gây khó khăn cho Machine Unlearning;
- Model Inversion Attack có thể được sử dụng làm "thước đo" độ hiệu quả của Machine Unlearning;
- Machine Unlearning được sử dụng như một biện pháp phòng thủ chống lại Gradient Inversion Attack

<div align="center">
    <figure>
        <img src="{{site.baseurl}}/images/machine-unlearning/mu-vs-inversion-attacks.png" alt="MU and Inversion Attacks" width="700">
        <figcaption><b>Phân loại Machine Unlearning và Inversion Attacks.</b></figcaption>
    </figure>
</div>

#### a. Model Inversion Attack với Machine Unlearning
Trong công trình nghiên cứu [[Hu et al., 2024]](#hu2024), nhóm tác giả trình bày một cơ chế unlearning inversion attacks như một mối đe doạ quyền riêng tư mới nhắm vào Machine Unlearning trong DNN. Các cuộc tấn công này tiết lộ những thông tin nhạy cảm về dữ liệu unlearned thông qua hai phương pháp chính: rò rỉ đặc trưng (feature leakage) và rò rỉ nhãn (label leakage). Trong feature leakage, kẻ tấn công có quyền truy cập white-box vào cả mô hình gốc và mô hình unlearned sử dụng các kỹ thuật đảo ngược gradient để tái tạo các đặc trưng của mẫu unlearned từ những thay đổi trong tham số mô hình. Ngược lại, trong label leakage, kẻ tấn công có quyền truy cập black-box tạo ra các mẫu thăm dò để đánh giá sự khác biệt trong dự đoán giữa mô hình gốc và mô hình unlearned, cho phép chúng suy ra nhãn của dữ liệu unlearned. Thông qua các thực nghiệm, họ đã chứng minh được các lỗ hổng này bằng phương pháp Machine Unlearning.

#### b. Model Inversion Attack as an evaluation metric for MU
Nghiên cứu của [[Graves et al., 2020]](#graves2020) đã tập trung vào việc "thử thách" khả năng chống chịu của các phương pháp "quên đi" thông tin (unlearning) trước những cuộc tấn công "lật mặt" mô hình (Model Inversion Attacks). Thay vì chỉ sử dụng các cuộc tấn công "lật mặt" thông thường, họ đã "nâng cấp" nó một chút để kiểm tra kỹ lưỡng hơn. Đặc biệt, Graves et al. đưa ra khái niệm "unlearning mất trí nhớ" (amnesiac unlearning), trong đó AI được dạy quên đi một số thông tin, nhưng kẻ tấn công lại không biết chính xác thông tin đó là gì. Nghiên cứu này coi như một cuộc tấn công "lật mặt" là thành công nếu kẻ tấn công có thể "mò mẫm" và suy ra được ý nghĩa của các nhóm dữ liệu khác nhau (các "lớp") từ mô hình AI.


#### c. MU as a defense against Gradient inversion attack
Công trình của [[Gao et al., 2024]](#GAO2024) tập trung vào việc sử dụng các phương pháp "quên đi" thông tin dựa trên thống kê (statistical Machine Unlearning) trong môi trường học liên kết (Federated Learning). Mục tiêu chính là tạo ra một "chiến thuật" phòng thủ chống lại các cuộc tấn công "lật mặt" độ dốc (gradient inversion attacks), lấy cảm hứng từ các nguyên tắc của statistical Machine Unlearning. Để làm được điều này, các tác giả đề xuất "chuyển hướng" từ việc xử lý dữ liệu cá nhân sang xử lý các "bản tóm tắt" thống kê của dữ liệu. Cách tiếp cận này tạo ra một "tầng trừu tượng" giúp bảo vệ quyền riêng tư và cho phép "quên" đi dữ liệu một cách chọn lọc. Cụ thể, phương pháp này "phá vỡ" quá trình biến đổi từ dữ liệu thô thành "hướng dẫn học tập" (gradients) bằng cách thay thế chúng bằng các "hướng dẫn học tập" được tính toán từ thông tin thống kê. Đồng thời, họ sử dụng các mô hình "thầy-trò" được huấn luyện với các hàm mất mát kép để "đánh lạc hướng" những kẻ tấn công (trong trường hợp các máy chủ tò mò nhưng trung thực).

## 3. Kết luận

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