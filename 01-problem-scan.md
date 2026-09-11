# 01 — Problem Scan & Quick Cards (Vin Smart Future)

> **Người thực hiện:** Lâm Hoàng Phúc — AI Product Engineer (vai trò trong lab) tại Vin Smart Future
> **Lưu ý về số liệu:** Các con số thời gian/khối lượng dưới đây là **ước tính giả định** dựa trên quan sát trải nghiệm khách hàng và tài liệu công khai, dùng cho mục đích scoping trong lab. Cần xác minh lại bằng log vận hành thực tế trước khi ra quyết định đầu tư.

---

# 🔍 Phase 1 — SCAN

Sử dụng **4 Lenses**: Lặp lại (Repetitive) · Tốn thời gian (Time-consuming) · AI có thể tốt hơn (AI-upgrade) · Pain từ người khác (Stakeholder Pain).

| # | Subsidiary | Lens | Mô tả ngắn bài toán |
|---|------------|------|---------------------|
| 1 | **VinFast** | AI có thể tốt hơn | Cố vấn dịch vụ tại xưởng phải tự "dịch" mô tả lỗi bằng tiếng Việt đời thường của khách (*"xe qua gờ giảm tốc kêu cụp cụp ở bánh trước"*, *"xe ì, đạp ga không bốc"*) thành nhóm lỗi kỹ thuật để tạo lệnh sửa chữa. Mất ~20-25 phút/ticket, hay xếp nhầm khoang/kỹ thuật viên. |
| 2 | **Vinhomes** | Lặp lại | Ban quản lý phân loại thủ công hàng trăm phản ánh/ngày trên App Vinhomes Resident (mất nước, hỏng đèn hành lang, ồn ào, vệ sinh) rồi chuyển đến đúng bộ phận kỹ thuật từng tòa. |
| 3 | **Xanh SM** | Pain từ người khác | Tài xế phàn nàn điểm đón không chính xác vì khách ghi chú địa chỉ tự do (*"cổng sau S2.05, cạnh Circle K"*) trong khi ghim GPS lệch; tài xế phải gọi điện hỏi lại, mất 3-5 phút/cuốc và tăng tỉ lệ hủy chuyến. |
| 4 | **Vinpearl** | Tốn thời gian | Nhân viên sales đọc email đặt phòng theo đoàn (group booking) dài, không theo mẫu từ công ty lữ hành, trích xuất số khách/loại phòng/ngày ở, kiểm tra quỹ phòng rồi soạn báo giá — ~40 phút/email. |
| 5 | **Vinmec** | Tốn thời gian | Bác sĩ soạn bản tóm tắt xuất viện từ bệnh án điện tử, kết quả xét nghiệm và ghi chú điều trị — 20-30 phút/bệnh nhân vào cuối ca trực. |
| 6 | **VinFast** | Lặp lại | Kế toán đối chiếu hằng tuần dữ liệu phiên sạc từ trụ sạc đối tác với hóa đơn gửi về (lệch mã trụ, lệch kWh). |

**Ghi chú tự phản biện:** Bài toán #6 là đối chiếu dữ liệu có cấu trúc (mã trụ, kWh, thời gian) → **rule-based/SQL matching** giải quyết tốt hơn và rẻ hơn LLM, nên không chọn vào top 3. Bài toán #5 có giá trị cao nhưng rủi ro y khoa rất lớn, cần dữ liệu bệnh án được cấp quyền — không phù hợp trong phạm vi lab.

---

# 🃏 Phase 2 — QUICK-ASSESS: 3 Quick Problem Cards

Chọn top 3: **#1 (VinFast chẩn đoán sơ bộ)**, **#4 (Vinpearl group booking)**, **#3 (Xanh SM điểm đón)**.

## Card #1 — VinFast: Phân loại sơ bộ lỗi xe từ mô tả tiếng Việt của khách

