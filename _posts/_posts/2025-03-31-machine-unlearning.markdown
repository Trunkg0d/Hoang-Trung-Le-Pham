---
title: "Machine Unlearning: Khi AI 'quên' đi những gì đã học"
date: 2025-03-31
layout: post
categories: jekyll update
---

<h2>1. Giới thiệu</h2>
<p>Trong kỷ nguyên của Trí tuệ Nhân tạo (AI), các mô hình học máy ngày càng trở nên mạnh mẽ và được ứng dụng rộng rãi trong nhiều lĩnh vực của đời sống. Tuy nhiên, đi kèm với sự phát triển này là những lo ngại về quyền riêng tư và đạo đức liên quan đến việc thu thập và sử dụng dữ liệu cá nhân.</p>

<p>Hãy tưởng tượng một mô hình AI được đào tạo trên dữ liệu của hàng triệu người dùng, bao gồm cả những thông tin nhạy cảm như lịch sử tìm kiếm, vị trí và thông tin tài chính. Nếu một người dùng muốn xóa dữ liệu của họ khỏi mô hình này, liệu điều đó có khả thi? Kiểu như bạn từng "lỡ" dạy AI nhận diện món phở, xong nó nhận diện nhầm tô bún riêu nhà kế bên? Hoặc "lỡ" cho AI xem hết ảnh "dìm hàng" của bạn, giờ nó đem đi "tám chuyện" với AI khác? Làm thế nào chúng ta có thể đảm bảo rằng AI "quên" đi những gì đã học, đồng thời duy trì hiệu suất của mô hình?</p>

<p>Đây chính là lúc Machine Unlearning xuất hiện, như một "cục tẩy" cho AI. Machine Unlearning là một lĩnh vực nghiên cứu mới nổi, tập trung vào việc phát triển các phương pháp cho phép các mô hình học máy loại bỏ thông tin cụ thể khỏi quá trình đào tạo. Nói một cách đơn giản, Machine Unlearning giúp AI "quên" đi những dữ liệu không còn cần thiết hoặc không phù hợp, từ đó bảo vệ quyền riêng tư của người dùng và giải quyết các vấn đề đạo đức liên quan đến việc sử dụng AI. Kiểu như, khi bạn muốn AI "quên" đi mấy tấm hình "dìm hàng" kia, machine unlearning sẽ giúp AI "xoá kí ức" nhẹ nhàng.</p>

<p>Trong bài blog này, chúng ta sẽ cùng nhau khám phá sâu hơn về Machine Unlearning, bao gồm các phương pháp, thách thức và ứng dụng tiềm năng của nó. Hãy cùng nhau tìm hiểu về tương lai của việc kiểm soát dữ liệu trong kỷ nguyên AI!</p>

<h2>2. Các phương pháp Machine Unlearning</h2>
Phần này sẽ đi sâu vào các kỹ thuật và phương pháp giúp AI "quên" đi dữ liệu không mong muốn. Từ việc định nghĩa các tập dữ liệu liên quan đến việc quên, đến việc phân loại các phương pháp unlearning, bạn sẽ khám phá cách mà các nhà nghiên cứu đang giải quyết vấn đề này.

<h3>a. Định nghĩa các tập dữ liệu</h3>
Trước khi đi vào các phương pháp cụ thể, việc định nghĩa rõ ràng các tập dữ liệu liên quan là bước đầu tiên và quan trọng. Điều này giúp chúng ta có một khung làm việc rõ ràng và chính xác trong quá trình Machine Unlearning.
<ul>
<li>$D$: Tập dữ liệu gốc, dùng để huấn luyện mô hình ban đầu</li>
<li>$D_{retain}$: Tập dữ liệu còn lại, sau khi loại bỏ dữ liệu cần quên</li>
<li>$D_{forget}$: Tập dữ liệu cần quên, sẽ được loại bỏ ảnh hưởng khỏi mô hình</li>
</ul>

<h3>b. Phân loại</h3>
Ở đây, chúng ta sẽ phân tích và so sánh hai phương pháp chính trong Machine Unlearning: Exact Unlearning và Approximate Unlearning. Mỗi phương pháp có những ưu và nhược điểm riêng, phù hợp với các tình huống và yêu cầu khác nhau.

<h4>b.1. Exact Unlearning</h4>
<p>Đây là phương pháp lý tưởng, nhưng khó thực hiện nhất. Mục tiêu là tạo ra một mô hình $M'$ sao cho nó tương đương với mô hình được huấn luyện trực tiếp trên $D_{retain}$. Công thức lý tưởng như sau:</p>
$$M' = Train(D_{retain})$$
<p>trong đó, $Train(D_{retain})$ là hàm huấn luyện mô hình trên tập dữ liệu $D_{retain}$. Tuy nhiên, cách này gặp các khó khăn đó là:</p>
<ul>
<li>Tính toán lại toàn bộ mô hình từ $D_{retain}$ có thể tốn kém, đặc biệt đối với các mô hình lớn.</li>
<li>Trong một số trường hợp, việc tính toán lại chính xác là không khả thi.</li>
</ul>

