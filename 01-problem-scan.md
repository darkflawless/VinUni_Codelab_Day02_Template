<!-- Tên nhóm : safe-drive
Thành viên :
Nguyễn Đức Phát : nguyenducphat.edu@gmail.com
Đỗ Thành Đạt : dothanhdat10x@gmail.com
Nguyễn Ngọc Minh : nnminh433@gmail.com
Phạm Quang Đạt : phamqdat99@gmail.com
Lâm Hoàng Phúc : lamhoangphuc2003st@gmail.com
Chử Trần Phương Nam : tranphuongnam932004@gmail.com -->

# Phase 1 — SCAN: Bảng quét cơ hội AI tại Vingroup

> Tôi là AI Engineer tại **Vin Smart Future**, được giao hỗ trợ các mảng kinh doanh của Vingroup.
>
> **Bối cảnh thực tế:**
> - Xanh SM (GSM) chiếm **54.51% thị phần** taxi công nghệ Việt Nam Q1/2026, vượt Grab (40.92%) — Mordor Intelligence
> - Vận hành hơn **30.000 xe điện VinFast** tại 34 tỉnh thành, phục vụ hơn **100 triệu lượt khách** kể từ 14/4/2023
> - VinFast thông qua V-Green triển khai hạ tầng sạc công suất 7.4–120 kW trên toàn quốc
> - Thực tế vận hành ghi nhận: quãng đường thực tế của xe điện VinFast thấp hơn công bố do điều kiện đường, thời tiết, tốc độ và tình trạng pin *(VinFast engineer tại Hà Nội, vietnam.vn 2026)*

| # | Subsidiary | Lens | Mô tả ngắn bài toán |
|---|------------|------|---------------------|
| 1 | **Xanh SM** | Tốn thời gian | Dispatcher xử lý thủ công từng sự cố pin thực địa — tra vị trí xe, tìm trạm sạc V-Green còn trụ trống phù hợp loại cổng, soạn tin hướng dẫn — toàn bộ thủ công trong khi tài xế chờ ngoài đường. |
| 2 | **Xanh SM** | Lặp lại | Tóm tắt và phân loại lý do khách hủy chuyến từ ghi chú tài xế để tìm pattern lỗi hệ thống — lặp đi lặp lại mỗi ngày, hoàn toàn thủ công. |
| 3 | **VinFast** | Lặp lại | Đối chiếu hóa đơn sạc điện định kỳ từ hàng nghìn trụ sạc V-Green đối tác (7.4–120 kW) với hệ thống tài chính — tác vụ lặp, dễ sai sót khi làm thủ công. |
| 4 | **Vinhomes** | AI có thể tốt hơn | Phân loại và route khiếu nại cư dân gửi qua App Vinhomes Resident đến đúng ban quản lý — hiện nhân viên CSKH phân loại thủ công, phản hồi chậm và rập khuôn. |
| 5 | **Vinmec** | Pain từ người khác | Bác sĩ phải tự viết tóm tắt hồ sơ xuất viện cho từng bệnh nhân — tốn thời gian đáng kể, bác sĩ phàn nàn vì ảnh hưởng đến thời gian khám chữa bệnh. |

---

# Phase 2 — QUICK-ASSESS: 3 Quick Problem Cards

## Card #1 — Xanh SM: Xử lý sự cố sạc pin thực địa

```
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #1                                       │
│                                                             │
│ Bài toán: Tài xế Xanh SM hết pin giữa đường cần được       │
│ hướng dẫn đến trạm sạc V-Green hoặc điều xe cứu hộ pin.    │
│ Công ty thành viên: [x] Xanh SM (GSM)                       │
│                                                             │
│ Ai đang đau (Actor)?                                        │
│ Tài xế (chờ ngoài đường, mất cuốc tiếp theo)               │
│ Dispatcher (phải xử lý thủ công từng ca sự cố)             │
│                                                             │
│ Workflow thủ công hiện tại (5 bước):                        │
│   1. Tài xế gọi hotline báo hết pin                         │
│   → 2. Dispatcher tra vị trí xe trên bản đồ nội bộ         │
│   → 3. Tra thủ công trạm V-Green còn trụ trống             │
│      (cần khớp loại cổng: CCS2 cho VF8/VF9, Type 2 cho VF5)│
│   → 4. Soạn tin nhắn hướng dẫn đường đi, gửi qua App       │
│   → 5. Điều xe cứu hộ pin di động nếu pin < 5%             │
│                                                             │
│ Bước nào tốn nhất? Bước 3-4: tra trạm + soạn tin thủ công  │
│ AI hỗ trợ ở đâu? Bước 3-4: tự động tra + soạn tin nháp     │
│                                                             │
│ Đo thành công bằng gì?                                      │
│ Giảm thời gian dispatcher xử lý mỗi sự cố (baseline cần    │
│ đo thực tế tại trung tâm điều vận trước khi triển khai)     │
│                                                             │
│ Quick Architecture: [x] LLM Feature                        │
└─────────────────────────────────────────────────────────────┘
```

