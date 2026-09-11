<!--
Tên nhóm: safe-drive
Thành viên:
- Nguyễn Đức Phát: nguyenducphat.edu@gmail.com
- Đỗ Thành Đạt: dothanhdat10x@gmail.com
- Nguyễn Ngọc Minh: nnminh433@gmail.com
- Phạm Quang Đạt: phamqdat99@gmail.com
- Lâm Hoàng Phúc: lamhoangphuc2003st@gmail.com
- Chử Trần Phương Nam: tranphuongnam932004@gmail.com
-->

# 🔍 Phase 1 — SCAN & Phase 2 — QUICK-ASSESS: Vin Smart Future Problem Discovery

---

## 🏛️ Thông tin Nhóm & Dự án
* **Tên nhóm:** safe-drive
* **Thành viên:**
  1. Nguyễn Đức Phát (nguyenducphat.edu@gmail.com)
  2. Đỗ Thành Đạt (dothanhdat10x@gmail.com)
  3. Nguyễn Ngọc Minh (nnminh433@gmail.com)
  4. Phạm Quang Đạt (phamqdat99@gmail.com)
  5. Lâm Hoàng Phúc (lamhoangphuc2003st@gmail.com)
  6. Chử Trần Phương Nam (tranphuongnam932004@gmail.com)
* **Đơn vị:** Vin Smart Future (Vingroup Technology Division)
* **Dự án:** Problem Scan & Initial Scoping for Vingroup Ecosystem

---

# 🔍 Phase 1 — SCAN: Danh sách 5 Bài toán Vận hành Vingroup

Dưới đây là danh sách 5 bài toán và bottleneck thực tế được quét qua các công ty thành viên Vingroup sử dụng **4 Lenses** (Repetitive, Time-consuming, AI-upgrade, Stakeholder Pain):

| # | Subsidiary (Công ty thành viên) | Lens | Mô tả ngắn bài toán & Bottleneck |
|---|----------------------------------|------|-----------------------------------|
| 1 | **Xanh SM (GSM)** | Tốn thời gian / Pain từ người khác | Điều phối viên xử lý thủ công các báo cáo sự cố cạn pin/hết pin thực địa của tài xế, tra cứu trụ sạc trống và gọi cứu hộ (mất 15-20 phút/lượt, gây rò rỉ doanh thu). |
| 2 | **Vinhomes** | Time-consuming / AI-upgrade | BQL Vinhomes phản hồi khiếu nại/góp ý của cư dân trên App Vinhomes Resident thủ công, phản hồi rập khuôn hoặc chậm trễ (mất 12-24 giờ để xử lý 1 phiếu). |
| 3 | **Vinmec** | Time-consuming / Repetitive | Bác sĩ và y tá mất quá nhiều thời gian viết tóm tắt hồ sơ xuất viện (Discharge Summary) từ hàng chục kết quả xét nghiệm và ghi chú lâm sàng (mất 20-30 phút/bệnh nhân). |
| 4 | **VinFast** | Repetitive / AI-upgrade | So khớp hóa đơn sạc điện tại các trạm sạc đối tác công cộng hằng tuần, phát hiện sai lệch chỉ số kWh và đơn giá thủ công. |
| 5 | **Vinpearl / VinWonders** | AI-upgrade / Stakeholder Pain | Chatbot CSKH Vinpearl hỗ trợ du khách đặt vé vui chơi, đặt phòng và tư vấn lịch trình còn rập khuôn, không hiểu ngữ cảnh tiếng Việt tự nhiên phức tạp. |

---

# 🃏 Phase 2 — QUICK-ASSESS: 3 Quick Problem Cards

