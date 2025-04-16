---
layout: post
title: "Giải mã Learn What You Want to Unlearn: Tấn công Đảo ngược Bỏ học"
date: 2025-04-16
categories: jekyll update
---
## 1. Giới thiệu
Trong kỷ nguyên số, dữ liệu cá nhân ngày càng được thu thập và xử lý nhiều hơn bao giờ hết. Đi kèm với đó là những lo ngại về quyền riêng tư và nhu cầu kiểm soát thông tin cá nhân. Các quy định như GDPR ở Châu Âu và CCPA ở California đã ra đời, trao cho người dùng "Quyền được Lãng quên" (Right to be Forgotten) - quyền yêu cầu các tổ chức xóa dữ liệu của họ.   

Việc xóa dữ liệu khỏi cơ sở dữ liệu truyền thống tương đối đơn giản. Tuy nhiên, với các mô hình Học máy (Machine Learning - ML), đặc biệt là Mạng Nơ-ron Sâu (Deep Neural Networks - DNNs), việc này phức tạp hơn nhiều do bản chất phức tạp và ngẫu nhiên của quá trình huấn luyện. Để giải quyết thách thức này, kỹ thuật **Machine Unlearning** (nếu bạn không biết gì về Machine Unlearning thì có thể đọc blog trước tại <a href="https://trunkg0d.github.io/Hoang-Trung-Le-Pham/jekyll/update/2025/03/31/machine-unlearning.html">đây</a>) đã ra đời, nhằm mục đích loại bỏ ảnh hưởng của một số dữ liệu huấn luyện cụ thể khỏi mô hình đã được huấn luyện.   

Nghe có vẻ đây là một giải pháp hoàn hảo cho quyền riêng tư? Nhưng liệu quá trình "quên" này có thực sự an toàn? Bài báo **"Learn What You Want to Unlearn: Unlearning Inversion Attacks against Machine Unlearning"**  đã thực hiện một nghiên cứu tiên phong, chỉ ra rằng chính quá trình xóa học lại có thể mở ra một bề mặt tấn công mới, làm lộ thông tin nhạy cảm của chính dữ liệu đáng lẽ phải được "quên" đi. Bài viết này sẽ giải mã ý tưởng cốt lõi và các phương pháp tấn công được đề xuất trong nghiên cứu quan trọng này.

## 2. Vấn đề cốt lõi: Lỗ hổng từ sự Khác biệt giữa hai Phiên bản Mô hình

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