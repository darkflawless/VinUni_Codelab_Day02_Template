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

# 🏗️ Deep-Dive Report: Xanh SM Intelligent Dispatcher (Vin Smart Future)

---

## 🏛️ Executive Summary & Thông tin Nhóm
* **Tên nhóm:** safe-drive
* **Thành viên:**
  1. Nguyễn Đức Phát (nguyenducphat.edu@gmail.com)
  2. Đỗ Thành Đạt (dothanhdat10x@gmail.com)
  3. Nguyễn Ngọc Minh (nnminh433@gmail.com)
  4. Phạm Quang Đạt (phamqdat99@gmail.com)
  5. Lâm Hoàng Phúc (lamhoangphuc2003st@gmail.com)
  6. Chử Trần Phương Nam (tranphuongnam932004@gmail.com)
* **Đơn vị phát triển:** Vin Smart Future (Vingroup Technology Division)
* **Đối tác vận hành:** Khối Vận Hành Xanh SM (GSM)
* **Tên giải pháp:** AI Dispatcher Co-Pilot — Trợ lý AI Điều phối Sự cố Pin Thực địa cho Taxi Điện Xanh SM
* **Quyết định đề xuất:** **GO** (Tiến hành xây dựng Prototype)

---

# 📋 1. Problem Statement (6-Field Standard)

| Field | Nội dung chi tiết |
|---|---|
| **1. Actor / Operator** | **Điều phối viên (Dispatcher)** tại Trung tâm Điều vận Xanh SM Hà Nội & TP.HCM. |
| **2. Current Workflow** | Khi tài xế báo sự cố cạn pin giữa ca trực: Dispatcher nhận cuộc gọi -> tra cứu vị trí GPS xe -> mở Dashboard trạm sạc VinFast tìm trụ trống -> viết tin nhắn hướng dẫn đường đi -> gọi xe cứu hộ nếu pin < 5%. Toàn bộ 5 bước xử lý thủ công. |
| **3. Bottleneck** | **Bước 3 & 4 (mất 10-12 phút/lượt):** Dispatcher phải tra cứu thủ công tình trạng trụ sạc trống theo đúng cổng sạc xe (VF5/VFe34/VF8) và gõ tin nhắn hướng dẫn Tiếng Việt cho tài xế. |
| **4. Business Impact** | Trung bình ~85 sự cố pin/ngày tại Hà Nội. Ngốn ~21 giờ làm việc/ngày của team điều vận. Gây chậm trễ phục vụ, tài xế stress và tổn thất doanh thu ước tính ~15% do xe dừng hoạt động chờ chỉ dẫn. |
| **5. Success Metric** | 1. **Thời gian xử lý:** Giảm tổng thời gian xử lý sự cố pin từ 15 phút xuống dưới 3 phút (giảm 80%).<br>2. **Độ chính xác:** Tỉ lệ thông tin trạm sạc & loại trụ tương thích đạt ≥ 98%. |
| **6. Operational Boundary** | AI chỉ đóng vai trò **Co-pilot (Soạn thảo bản nháp)**. **CẤM TUYỆT ĐỐI:** AI không được tự gửi tin nhắn trực tiếp cho tài xế mà không gắn thẻ `[DRAFT_ONLY]` cho Dispatcher duyệt; khi pin < 5%, AI không được điều xe đến trạm sạc xa > 5km mà phải lập tức phát lệnh điều động **Xe sạc pin di động (dispatch_mobile_charger)**. |

---

# 🔄 2. Workflow Transformation

## 2.1. Quy trình hiện tại (Current-State Workflow)

```text
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│ Bước 1       │     │ Bước 2       │     │ Bước 3       │     │ Bước 4       │
│ Nhận cuộc    │ ──> │ Tra cứu định │ ──> │ Tra cứu trạm │ ──> │ Soạn văn bản │
│ gọi sự cố    │     │ vị GPS xe    │     │ sạc VinFast  │     │ hướng dẫn    │
│ (Dispatcher) │     │ (Dispatcher) │     │ còn trụ trống│     │ gửi tài xế   │
│ ⏱ 2 phút     │     │ ⏱ 2 phút     │     │ ⏱ 5 phút 🔴  │     │ ⏱ 5 phút 🔴  │
└──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
                                                                      │
                                                                      ▼
                                                               ┌──────────────┐
                                                               │ Bước 5       │
                                                               │ Gọi xe cứu   │
                                                               │ hộ (nếu cần) │
                                                               │ ⏱ 1 phút     │
                                                               └──────────────┘
🔴 = Bottleneck (Tốn 10 phút thủ công)
⏱ Tổng thời gian: 15 phút/lượt xử lý
```

