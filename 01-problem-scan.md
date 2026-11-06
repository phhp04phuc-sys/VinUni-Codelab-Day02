# 01 — Problem Scan & Quick Assess

**Nhóm:** DILO
**Thành viên:** Phan Hoàng Phúc (trưởng nhóm, phhp04phuc@gmail.com); Nguyễn Duy Hưng (email: cần bổ sung)

> Mọi con số trong tài liệu này là **giả định để scoping**, chưa phải dữ liệu thật của Vingroup.

## Phase 1 — SCAN

| # | Subsidiary | Lens | Mô tả ngắn bài toán |
|---|---|---|---|
| 1 | Xanh SM | Pain từ người khác | Tài xế chỉ biết sắp hết pin khi đã nguy cấp, tổng đài xử lý sau khi sự cố xảy ra |
| 2 | Xanh SM / V-Green | Tốn thời gian | Tài xế tới trạm mới biết hết chỗ, phải xếp hàng hoặc chạy sang trạm khác |
| 3 | Xanh SM | Lặp lại | Điều phối viên gọi điện từng trạm để xác nhận tình trạng cổng sạc |
| 4 | VinFast | Lặp lại | So khớp hoá đơn sạc với lịch sử phiên sạc thủ công |
| 5 | Vinhomes | AI có thể tốt hơn | Phản hồi cư dân về sự cố trạm sạc trong khu đô thị còn chậm và rập khuôn |

## Phase 2 — 3 Quick Problem Cards

### Card #1 — Điều phối pin toàn đội xe theo thời gian thực (bài toán được chọn)
- **Bài toán:** Không có một màn hình chung cho biết xe nào sắp hết pin và trạm nào còn chỗ, nên đội xe chỉ phản ứng sau khi sự cố xảy ra.
- **Công ty:** Xanh SM (phối hợp dữ liệu trạm sạc V-Green)
- **Actor:** Điều phối viên, tài xế
- **Workflow hiện tại:** 1. Tài xế thấy pin thấp → 2. Gọi tổng đài → 3. Điều phối viên tra trạm, gọi xác nhận → 4. Nhắn chỉ đường hoặc gọi xe cứu hộ
- **Bước tệ nhất:** Bước 3 (⏱ 8 phút/lượt, giả định)
- **AI nhảy vào:** Dự báo nguy cơ, tổng hợp trạng thái, soạn đề xuất cho người duyệt
- **Metric:** Giảm thời gian xử lý từ 15 phút xuống dưới 3 phút; giảm 50% ca xe cạn pin giữa đường
- **Quick Architecture:** [x] Rule [x] LLM [ ] Agent

### Card #2 — Giờ cao điểm hàng chờ trạm sạc
- **Bài toán:** Nhiều xe cùng sạc giờ cao điểm làm tăng thời gian chết.
- **Công ty:** Xanh SM
- **Actor:** Tài xế
- **Workflow hiện tại:** 1. Tự chọn trạm → 2. Tới nơi → 3. Xếp hàng → 4. Sạc
- **Bước tệ nhất:** Xếp hàng (⏱ 20 phút/lượt, giả định)
- **AI nhảy vào:** Không bắt buộc; thuật toán phân bổ đủ dùng
- **Metric:** Giảm thời gian chờ trung bình 30%
- **Quick Architecture:** [x] Rule

### Card #3 — Đối soát hoá đơn sạc
- **Bài toán:** Đối soát thủ công chênh lệch hoá đơn sạc.
- **Công ty:** VinFast
- **Actor:** Nhân viên kế toán
- **Workflow hiện tại:** 1. Xuất log → 2. So hoá đơn → 3. Tìm chênh lệch → 4. Liên hệ xử lý
- **Bước tệ nhất:** Tìm chênh lệch (⏱ 10 phút/lượt, giả định)
- **AI nhảy vào:** Giải thích chênh lệch bất thường
- **Metric:** 90% hoá đơn tự đối soát dưới 1 phút
- **Quick Architecture:** [x] Rule [x] LLM

## Lựa chọn của nhóm
Chọn **Card #1**, gộp cả ý của Card #2 ở mức hiển thị chỗ trống trạm. Đây là bài có rủi ro an toàn cao nhất nên cần ranh giới rõ, và khớp với prototype prompt của nhóm.