| Trường | Nội dung |
|---|---|
| **Bài toán (1 câu)** | Chuyển mô tả lỗi tự do bằng tiếng Việt của khách thành nhóm lỗi kỹ thuật + lệnh sửa chữa (RO) sơ bộ để xếp đúng khoang và kỹ thuật viên ngay lần đầu. |
| **Công ty thành viên** | [x] VinFast &nbsp; [ ] Xanh SM &nbsp; [ ] Vinhomes &nbsp; [ ] Vinmec &nbsp; [ ] Khác |
| **Ai đang đau (Actor)?** | Cố vấn dịch vụ (Service Advisor) tại xưởng — quá tải giờ cao điểm; khách hàng — chờ lâu, phải kể lại lỗi nhiều lần; kỹ thuật viên — nhận xe sai chuyên môn. |
| **Workflow thủ công (5 bước)** | 1. Tổng đài/App nhận lời kể của khách → 2. Cố vấn dịch vụ đọc ticket & gọi lại hỏi thêm → 3. Tra tài liệu kỹ thuật + lịch sử xe trên DMS để đoán nhóm lỗi → 4. Tạo RO sơ bộ, chọn khoang, đặt lịch → 5. Kỹ thuật viên kiểm tra thực tế |
| **Bước tốn thời gian/lỗi nhất** | Bước 2-3 (⏱ ~18 phút/lượt); đoán sai nhóm lỗi ở bước 3 gây đổi khoang/đặt lại lịch (ước tính ~25% ca). |
| **AI hỗ trợ ở bước nào?** | Bước 2-3: LLM trích xuất triệu chứng, gợi ý **top-3 nhóm lỗi kèm độ tự tin**, sinh sẵn câu hỏi làm rõ và soạn nháp RO để cố vấn duyệt. |
| **Metric có số** | Giảm thời gian từ tiếp nhận → RO sơ bộ từ **~27 phút → ≤ 10 phút**; độ chính xác top-3 nhóm lỗi **≥ 95%**; tỉ lệ đổi khoang do xếp sai **25% → ≤ 10%**. |
| **Quick Architecture** | [ ] No AI &nbsp; [x] Rule (lớp chặn từ khóa an toàn) &nbsp; [x] LLM &nbsp; [ ] Agent |

## Card #2 — Vinpearl: Trích xuất email đặt phòng theo đoàn & soạn nháp báo giá

| Trường | Nội dung |
|---|---|
| **Bài toán (1 câu)** | Email group booking của công ty lữ hành dài, không theo mẫu, khiến sales mất nhiều thời gian trích xuất yêu cầu và phản hồi báo giá chậm. |
| **Công ty thành viên** | [ ] VinFast &nbsp; [ ] Xanh SM &nbsp; [ ] Vinhomes &nbsp; [ ] Vinmec &nbsp; [x] Khác: **Vinpearl** |
| **Ai đang đau (Actor)?** | Nhân viên Sales/Reservation khách đoàn; đối tác lữ hành chờ báo giá (mất deal nếu phản hồi chậm hơn đối thủ). |
| **Workflow thủ công (4 bước)** | 1. Đọc email + file đính kèm (danh sách đoàn) → 2. Chép tay số khách, loại phòng, ngày in/out, yêu cầu đặc biệt vào Excel → 3. Kiểm tra quỹ phòng trên PMS → 4. Soạn email báo giá/đề xuất phương án |
| **Bước tốn thời gian/lỗi nhất** | Bước 2 (⏱ ~15 phút/email), dễ nhầm ngày và số phòng khi email có nhiều lần chỉnh sửa trong cùng thread. |
| **AI hỗ trợ ở bước nào?** | Bước 2 (LLM trích xuất ra JSON có cấu trúc) và bước 4 (soạn nháp email). Bước 3 kiểm tra quỹ phòng giữ nguyên bằng API PMS (rule). |
| **Metric có số** | Thời gian phản hồi báo giá đầu tiên từ **~40 phút → ≤ 10 phút/email**; độ chính xác trích xuất trường ngày & số phòng **≥ 98%**. |
| **Quick Architecture** | [ ] No AI &nbsp; [ ] Rule &nbsp; [x] LLM &nbsp; [ ] Agent |

