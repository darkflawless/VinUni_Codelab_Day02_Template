# Lab 02 — Problem Scan & Quick Assess (Vin Smart Future)

* **Nhóm thực hiện:** Vin Smart Future Engineering Team
* **Thành viên:** Chu Trần Phương Nam (Branch: `chutranphuongnam`)
* **Chủ đề:** Tối ưu hóa vận hành hệ sinh thái Vingroup bằng Trí tuệ Nhân tạo

---

## Phase 1 — SCAN (Quét cơ hội qua 4 Lăng kính)

| # | Subsidiary | Lens | Mô tả ngắn bài toán & Điểm nghẽn vận hành |
|---|---|---|---|
| 1 | **VinFast** | AI-upgrade | Khách hàng mô tả lỗi xe bằng tiếng Việt phi kỹ thuật (ví dụ: *"xe đạp ga kêu rít rít ở bánh trước bên phụ"*), kỹ thuật viên mất 30-45 phút lái thử để khoanh vùng lỗi. Cần AI phân tích mô tả và âm thanh để chẩn đoán sơ bộ nhóm linh kiện nghi vấn. |
| 2 | **Vinhomes** | Tốn thời gian | Ban quản lý tòa nhà mất 3-5 ngày làm việc để rà soát thủ công bản vẽ kỹ thuật, danh mục vật liệu thi công nội thất căn hộ cư dân nộp lên so với quy chuẩn an toàn phòng cháy chữa cháy (PCCC) và kết cấu tòa nhà. |
| 3 | **Vinpearl** | Pain từ người khác | Khách sạn mất trung bình 24-48 giờ để phát hiện các đánh giá tiêu cực nghiêm trọng (1-star về vệ sinh, thái độ phục vụ) trên TripAdvisor/Booking.com/Google Maps, dẫn đến việc xử lý khiếu nại chậm trễ và ảnh hưởng uy tín thương hiệu. |
| 4 | **Vinmec** | Lặp lại | Bộ phận Giám định BHYT phải đối chiếu thủ công từng dòng mã thuốc, vật tư tiêu hao và phác đồ điều trị của hàng trăm bệnh nhân mỗi ngày với danh mục chi trả của Bảo hiểm Xã hội nhằm tránh nguy cơ bị xuất toán. |
| 5 | **VinFast Energy** | Tốn thời gian | Kỹ sư vận hành trạm sạc phải lọc thủ công log dữ liệu nhiệt độ súng sạc và độ sụt áp của hàng nghìn trụ sạc siêu nhanh (Supercharger) trên toàn quốc để tìm ra các trụ sạc có nguy cơ hỏng hóc trong 48 giờ tới. |
| 6 | **Xanh SM & VinFast** | Lặp lại | Hệ thống Telematics ghi nhận hàng trăm cảnh báo pin sụt giảm nguy cấp (SoC < 5%) của xe taxi điện mỗi ngày; điều phối viên phải tính toán thủ công khoảng cách, tìm xe sạc lưu động và gọi điện thoại hướng dẫn tài xế (mất 15-20 phút/vụ). |

---

## Phase 2 — QUICK-ASSESS (3 Quick Problem Cards)

### Thẻ #1: VinFast — Trợ lý chẩn đoán sơ bộ lỗi xe qua mô tả khách hàng

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #1                                       │
│                                                             │
│ Bài toán: Hỗ trợ Cố vấn Dịch vụ chẩn đoán ban đầu lỗi kỹ    │
│ thuật xe điện từ mô tả hiện tượng của khách hàng.           │
│ Công ty thành viên: [x] VinFast (After-sales Service)       │
│                                                             │
│ Ai đang đau? Cố vấn dịch vụ (Service Advisor), Khách hàng   │
│                                                             │
│ Workflow thủ công hiện tại (5 bước):                        │
│   1. Tiếp nhận xe & nghe khách tả hiện tượng tiếng ồn/lỗi    │
│   ──> 2. Ghi chép mô tả tay vào phiếu tiếp nhận             │
│   ──> 3. Cố vấn cùng Kỹ thuật viên lái thử xe (30-45 min)   │
│   ──> 4. Cắm máy quét OBD tra mã lỗi DTC                    │
│   ──> 5. Lập báo giá & phương án sửa chữa gửi khách         │
│                                                             │
│ Bước nào tốn thời gian/lỗi nhất? Bước 1 & 3 (⏱ 40 phút/xe)   │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 1 & 2            │
│ (Trích xuất triệu chứng, so khớp cẩm nang sửa chữa VinFast, │
│  gợi ý top 3 nguyên nhân và linh kiện nghi vấn kèm tỷ lệ %) │
│                                                             │
│ Đo thành công bằng gì (Metric có số)?                        │
│   "Giảm thời gian chẩn đoán sơ bộ từ 40 phút xuống dưới     │
│    8 phút; độ chính xác khoanh vùng đúng cụm linh kiện >= 90%"│
│                                                             │
│ Quick Architecture: [x] LLM Feature (Co-pilot RAG sổ tay KT)│
└─────────────────────────────────────────────────────────────┘
```

---

### Thẻ #2: Vinhomes — Thẩm định tự động hồ sơ đăng ký thi công nội thất

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #2                                       │
│                                                             │
│ Bài toán: Tự động rà soát hồ sơ bản vẽ và danh mục vật tư    │
│ đăng ký hoàn thiện nội thất căn hộ cư dân Vinhomes.         │
│ Công ty thành viên: [x] Vinhomes (Ban Quản Lý Đô Thị)       │
│                                                             │
│ Ai đang đau? Kỹ sư Ban Quản Lý tòa nhà, Chủ hộ & Nhà thầu   │
│                                                             │
│ Workflow thủ công hiện tại (4 bước):                        │
│   1. Nhận tệp PDF hồ sơ bản vẽ và cam kết thi công          │
│   ──> 2. Kỹ sư mở từng trang kiểm tra chỉ tiêu chịu tải     │
│   ──> 3. Đối chiếu danh mục vật liệu với tiêu chuẩn PCCC    │
│   ──> 4. Soạn công văn phản hồi phê duyệt hoặc yêu cầu sửa  │
│                                                             │
│ Bước nào tốn thời gian/lỗi nhất? Bước 2 & 3 (⏱ 3-5 ngày/hồ sơ)│
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 2 & 3            │
│ (Trích xuất thông số kỹ thuật bản vẽ, kiểm tra checklist    │
│  quy định Vinhomes, highlight các vi phạm PCCC và draft văn bản)│
│                                                             │
│ Đo thành công bằng gì (Metric có số)?                        │
│   "Rút ngắn thời gian thẩm định từ 72 giờ xuống dưới 4 giờ; │
│    100% hồ sơ vi phạm quy chuẩn an toàn chịu lực được phát hiện"│
│                                                             │
│ Quick Architecture: [x] Document AI + Rule Engine + LLM HITL│
└─────────────────────────────────────────────────────────────┘
```

