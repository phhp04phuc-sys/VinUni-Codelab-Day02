# 02 — Deep-Dive Report: Điều phối sạc thời gian thực cho đội xe Xanh SM

**Nhóm:** DILO
**Thành viên:** Phan Hoàng Phúc (trưởng nhóm, phhp04phuc@gmail.com); Nguyễn Duy Hưng (email: cần bổ sung)

## Mức độ kiểm chứng

| Thông tin | Trạng thái |
| --- | --- |
| Màn hình xe hiện % pin và có bản đồ trạm sạc | Có nguồn báo chí, trưởng nhóm xác nhận |
| Có công cụ xem cổng trống | Chỉ có nguồn bên thứ ba (MoMo), chưa xác nhận trong app chính thức |
| Đặt chỗ trước tại trạm | Chưa tìm thấy nguồn |
| Trụ sạc kín giờ cao điểm | Báo Vietnamnet ghi nhận 10-13h, 16-19h tại một TTTM Hà Nội, bài cũ |
| Tài xế Xanh SM tới trạm rồi phải chờ, tần suất bao nhiêu | **Chưa có số liệu**, cần khảo sát tài xế |
| Xe sạc di động trong Rule 2 | Lấy theo đề lab, chưa kiểm chứng ở vận hành thật |

## 3.1. Current-State Workflow
Xem `04-workflow-diagram.png`. Thời gian từng bước **cần đo**.

| Bước | Ai | Ghi chú |
|---|---|---|
| 1. Xem % pin trên màn hình xe | Tài xế | Tự quyết định khi nào sạc |
| 2. Tìm trạm gần nhất trên bản đồ/app | Tài xế | 🔄 Handoff: tài xế → hệ thống bản đồ |
| 3. Lái tới trạm | Tài xế | Không chạy cuốc; trạng thái có thể đổi trên đường |
| 4. Tới nơi: có cổng thì sạc; đông thì chờ hoặc tìm trạm khác | Tài xế | 🔴 Bottleneck: chỉ biết khi đã tới |
| 5. Sạc, rồi rời cổng | Tài xế | 🔄 Handoff: tài xế → trụ sạc; đỗ miễn phí 10 phút |

**Tổng cộng = cần đo** (phút từ lúc quyết định sạc đến lúc cắm sạc).

## 3.2. Problem Statement (6-field)

| Field | Nội dung |
|---|---|
| **1. Actor** | Tài xế Xanh SM; bộ phận vận hành đội xe |
| **2. Current Workflow** | Tài xế xem pin trên màn hình, tự tìm trạm gần nhất, lái tới, gặp đông thì chờ hoặc đổi trạm |
| **3. Bottleneck** | Mỗi xe quyết định riêng lẻ. Bản đồ cho biết trạm gần, nhưng không dự báo chỗ trống lúc xe tới và không tính các xe khác đang tới cùng trạm |
| **4. Business Impact** | Mỗi phút chờ sạc là phút xe không chạy cuốc, cộng với rủi ro xe xuống pin thấp. Mức tổn thất thật: cần đo từ log |
| **5. Success Metric** | Mục tiêu đề xuất: giảm 30% thời gian từ quyết định sạc đến cắm sạc; giảm 50% ca tới trạm phải đổi trạm; 100% xe dưới 5% pin được cảnh báo; không tin nào tới tài xế mà chưa có người duyệt ở giai đoạn thử nghiệm |
| **6. Operational Boundary** | AI **được** dự báo, tổng hợp, soạn đề xuất. AI **không được** tự gửi tin, tự điều xe, hay chỉ đến trạm xa hơn 5km khi pin dưới 5%. **Cần duyệt:** tin gửi tài xế và lệnh điều xe sạc di động |

## 3.3. Future-State Flow & AI Fit

**AI-Fit:** [x] Rule / State-Machine [x] LLM Feature [ ] Agentic Loop

| Thành phần | Cách làm | Lý do |
|---|---|---|
| Máy chủ tổng: pin, vị trí xe, trạng thái cổng trạm | Luồng sự kiện real-time, không AI | Là dữ liệu, phải chính xác |
| Dự báo xe sắp cần sạc | Rule trước; mô hình dự báo khi đủ dữ liệu | Giải thích được |
| Dự báo chỗ trống lúc xe tới | Tính từ cổng trống, xe đang sạc, xe đang tới | Là phép tính, không cần LLM |
| Chọn trạm | Rule: trong tầm pin, còn chỗ, ràng buộc 5km | An toàn phải là code cứng |
| Soạn tin cho tài xế | LLM | Ngôn ngữ tự nhiên, theo tình huống |

Không chọn Agentic Loop vì không cần AI tự hành động, và lỗi có thể gây nguy hiểm giao thông.

**Flow tương lai:**
1. Xe và trạm liên tục gửi trạng thái về máy chủ tổng.
2. Rule phát hiện xe sắp cần sạc, tính trạm còn chỗ khi xe tới.
3. 🔵 **AI Step:** LLM soạn đề xuất có `[DRAFT_ONLY]`, hoặc JSON `dispatch_mobile_charger` nếu pin dưới 5%.
4. Code kiểm tra đầu ra: có thẻ, không vi phạm 5km.
5. 🟢 **Human Step:** nhân viên vận hành duyệt hoặc sửa.
6. Đề xuất hiện cho tài xế; hệ thống tính xe này vào số xe đang tới trạm.

↩️ **Fallback:** LLM lỗi thì dùng mẫu tin. Dữ liệu trạm cũ hoặc mất kết nối thì đánh dấu "chưa xác nhận", tài xế tự tìm trạm như hiện nay.

## Phase 4 — Prompt Prototype
Code nằm ở nhánh cá nhân. 3 test tấn công: pin 2% ép đi trạm 8km, ép bỏ `[DRAFT_ONLY]`, giả danh quản trị viên.
**Kết quả chạy thật:** cần điền sau khi chạy với Gemini.

## Phase 5 — Evaluate

1. [ ] Có sẵn dữ liệu mẫu sạch? **Chưa**, nhóm không có log thật.
2. [x] Rủi ro khi AI sai nằm trong tầm kiểm soát? **Có**, nhờ người duyệt, kiểm tra bằng code và fallback.
3. [ ] Stakeholders sẵn sàng đổi quy trình? **Chưa xác nhận**.

**Quyết định:** [x] **NOT YET**

**Justification:** Chưa đo được thời gian chờ và tần suất đổi trạm, nên chưa chứng minh được giá trị; cũng chưa xác nhận có API trạng thái cổng real-time. Bước đầu là khảo sát tài xế và lấy baseline, sau đó thử ở một khu vực.

## Nguồn
- [Bản đồ trạm trên màn hình xe](https://doanhnhan.baophapluat.vn/xuyen-viet-bang-xe-dien-vinfast-qua-nhan-vi-tram-sac-de-tim-nap-pin-ngay-cang-nhanh.html)
- [MoMo: tra cứu cổng trống](https://www.momo.vn/tien-ich-giao-thong/tram-sac/vinfast)
- [Đỗ miễn phí 10 phút, ưu đãi tài xế](https://vnbusiness.vn/vinfast-rut-ngan-thoi-gian-mien-phi-do-sau-sac.html)
- [Trụ sạc kín giờ cao điểm, quy định tranh trụ](https://vietnamnet.vn/xe-xang-hang-quan-chiem-cho-tram-sac-xe-dien-ha-noi-2169566.html)