## Card #3 — Xanh SM: Chuẩn hóa điểm đón từ ghi chú tự do của khách

| Trường | Nội dung |
|---|---|
| **Bài toán (1 câu)** | Ghi chú điểm đón tự do + ghim GPS lệch khiến tài xế không tìm được khách, phải gọi điện hỏi lại và tăng hủy chuyến. |
| **Công ty thành viên** | [ ] VinFast &nbsp; [x] Xanh SM &nbsp; [ ] Vinhomes &nbsp; [ ] Vinmec &nbsp; [ ] Khác |
| **Ai đang đau (Actor)?** | Tài xế Xanh SM (mất thời gian, bị đánh giá thấp); khách hàng (chờ lâu). |
| **Workflow thủ công (4 bước)** | 1. Khách đặt xe, ghi chú điểm đón tự do → 2. Tài xế đọc ghi chú khi đang lái → 3. Tài xế gọi điện xác nhận vị trí → 4. Di chuyển lại đến điểm đón thật |
| **Bước tốn thời gian/lỗi nhất** | Bước 3-4 (⏱ ~3-5 phút/cuốc có ghi chú mơ hồ). |
| **AI hỗ trợ ở bước nào?** | Trước bước 2: LLM kết hợp ghi chú + dữ liệu POI khu đô thị (tòa, cổng, sảnh) để đề xuất một điểm đón chuẩn hóa ngắn gọn cho tài xế; khách bấm xác nhận. |
| **Metric có số** | Giảm tỉ lệ cuốc phải gọi điện xác nhận điểm đón **từ ~30% → ≤ 15%**; giảm thời gian đón trung bình **1-2 phút/cuốc**. |
| **Quick Architecture** | [ ] No AI &nbsp; [x] Rule (geofence POI) &nbsp; [x] LLM &nbsp; [ ] Agent |

---

## 🗳️ Đề xuất mang vào Deep-Dive nhóm

Đề xuất **Card #1 — VinFast phân loại sơ bộ lỗi xe**, vì:
* Bài toán ngôn ngữ tự nhiên thực sự (tiếng lóng, mô tả cảm giác) — rule/keyword thuần không xử lý tốt → **LLM có lợi thế rõ ràng**.
* Có ranh giới an toàn cần thiết kế cẩn thận (lỗi phanh, pin cao áp) → phù hợp tiêu chí Operational Boundary của lab.
* Có sẵn "nhãn thật" để đo lường: kết quả kiểm tra của kỹ thuật viên ở bước 5.

Lý do chưa chọn các card còn lại:
* **Card #2 (Vinpearl):** Giá trị tốt nhưng phụ thuộc tích hợp PMS; phần AI chủ yếu là trích xuất → có thể làm nhanh sau khi có kết quả Card #1.
* **Card #3 (Xanh SM):** Phụ thuộc dữ liệu POI chi tiết từng tòa/cổng — nếu chưa có thì LLM sẽ "bịa" vị trí; nên xây dữ liệu POI (rule) trước.

---

> **🤖 Stress-test bằng AI (Phase 2 TIP):** Nhờ AI trợ lý (Claude) phản biện Card #1 theo vai CFO & Trưởng phòng vận hành. Ba điểm yếu được chỉ ra và cách xử lý:
> 1. *"Metric 27 phút chưa có baseline đo thực tế"* → Đã ghi rõ là ước tính và đưa việc đo baseline vào điều kiện của quyết định ở báo cáo Deep-Dive.
> 2. *"Rule-based keyword có thể bắt được phần lớn ca lỗi phổ biến"* → Đồng ý một phần: dùng rule cho **lớp chặn an toàn** (phanh, khói, mùi khét, cảnh báo pin), LLM chỉ cho phần mô tả mơ hồ còn lại.
> 3. *"Nếu AI xếp sai khoang thì chi phí rework không giảm"* → Thêm metric tỉ lệ đổi khoang và bắt buộc cố vấn dịch vụ duyệt (HITL).