---

### Thẻ #3: Xanh SM & VinFast — Điều phối thông minh cứu hộ pin di động (Mobile Charger)

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #3                                       │
│                                                             │
│ Bài toán: Giám sát Telematics thời gian thực, phát hiện xe   │
│ cạn pin khẩn cấp (< 5%) và tự động lập lệnh điều phối xe    │
│ sạc pin lưu động (Mobile Fast-Charger) ứng cứu tài xế taxi. │
│ Công ty thành viên: [x] Xanh SM (GSM) & VinFast Energy      │
│                                                             │
│ Ai đang đau? Điều phối viên (Dispatcher), Tài xế Xanh SM    │
│                                                             │
│ Workflow thủ công hiện tại (5 bước):                        │
│   1. Nhận cảnh báo pin cạn hoặc cuộc gọi cầu cứu từ tài xế  │
│   ──> 2. Tra cứu tọa độ GPS xe và đánh giá mật độ giao thông│
│   ──> 3. Lọc trạm sạc gần nhất (thường quá xa tầm di chuyển) │
│   ──> 4. Tìm xe sạc di động đang rảnh và tính cự ly tiếp cận│
│   ──> 5. Soạn tin nhắn hướng dẫn và điều xe cứu hộ          │
│                                                             │
│ Bước nào tốn thời gian/lỗi nhất? Bước 3, 4, 5 (⏱ 15-20 phút) │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 3, 4, 5          │
│ (Xác thực ngưỡng an toàn < 5%, chặn gợi ý trạm sạc xa > 5km,│
│  tự động draft lệnh cứu hộ pin và tạo hướng dẫn an toàn)    │
│                                                             │
│ Đo thành công bằng gì (Metric có số)?                        │
│   "Giảm thời gian phát lệnh cứu hộ từ 18 phút xuống dưới 2 phút;│
│    triệt tiêu 100% sự cố xe chết máy do điều hướng trạm sạc sai"│
│                                                             │
│ Quick Architecture: [x] Telematics Event Engine + LLM Co-pilot│
└─────────────────────────────────────────────────────────────┘
```

---

## 🗳️ Quyết định chọn bài toán cho Deep-Dive của nhóm:

Nhóm thống nhất chọn bài toán **Quick Problem Card #3: Hệ thống Telematics & AI Dispatcher Co-pilot Điều phối Cứu hộ Sạc Pin Khẩn cấp (Xanh SM & VinFast Energy)** để tiến hành nghiên cứu sâu trong `02-deep-dive-report.md`.

### Lý do lựa chọn:
1. **Tính cấp thiết và ảnh hưởng vận hành thời gian thực (Real-time Criticality):** Xe taxi điện hoạt động liên tục trên đường. Khi pin xuống dưới 5%, nếu xử lý chậm 5-10 phút xe sẽ chết máy giữa đường, gây ùn tắc giao thông, nguy cơ tai nạn và làm gián đoạn doanh thu trực tiếp của GSM.
2. **Có ranh giới an toàn vật lý rõ ràng (Strict Operational Boundary):** Dễ dàng thiết lập các quy tắc bất biến cho AI (ví dụ: cấm điều hướng trạm xa khi pin < 5%, bắt buộc có thẻ duyệt `[DRAFT_ONLY]`, tự động kích hoạt `dispatch_mobile_charger`).
3. **Tính khả thi kỹ thuật cao:** Dữ liệu cảm biến xe (SoC, GPS, mã dòng xe VF5/VFe34/VF8) đã có sẵn qua hệ thống VinFast Cloud Telematics. Giải pháp kết hợp giữa Rule ranh giới an toàn và LLM Feature Co-pilot mang lại hiệu quả vượt trội.