---

## Card #2 — Vinhomes: Phân loại khiếu nại cư dân tự động

```
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #2                                       │
│                                                             │
│ Bài toán: Khiếu nại cư dân gửi qua App Vinhomes Resident   │
│ bị phân loại thủ công, phản hồi chậm và rập khuôn.         │
│ Công ty thành viên: [x] Vinhomes                            │
│                                                             │
│ Ai đang đau (Actor)?                                        │
│ Cư dân (chờ phản hồi lâu) & CSKH (phân loại thủ công)      │
│                                                             │
│ Workflow thủ công hiện tại (4 bước):                        │
│   1. Cư dân gửi khiếu nại qua App                           │
│   → 2. CSKH đọc và phân loại thủ công từng ticket          │
│   → 3. Forward đến bộ phận phụ trách đúng tòa              │
│   → 4. Bộ phận xử lý và phản hồi cư dân                    │
│                                                             │
│ Bước nào tốn nhất? Bước 2: đọc + phân loại thủ công        │
│ AI hỗ trợ ở đâu? Tự động phân loại và route ticket         │
│                                                             │
│ Đo thành công bằng gì?                                      │
│ Tỉ lệ phân loại đúng & thời gian route đến bộ phận xử lý   │
│ (baseline cần đo từ dữ liệu ticket thực tế của Vinhomes)    │
│                                                             │
│ Quick Architecture: [x] LLM Feature                        │
└─────────────────────────────────────────────────────────────┘
```

---

## Card #3 — Vinmec: Soạn thảo tóm tắt hồ sơ xuất viện

```
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #3                                       │
│                                                             │
│ Bài toán: Bác sĩ Vinmec phải tự viết tóm tắt hồ sơ xuất    │
│ viện cho từng bệnh nhân — tốn thời gian, bác sĩ phàn nàn   │
│ vì cắt vào thời gian khám chữa bệnh.                        │
│ Công ty thành viên: [x] Vinmec                              │
│                                                             │
│ Ai đang đau (Actor)?                                        │
│ Bác sĩ (quá tải giấy tờ) & Bệnh nhân (chờ lâu ra viện)    │
│                                                             │
│ Workflow thủ công hiện tại (4 bước):                        │
│   1. Bác sĩ đọc toàn bộ bệnh án điện tử + xét nghiệm       │
│   → 2. Tự trích xuất thông tin lâm sàng quan trọng          │
│   → 3. Viết tóm tắt ngôn ngữ dễ hiểu cho bệnh nhân        │
│   → 4. Ký duyệt và in tóm tắt xuất viện                    │
│                                                             │
│ Bước nào tốn nhất? Bước 2-3: đọc bệnh án + viết tóm tắt    │
│ AI hỗ trợ ở đâu? Trích xuất thông tin + soạn nháp tóm tắt  │
│                                                             │
│ Đo thành công bằng gì?                                      │
│ Thời gian bác sĩ tiêu tốn cho bước soạn thảo/bệnh nhân     │
│ (baseline cần đo từ dữ liệu thực tế tại Vinmec)            │
│                                                             │
│ Quick Architecture: [x] LLM Feature                        │
└─────────────────────────────────────────────────────────────┘
```

---

## Quyết định chọn bài toán để Deep-Dive

Nhóm chọn **Card #1 — Xanh SM: Xử lý sự cố pin thực địa**.

**Lý do:**
- Bài toán xảy ra thường xuyên và có cấu trúc rõ ràng: mỗi sự cố đều đi qua đúng 5 bước cố định, dễ đo và dễ tự động hóa từng bước.
- Tài xế VinFast thực tế gặp tình trạng pin thực tế thấp hơn công bố do điều kiện đường, thời tiết, tải trọng *(VinFast engineer, vietnam.vn 2026)* — bài toán này không phải giả định mà đang xảy ra trong vận hành.
- Xanh SM đang vận hành 30.000+ xe trên 34 tỉnh thành với tốc độ mở rộng nhanh → áp lực lên đội dispatcher tỉ lệ thuận với quy mô đội xe.
- Ranh giới an toàn rõ ràng và kiểm soát được: HITL bắt buộc, fallback về thủ công.

**Lý do loại các card khác:**
- **Card #2 (Vinhomes):** Phân loại sai khiếu nại liên quan phí quản lý, tranh chấp tài sản có thể gây rủi ro pháp lý cho Vinhomes — cần rule-based router trước, AI bổ sung sau.
- **Card #3 (Vinmec):** Y tế là lĩnh vực nhạy cảm nhất. Tóm tắt xuất viện sai có thể ảnh hưởng đến quyết định điều trị tiếp theo của bệnh nhân. Cần HITL chặt chẽ hơn và quy trình kiểm duyệt y tế riêng.
