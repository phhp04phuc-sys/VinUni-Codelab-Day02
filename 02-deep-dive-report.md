# 02 — Deep-Dive Report: Điều phối pin thời gian thực cho Xanh SM

**Nhóm:** DILO
**Thành viên:** Phan Hoàng Phúc (trưởng nhóm, phhp04phuc@gmail.com); Nguyễn Duy Hưng (email: cần bổ sung)

> Mọi con số là **giả định để scoping**. Baseline thật cần đo từ log vận hành.

## 3.1. Current-State Workflow
Xem `04-workflow-diagram.png`.

| Bước | Ai | Thời gian | Ghi chú |
|---|---|---:|---|
| 1. Thấy pin thấp, gọi tổng đài | Tài xế | 3 phút | 🔄 Handoff tài xế → tổng đài |
| 2. Hỏi vị trí, mức pin | Tổng đài | 2 phút | Nhập tay |
| 3. Tra trạm, gọi xác nhận chỗ trống | Điều phối viên | 8 phút | 🔴 Bottleneck: dữ liệu rời rạc |
| 4. Soạn chỉ đường hoặc gọi xe sạc di động | Điều phối viên | 2 phút | 🔄 Handoff điều phối → tài xế |

**Tổng cộng = 15 phút/lượt**, chỉ bắt đầu sau khi tài xế gọi.

## 3.2. Problem Statement (6-field)

| Field | Nội dung |
|---|---|
| **1. Actor** | Điều phối viên Xanh SM; tài xế xe điện |
| **2. Current Workflow** | Tài xế gọi tổng đài, điều phối viên tra nhiều hệ thống và gọi trạm để xác nhận, rồi nhắn chỉ đường |
| **3. Bottleneck** | Không có nơi chung cho biết pin từng xe, tình trạng trạm và chỗ trống; xử lý sau khi sự cố xảy ra |
| **4. Business Impact** | Giả định 40 ca/ngày × 15 phút = 10 giờ điều phối/ngày; xe cạn pin giữa đường gây mất cuốc và rủi ro giao thông |
| **5. Success Metric** | Cảnh báo trước khi xe xuống dưới 5% ở ≥ 80% ca; thời gian xử lý < 3 phút; giảm 50% ca cạn pin giữa đường; 100% tin gửi tài xế có người duyệt |
| **6. Operational Boundary** | AI **được** dự báo, tổng hợp, soạn đề xuất. AI **không được** tự gửi tin, tự điều xe, hay chỉ đến trạm xa hơn 5km khi pin dưới 5%. **Cần duyệt:** mọi tin gửi tài xế và lệnh điều xe sạc di động |

## 3.3. Future-State Flow & AI Fit

**AI-Fit:** [x] Rule / State-Machine [x] LLM Feature [ ] Agentic Loop

| Thành phần | Cách làm | Lý do |
|---|---|---|
| Máy chủ tổng: pin, vị trí xe, trạng thái trạm, chỗ trống | Luồng sự kiện real-time, không AI | Là dữ liệu, phải chính xác |
| Dự báo xe sắp hết pin | Rule trước (pin, quãng đường còn lại); mô hình dự báo khi đủ dữ liệu | Rule giải thích được, không cần LLM |
| Chọn trạm khả thi | Rule: trong tầm pin, còn chỗ, có ràng buộc 5km | Ràng buộc an toàn phải là code cứng |
| Soạn tin cho tài xế | LLM | Ngôn ngữ tự nhiên, theo tình huống |

Không chọn Agentic Loop vì hệ thống không cần AI tự hành động, và lỗi có thể gây nguy hiểm giao thông.

**Flow tương lai:**
1. Xe và trạm liên tục gửi trạng thái về máy chủ tổng.
2. Rule phát hiện xe có nguy cơ, tính trạm còn chỗ trong tầm pin.
3. 🔵 **AI Step:** LLM soạn đề xuất có `[DRAFT_ONLY]`, hoặc lệnh JSON `dispatch_mobile_charger` nếu pin dưới 5%.
4. Kiểm tra đầu ra bằng code: có thẻ, không vi phạm 5km.
5. 🟢 **Human Step:** điều phối viên duyệt hoặc sửa.
6. Gửi tài xế, giữ chỗ trạm, theo dõi tới khi xe sạc.

↩️ **Fallback:** LLM lỗi hoặc đầu ra sai định dạng thì dùng mẫu tin có sẵn. Dữ liệu trạm cũ hoặc mất kết nối thì đánh dấu "chưa xác nhận" và quay về gọi điện như quy trình cũ.

## Phase 4 — Prompt Prototype
Code nằm ở nhánh cá nhân. 3 test tấn công: pin 2% ép đi trạm 8km, ép bỏ `[DRAFT_ONLY]`, giả danh quản trị viên.
**Kết quả chạy thật:** cần điền sau khi chạy với Gemini.

## Phase 5 — Evaluate

1. [ ] Có sẵn dữ liệu mẫu sạch để test? **Chưa**, nhóm không có log thật.
2. [x] Rủi ro khi AI sai nằm trong tầm kiểm soát? **Có**, nhờ người duyệt, kiểm tra bằng code và fallback.
3. [ ] Stakeholders sẵn sàng đổi quy trình? **Chưa xác nhận**.

**Quyết định:** [x] **NOT YET**

**Justification:** Giá trị tiềm năng rõ, rủi ro kiểm soát được, nhưng chưa có dữ liệu real-time và baseline thật để chứng minh metric. Nên làm trước phần máy chủ tổng và rule, đo baseline 2-4 tuần, chạy thử ở một khu vực rồi mới đánh giá lại phần LLM.
