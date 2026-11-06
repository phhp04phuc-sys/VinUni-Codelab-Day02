# 01 — Problem Scan & Quick Assess

**Nhóm:** DILO
**Thành viên:** Phan Hoàng Phúc (trưởng nhóm, phhp04phuc@gmail.com); Nguyễn Duy Hưng (email: cần bổ sung)

> Nhóm chưa có dữ liệu nội bộ. Thời gian và tổn thất thật để **"cần đo"**; các con số mục tiêu là đề xuất, không phải số liệu đã xảy ra.

## Phase 1 — SCAN

| # | Subsidiary | Lens | Mô tả ngắn bài toán |
|---|---|---|---|
| 1 | Xanh SM / V-Green | Pain từ người khác | Màn hình xe không hiện số cổng trống, không đặt chỗ được; nhiều tài xế có thể cùng tới một trạm |
| 2 | Xanh SM | Tốn thời gian | Thời gian tài xế tới trạm rồi phải chờ hoặc đổi trạm là thời gian không chạy cuốc |
| 3 | V-Green | AI có thể tốt hơn | Tài xế để xe sạc rồi đi ăn, ngủ; sạc xong xe vẫn chiếm cổng |
| 4 | Xanh SM | Lặp lại | Tài xế tự cân nhắc nhận thêm cuốc hay đi sạc, dễ quyết định muộn |
| 5 | Vinhomes | Pain từ người khác | Trụ sạc chung cư bị chiếm chỗ làm cư dân khó sạc giờ cao điểm |

## Phase 2 — 3 Quick Problem Cards

### Card #1 — Điều phối sạc toàn đội xe theo thời gian thực (được chọn)
- **Bài toán:** Màn hình xe hiện trạm sạc nhưng không có số cổng trống và không đặt chỗ được, nên tài xế tới nơi mới biết trạm kín; mỗi lần sạc khoảng 1 tiếng nên chờ rất tốn thời gian.
- **Công ty:** Xanh SM, phối hợp V-Green
- **Actor:** Tài xế; bộ phận vận hành đội xe
- **Workflow hiện tại:** 1. Xem % pin trên màn hình → 2. Tìm trạm gần nhất trên bản đồ/app → 3. Lái tới → 4. Nếu đông thì chờ hoặc tìm trạm khác → 5. Sạc khoảng 1 tiếng
- **Bước tệ nhất:** Bước 4 (⏱ cần đo)
- **AI nhảy vào:** Dự báo pin theo lộ trình và cuốc, dự báo chỗ trống, soạn đề xuất
- **Metric mục tiêu:** Giảm 30% thời gian từ lúc quyết định sạc đến lúc cắm sạc
- **Quick Architecture:** [x] Rule [x] LLM [ ] Agent

### Card #2 — Giảm xe chiếm cổng sau khi sạc xong
- **Bài toán:** Xe đỗ quá giờ làm giảm số cổng khả dụng.
- **Công ty:** V-Green
- **Actor:** Tài xế, khách hàng chờ sạc
- **Workflow hiện tại:** 1. Sạc xong → 2. Hết 10 phút miễn phí → 3. Tính phí quá giờ
- **Bước tệ nhất:** Bước 3 (⏱ cần đo)
- **AI nhảy vào:** Không cần; thông báo theo rule là đủ
- **Metric mục tiêu:** Giảm 30% phút đỗ quá giờ
- **Quick Architecture:** [x] Rule

### Card #3 — Trợ lý quyết định nhận cuốc hay đi sạc
- **Bài toán:** Tài xế nhận cuốc dài khi pin không đủ để về trạm thuận tiện.
- **Công ty:** Xanh SM
- **Actor:** Tài xế
- **Workflow hiện tại:** 1. Nhận đề xuất cuốc → 2. Tự ước lượng pin → 3. Quyết định
- **Bước tệ nhất:** Bước 2 (⏱ cần đo)
- **AI nhảy vào:** Dự báo pin sau cuốc
- **Metric mục tiêu:** Giảm 50% ca xuống dưới 5% pin khi đang chạy
- **Quick Architecture:** [x] Rule [x] LLM

## Lựa chọn của nhóm
Chọn **Card #1**, gồm cả dự báo của Card #3: gộp dữ liệu xe, trạm và chỗ trống real-time vào một máy chủ chung.

## Nguồn kiểm chứng
- [Trải nghiệm xuyên Việt: bản đồ trạm sạc trên màn hình xe](https://doanhnhan.baophapluat.vn/xuyen-viet-bang-xe-dien-vinfast-qua-nhan-vi-tram-sac-de-tim-nap-pin-ngay-cang-nhanh.html)
- [MoMo: tra cứu trạm và cổng trống](https://www.momo.vn/tien-ich-giao-thong/tram-sac/vinfast) (bên thứ ba)
- [VnBusiness: phí đỗ sau sạc, ưu đãi tài xế](https://vnbusiness.vn/vinfast-rut-ngan-thoi-gian-mien-phi-do-sau-sac.html)