## 2.2. Quy trình tương lai có AI (Future-State AI-Augmented Workflow)

```text
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│ Bước 1       │     │ Bước 2       │     │ Bước 3       │     │ Bước 4       │
│ Nhận cuộc    │ ──> │ 🔵 Auto-fetch│ ──> │ 🔵 AI Draft  │ ──> │ 🟢 Human-in- │
│ gọi sự cố    │     │ GPS xe &     │     │ SMS & Lộ     │     │ the-Loop     │
│ (Dispatcher) │     │ Trạm sạc API │     │ trình sạc    │     │ Click Duyệt  │
│ ⏱ 0.5 phút   │     │ ⏱ < 2 giây   │     │ ⏱ < 3 giây   │     │ ⏱ 0.5 phút   │
└──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
                                                                      │
                                                                      ▼
                                                               ↩️ Fallback:
                                                               Nếu AI bị lỗi API
                                                               hoặc không tự tin,
                                                               Dispatcher tự gõ
                                                               tin nhắn thủ công.
```

---

# 🛡️ 3. AI Fit & Boundary Definition

## 3.1. Phân tích AI Fit Matrix
* **Giải pháp lựa chọn:** **LLM Feature + Rule Guardrails**
* **Lý giải:** Bài toán cần khả năng tổng hợp ngôn ngữ tự nhiên Tiếng Việt thân thiện, linh hoạt giải thích lộ trình sạc cho tài xế Xanh SM, nhưng KHÔNG CẦN đến Agent tự trị hoàn toàn (Autonomous Agent) vì quy trình nghiệp vụ yêu cầu kiểm soát rủi ro an toàn tuyệt đối.

## 3.2. Ranh giới vận hành (Operational Guardrails)
1. **Quy tắc thẻ nháp [DRAFT_ONLY]:**
   * Output của AI bắt buộc phải bắt đầu bằng thẻ `[DRAFT_ONLY]`. Hệ thống Backend sẽ chặn không cho SMS gateway gửi tin nếu thiếu thẻ này.
2. **Nội quy pin cạn kiệt (< 5%):**
   * Nếu dữ liệu pin xe cạn dưới 5%, AI không bao giờ gợi ý trạm sạc xa > 5km. AI bắt buộc kích hoạt hành động cứu hộ pin di động:
     `{"action": "dispatch_mobile_charger", "reason": "Pin cạn < 5%"}`.

## 3.3. Cơ chế duyệt (Human-in-the-Loop) & Dự phòng (Fallback)
* **Human-in-the-Loop (HITL):** Dispatcher xem bản nháp SMS trên màn hình điều vận, kiểm tra thông tin và nhấn nút **"Phê duyệt & Gửi"**.
* **Graceful Fallback:** Nếu dịch vụ LLM gặp sự cố timeout (> 5 giây) hoặc trả về kết quả không có thẻ `[DRAFT_ONLY]`, hệ thống tự động fallback về mẫu tin nhắn định sẵn (Static template) dựa trên trạm sạc gần nhất do Rule-based engine tính toán.

---

# 🏁 4. Phase 5 — Decision Framework & Evaluation

### Checklist Đánh giá Mức độ Sẵn sàng AI:
* [x] **Dữ liệu mẫu/Logs sạch:** Đã có log dữ liệu GPS xe, danh sách 500+ trạm sạc VinFast và lịch sử cuộc gọi điều vận.
* [x] **Kiểm soát rủi ro:** Có cơ chế HITL (Dispatcher phê duyệt) và Guardrail pin < 5% chặn hoàn toàn rủi ro xe chết máy giữa đường.
* [x] **Stakeholder Readiness:** Đội ngũ Dispatcher Xanh SM sẵn sàng tiếp nhận trợ lý Co-pilot để giảm áp lực ca trực.

### Quyết định cuối cùng của Nhóm safe-drive: **GO** (Bắt đầu xây dựng Prototype)

### Justification (Lý giải quyết định):
Dự án mang lại ROI cao (tiết kiệm 80% thời gian xử lý sự cố, giải phóng ~20 giờ làm việc/ngày cho team điều vận), kiến trúc công nghệ LLM Feature với Rule Guardrails đơn giản, chi phí vận hành thấp và độ an toàn vận hành được chứng thực qua chạy kiểm thử Prompt Prototype.