## 🃏 QUICK PROBLEM CARD #1 — Xanh SM: Xử lý sự cố sạc pin thực địa

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #1                                                       │
│                                                                             │
│ Bài toán: Tài xế Xanh SM báo cáo sự cố hết pin/pin dưới 5% cần điều phối     │
│ trạm sạc gần nhất hoặc điều xe cứu hộ sạc pin di động.                      │
│ Công ty thành viên: [x] Xanh SM (GSM)                                       │
│                                                                             │
│ Ai đang đau (Actor)? Tài xế (chờ đợi trên đường), Điều phối viên (quá tải)  │
│                                                                             │
│ Workflow thủ công hiện tại (5 bước):                                        │
│   1. Tài xế gọi tổng đài điều vận báo nguy cơ cạn pin                       │
│   ──> 2. Điều phối viên tra cứu thủ công vị trí GPS xe trên bản đồ          │
│   ──> 3. Tra cứu danh sách trạm sạc VinFast còn trụ trống phù hợp           │
│   ──> 4. Viết tin nhắn SMS/In-app hướng dẫn gửi tài xế                      │
│   ──> 5. Điều xe cứu hộ sạc di động nếu pin cạn < 5%                        │
│                                                                             │
│ Bước nào tốn thời gian/lỗi nhất? Bước 3 & 4 (⏱ 10-12 phút/lượt)              │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 3 & 4 (Auto-fetch & Draft SMS)   │
│                                                                             │
│ Đo thành công bằng gì (Metric có số)?                                       │
│   Giảm thời gian xử lý sự cố từ 15 phút ──> dưới 3 phút/lượt.                │
│   Tỉ lệ chỉ dẫn chính xác loại trụ sạc đạt ≥ 98%.                           │
│                                                                             │
│ Quick Architecture: [x] LLM Feature (Draft SMS + Guardrail Rules)           │
│ Quick Verdict: [x] GO                                                       │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 🃏 QUICK PROBLEM CARD #2 — Vinhomes: Tự động phân loại & soạn draft phản hồi khiếu nại cư dân

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #2                                                       │
│                                                                             │
│ Bài toán: BQL Vinhomes tiếp nhận hàng trăm phản hồi/khiếu nại cư dân        │
│ qua App Vinhomes Resident nhưng phân loại thủ công và trả lời rập khuôn.    │
│ Công ty thành viên: [x] Vinhomes                                            │
│                                                                             │
│ Ai đang đau (Actor)? Ban Quản lý Khu đô thị (BQL) & Cư dân Vinhomes         │
│                                                                             │
│ Workflow thủ công hiện tại (4 bước):                                        │
│   1. Cư dân gửi phản ánh lên App (tiếng ồn, rác thải, phí dịch vụ...)       │
│   ──> 2. Nhân viên CSKH đọc và phân loại phòng ban xử lý                    │
│   ──> 3. Chuyển tiếp phiếu đến bộ phận kỹ thuật/vệ sinh                      │
│   ──> 4. Soạn thư phản hồi thủ công gửi cư dân                              │
│                                                                             │
│ Bước nào tốn thời gian/lỗi nhất? Bước 2 & 4 (⏱ 4-8 giờ phản hồi)            │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 2 (Intent Classify) & 4 (Draft)  │
│                                                                             │
│ Đo thành công bằng gì (Metric có số)?                                       │
│   Rút ngắn thời gian phản hồi ban đầu từ 12 giờ ──> dưới 15 phút.           │
│                                                                             │
│ Quick Architecture: [x] LLM Feature                                         │
│ Quick Verdict: [ ] WAIT (Cần chuẩn hóa taxonomy phân loại sự cố trước)      │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 🃏 QUICK PROBLEM CARD #3 — Vinmec: Tổng hợp hồ sơ xuất viện (Discharge Summary Draft)

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #3                                                       │
│                                                                             │
│ Bài toán: Bác sĩ Vinmec mất 20-30 phút/bệnh nhân để tóm tắt quá trình điều  │
│ trị, chỉ số xét nghiệm và đơn thuốc vào Hồ sơ xuất viện (Discharge Summary).│
│ Công ty thành viên: [x] Vinmec                                              │
│                                                                             │
│ Ai đang đau (Actor)? Bác sĩ điều trị, Y tá hành chính                       │
│                                                                             │
│ Workflow thủ công hiện tại (4 bước):                                        │
│   1. Đọc lại toàn bộ ghi chú lâm sàng & kết quả xét nghiệm EMR              │
│   ──> 2. Trích xuất thông tin bệnh lý trọng tâm                             │
│   ──> 3. Viết bản tóm tắt hồ sơ xuất viện                                   │
│   ──> 4. Dặn dò đơn thuốc và lịch tái khám                                  │
│                                                                             │
│ Bước nào tốn thời gian/lỗi nhất? Bước 1 & 3 (⏱ 25 phút/bệnh nhân)           │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 1-3 (Summarize EMR data)          │
│                                                                             │
│ Đo thành công bằng gì (Metric có số)?                                       │
│   Giảm thời gian soạn hồ sơ xuất viện từ 25 phút ──> dưới 5 phút.           │
│                                                                             │
│ Quick Architecture: [x] Agentic / LLM with RAG                              │
│ Quick Verdict: [ ] WAIT (Yêu cầu nghiêm ngặt về an toàn dữ liệu y tế HIPAA)  │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 🗳️ Kết luận & Bài toán được chọn cho Deep-Dive

Nhóm **safe-drive** thống nhất chọn **Quick Problem Card #1 — Xanh SM: Xử lý sự cố sạc pin thực địa** để tiến hành Deep-Dive và lập trình Prompt Prototype. 

* **Lý do chọn Card #1:** Bài toán tác động trực tiếp đến doanh thu real-time và trải nghiệm tài xế Xanh SM. Scope bài toán rõ ràng, ranh giới operational boundary (như pin < 5% kích hoạt cứu hộ di động) có thể kiểm soát hoàn toàn bằng kĩ thuật Prompt Engineering và Guardrails.
