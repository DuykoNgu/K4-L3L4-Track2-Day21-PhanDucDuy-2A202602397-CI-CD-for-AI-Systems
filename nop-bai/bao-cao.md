# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Phan Đức Duy |
| MSSV | 2A202602397 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/DuykoNgu/K4-L3L4-Track2-Day21-PhanDucDuy-2A202602397-CI-CD-for-AI-Systems |
| Ngày nộp | 07/10/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 100 | 0.2 | 3 | 0.7290 | 0.8840 |
| 2 | 200 | 0.1 | 5 | 0.7149 | 0.8740 |
| 3 | 100 | 0.1 | 3 | 0.7109 | 0.8780 |
| 4 | 50 | 0.05 | 2 | 0.6051 | 0.8460 |

**Bộ siêu tham số đã chọn:** `n_estimators=100`, `learning_rate=0.2`, `max_depth=3`.

**Lý do:** Bộ tham số này đạt điểm `f1_score` cao nhất (0.7290) vượt qua ngưỡng yêu cầu 0.65 của hệ thống, đồng thời đạt `accuracy` 0.8840. Khi so sánh giữa lần chạy 1 và lần chạy 2, mô hình với số cây ít hơn nhưng tốc độ học phù hợp mang lại khả năng tổng quát hóa tốt hơn trên tập holdout, tránh hiện tượng overfitting do cây quá sâu (`max_depth=5`). Giữa `n_estimators` và `learning_rate` tồn tại sự đánh đổi rõ rệt: giảm tốc độ học xuống 0.05 đòi hỏi số cây lớn hơn nhiều để bù đắp, nếu chỉ dùng 50 cây nông thì F1 giảm sâu còn 0.6051 và bị quality gate chặn lại. Lần chạy có F1 cao nhất cũng là lần có độ chính xác cao nhất trong các thí nghiệm được khảo sát.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Tập dữ liệu Adult có phân bố lớp mất cân bằng nghiêm trọng với chỉ khoảng 24.8% số mẫu thuộc lớp thu nhập cao (>50K) và 75.2% thuộc lớp thu nhập thấp. Trong bài toán này, nếu xây dựng một mô hình ngây thơ luôn dự đoán nhãn 0 (thu nhập thấp) cho mọi trường hợp, mô hình vẫn đạt độ chính xác accuracy lên đến 75.2%, một con số trông rất ấn tượng trên lý thuyết nhưng trên thực tế mô hình hoàn toàn vô dụng vì không thể phát hiện bất kỳ cá nhân thu nhập cao nào. Do đó, accuracy tạo ra cảm giác an toàn giả tạo và không thể dùng làm căn cứ kiểm soát chất lượng triển khai. Chỉ số F1 của lớp dương đo lường trực tiếp sự cân bằng giữa Precision và Recall đối với nhóm mục tiêu quan trọng. Khi tính toán F1, ta bắt buộc không sử dụng `average="weighted"` hay `average="macro"` vì các trọng số này sẽ bị lớp đa số chi phối kéo lên cao, làm mất đi ý nghĩa giám sát nghiêm ngặt của quality gate.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| MLflow không kết nối được SQLite | Thư viện SQLAlchemy 2.1+ loại bỏ pool tương thích với MLflow 2.13.0 | Cài đặt phiên bản tương thích `sqlalchemy<2.1.0` (2.0.54) |
| Cổng 5000 bị lỗi HTTP 403 khi mở MLflow UI | macOS AirPlay Receiver mặc định chiếm dụng cổng 5000 | Truy cập trực tiếp qua IP loopback `127.0.0.1:5000` thay vì localhost |
| Model chạy thử ban đầu bị dưới ngưỡng F1 0.65 | Bộ siêu tham số quá nhỏ (`n_est=50`, `depth=2`) không đủ năng lực phân loại | Tinh chỉnh siêu tham số qua MLflow lên `n_est=100, lr=0.2, depth=3` |

---

## 4. So Sánh Bước 2 và Bước 3 (bắt buộc, 2 - 3 câu)

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | 0.7290 | 0.8840 |
| Bước 3 (thêm `train_batch2`) | 0.7330 | 0.8820 |

**Nhận xét:** Khi bổ sung thêm 22.361 mẫu dữ liệu ở Bước 3, F1 tăng nhẹ từ 0.7290 lên 0.7330 trong khi accuracy gần như giữ nguyên (0.8840 so với 0.8820). Sự biến thiên nhỏ này là hoàn toàn hợp lý do hai batch dữ liệu được phân chia ngẫu nhiên từ cùng một tổng thể và có cùng phân phối xác suất. Điều quan trọng nhất được chứng minh ở Bước 3 là tính tự động hóa khép kín: hệ thống tự động kích hoạt huấn luyện lại và chuyển giao mô hình mới lên môi trường phục vụ mà không cần sự can thiệp thủ công.
