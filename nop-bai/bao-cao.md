# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Đinh Đức Anh |
| MSSV | 2A202601714 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/AnhDc2004/Track2-K4-Day21-2A202601714-DinhDucAnh |
| Ngày nộp | 21/08/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 200 | 0.1 | 5 | 0.71493 | 0.874 |
| 2 | 100 | 0.1 | 3 | 0.71090 | 0.878 |
| 3 | 200 | 0.05 | 3 | 0.70142 | 0.874 |
| 4 | 50 | 0.05 | 2 | 0.60512 | 0.846 |

**Bộ siêu tham số đã chọn:** `n_estimators=200`, `learning_rate=0.1`, `max_depth=5`.

**Lý do:** Bộ này đạt f1_score cao nhất (0,71493), vượt ngưỡng 0,65. Lần có accuracy cao nhất (lần 2, 0,878) không trùng lần có f1 cao nhất, cho thấy accuracy không phản ánh đúng khả năng bắt lớp thiểu số nên không dùng để chọn mô hình. Đánh đổi giữa hai tham số cũng rõ: giảm learning_rate xuống 0,05 thì cần 200 cây mới giữ f1 ở 0,70 (lần 3), còn vừa learning_rate thấp vừa ít cây, cây nông (lần 4) thì f1 sụt còn 0,605 và trượt ngưỡng dù accuracy vẫn 0,846.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Tập Adult mất cân bằng lớp: chỉ 24,8% mẫu thuộc lớp thu nhập cao. Hệ quả là một mô hình vô dụng luôn trả lời "thu nhập thấp" vẫn đạt accuracy 0,752 — nghe có vẻ tốt nhưng không bắt được trường hợp thu nhập cao nào và có f1 bằng 0, nên accuracy gây hiểu nhầm nghiêm trọng trên dữ liệu lệch lớp. F1 của lớp dương đo đồng thời precision và recall trên đúng nhóm thiểu số cần dự đoán — thứ accuracy hoàn toàn bỏ qua. Khi gọi f1_score cũng không truyền average="weighted" hay "macro", vì các cách tính đó bị lớp đa số kéo lên cao, làm ngưỡng 0,65 mất ý nghĩa kiểm soát.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| Job Train lỗi dvc pull trên CI (exit 255) | requirements.txt ghi dvc[gs] trong khi remote là S3 | Đổi sang dvc[s3] rồi chạy lại pipeline |
| CI báo không tìm thấy profile AWS "income-lab" | Profile khai trong .dvc/config dùng chung, CI chỉ có credentials qua biến môi trường | Chuyển profile sang .dvc/config.local (không commit) |
| Job Release lỗi "ssh: no key found" | Secret SERVER_SSH_KEY dán thiếu định dạng private key | Dán lại đủ dòng BEGIN/END kèm newline cuối |

---

## 4. So Sánh Bước 2 và Bước 3 (bắt buộc, 2 - 3 câu)

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | 0.7149 | 0.874 |
| Bước 3 (thêm `train_batch2`) | 0.7354 | 0.882 |

**Nhận xét:** Gấp đôi dữ liệu làm f1 tăng từ 0,7149 lên 0,7354, accuracy từ 0,874 lên 0,882. Mức cải thiện khiêm tốn là hợp lý vì hai batch chia ngẫu nhiên từ cùng nguồn nên cùng phân phối, và với holdout chỉ 500 mẫu, chênh lệch 0,02 nằm gần biên độ dao động thống kê — không thể kết luận thêm dữ liệu luôn tốt hơn. Điều Bước 3 thực sự kiểm chứng là quy trình tự động chạy đúng: dữ liệu mới đi trọn vòng từ commit đến mô hình đang phục vụ, không cần thao tác thủ công.