<h4>b.2. Approximate Unlearning</h4>
<p>Phương pháp này cố gắng giảm thiểu ảnh hưởng của $D_{forget}$ mà không cần tính toán lại toàn bộ mô hình. Các phương pháp thường gặp của dạng này bao gồm:</p>
<ul>
<li><strong>Sử dụng Score function:</strong> Việc đánh giá độ ảnh hưởng của 1 điểm dữ liệu lên kết quả của mô hình. Các điểm dữ liệu nào có ảnh hưởng càng lớn thì sẽ cần điều chỉnh trọng số (weight) của Model trên các điểm dữ liệu đó càng nhiều.</li>
<li><strong>Phương pháp SISA (Sharded, Isolated, Sliced, Aggregated):</strong>
<ul>
<li>Chia $D$ thành nhiều "shards" nhỏ</li>
<li>Huấn luyện các mô hình con trên từng shard</li>
<li>Khi cần quên dữ liệu, chỉ cần huấn luyện lại các mô hình con liên quan</li>
<li>Tổng hợp các mô hình con để tạo ra mô hình cuối cùng</li>
</ul>
</li>
<li><strong>Phương pháp dựa trên giải thuật SVD (Singular Value Decomposition):</strong>
<ul>
<li>Sử dụng phân rã giá trị đơn lẻ để phân tích ma trận dữ liệu, và phân tích độ ảnh hưởng của từng giá trị đến kết quả cuối cùng.</li>
</ul>
</li>
</ul>
<p>Công thức chung (ví dụ về SISA):</p>
$$M' = Aggregate(Train(Shard_1), Train(Shard_2), ..., Train(Shard_n))$$
<p>trong đó, $Aggregate()$ là hàm tổng hợp các mô hình con.</p>

<h2>3. Đánh giá Machine Unlearning</h2>
<p>Sau khi đã triển khai các phương pháp unlearning, việc đánh giá hiệu quả của chúng là bước không thể thiếu. Phần này sẽ trình bày các tiêu chí và phương pháp đánh giá để đảm bảo rằng quá trình unlearning diễn ra thành công và an toàn. Các tiêu chí đánh giá có thể được phân thành các nhóm sau:</p>

<h3>a. Hiệu suất mô hình (Performance)</h3>

<h4>a.1. Độ chính xác (Accuracy)</h4>
<ul>
<li><strong>Trên tập dữ liệu quên (Forget Set):</strong>
<ul>
<li>Đánh giá sự giảm độ chính xác trên tập dữ liệu này để xác định mức độ "quên" của mô hình.</li>
<li>So sánh độ chính xác của mô hình sau unlearning với mô hình được huấn luyện lại trên tập dữ liệu quên (retrained model) để đảm bảo tính nhất quán.</li>
</ul>
</li>
<li><strong>Trên tập dữ liệu giữ lại (Retain Set):</strong>
<ul>
<li>Đảm bảo rằng mô hình sau unlearning vẫn duy trì hiệu suất trên tập dữ liệu này.</li>
<li>So sánh độ chính xác của mô hình sau unlearning với mô hình gốc để đánh giá sự chênh lệch.</li>
</ul>
</li>
<li><strong>Trên tập dữ liệu kiểm tra (Test Set):</strong>
<ul>
<li>Đảm bảo rằng mô hình sau unlearning vẫn có hiệu suất tốt trên tập dữ liệu kiểm tra, để đánh giá khả năng tổng quát hóa của mô hình.</li>
</ul>
</li>
</ul>

<h4>a.2. Thời gian Unlearn (Unlearn Time)</h4>
<p>Đo lường thời gian cần thiết để thực hiện quá trình unlearning.</p>

<h4>a.3. Thời gian Học lại (Relearn Time)</h4>
<p>Đo lường thời gian (hoặc số epoch) cần thiết để mô hình "học lại" thông tin đã quên bằng cách huấn luyện lại trên tập dữ liệu quên.</p>

<h3>b. Bảo mật và Rò rỉ thông tin (Security and Information Leakage)</h3>

<h4>b.1. Các phương pháp tấn công (Attack-based Evaluation)</h4>
<ul>
<li>Sử dụng các phương pháp tấn công như Membership Inference Attack (MIA) hoặc Inversion Attack để đánh giá mức độ rò rỉ thông tin không mong muốn.</li>
<li>So sánh mức độ rò rỉ thông tin giữa mô hình trước và sau unlearning.</li>
</ul>
