# 📊 Deep-Dive Report: Hệ Thống Telematics & AI Dispatcher Co-pilot Điều Phối Cứu Hộ Pin Khẩn Cấp (Xanh SM & VinFast Energy)

* **Nhóm thực hiện:** Vin Smart Future Engineering Team
* **Tác giả:** Chu Trần Phương Nam (Branch: `chutranphuongnam`)
* **Bài toán lựa chọn:** Giám sát Telematics thời gian thực, phát hiện xe điện cạn pin khẩn cấp (< 5%) và tự động lập lệnh điều phối xe sạc pin lưu động (Mobile Fast-Charger).

---

## 1. Workflow Mapping (Quy trình vận hành hiện tại)
*(Chi tiết trực quan hóa quy trình xem tại sơ đồ `04-workflow-diagram.png` đính kèm).*

Quy trình xử lý sự cố xe điện cạn pin nguy cấp tại Trung tâm Điều hành Xanh SM & VinFast Energy:

```text
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│ Bước 1       │     │ Bước 2       │     │ Bước 3       │     │ Bước 4       │
│ Cảnh báo SoC │     │ Tra cứu GPS  │     │ Đánh giá trạm│     │ Tìm xe sạc & │
│ từ xe (< 5%) │ ──→ │ & Dòng xe    │ ──→ │ sạc khả dụng │ ──→ │ Soạn lệnh cứu│
│              │     │              │     │ (🔴 Bottleneck)│    │ hộ (🔴 Bottl.)│
│ Ai: Dispatch │     │ Ai: Dispatch │     │ Ai: Dispatch │     │ Ai: Dispatch │
│ ⏱ 2 phút     │     │ ⏱ 2 phút     │     │ ⏱ 5 phút     │     │ ⏱ 7 phút     │
│ In: Telematics│    │ In: Mã xe    │     │ In: Toạ độ   │     │ In: Vị trí cứu│
│ Out: Ticket  │     │ Out: Vị trí  │     │ Out: Từ chối │     │ Out: Draft SMS│
└──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
                                                                      │
                                                                      ▼
                                                               ┌──────────────┐
                                                               │ Bước 5       │
                                                               │ Gọi xác nhận │
                                                               │ & Phái xe sạc│
                                                               │ Ai: Dispatch │
                                                               │ ⏱ 2 phút     │
                                                               └──────────────┘
🔴 Bottlenecks: Bước 3 & Bước 4 (Chiếm 12/18 phút)
⏱ Tổng thời gian xử lý thủ công: 18 phút/lượt.
```

* **Bước 1: Cảnh báo Telematics từ xe:** Cảm biến BMS trên xe gửi tín hiệu pin SoC < 5% về máy chủ trung tâm hoặc tài xế kích hoạt nút khẩn cấp SOS trên màn hình xe (2 phút).
* **Bước 2: Tra cứu thông số xe:** Điều phối viên mở hệ thống Telematics để kiểm tra vị trí GPS, dòng xe (VF5 / VFe34 / VF8) và trạng thái giao thông xung quanh (2 phút).
* **Bước 3 (🔴 Bottleneck 1): Tra cứu trạm sạc & đánh giá an toàn:** Điều phối viên kiểm tra các trạm sạc VinFast gần nhất. Thường các trạm sạc cách xa > 5km trong giờ cao điểm không đảm bảo an toàn cho xe di chuyển tiếp, dễ gây phán đoán sai lầm (5 phút).
* **Bước 4 (🔴 Bottleneck 2): Tìm xe cứu hộ & soạn văn bản chỉ dẫn:** Điều phối viên quét mạng lưới xe sạc pin di động (Mobile Charger Van) đang rảnh cuốc, tính toán khoảng cách và soạn tin nhắn SMS/App hướng dẫn tài xế tấp xe vào lề an toàn chờ cứu hộ (7 phút).
* **Bước 5: Xác nhận và phát lệnh:** Điều phối viên gọi bộ đàm hoặc điện thoại cho tài xế xe cứu hộ xác nhận nhiệm vụ và cập nhật trạng thái lên phần mềm quản lý (2 phút).

---

## 2. Problem Statement (6-field) — Vin Smart Future Standard

| Trường thông tin (Field) | Nội dung chi tiết |
|---|---|
| **1. Actor / Operator** | Điều phối viên khẩn cấp (Emergency Dispatcher) tại Trung tâm Điều hành Vận tải Xanh SM phối hợp cùng Khối Dịch vụ Cứu hộ Pin VinFast Energy. |
| **2. Current Workflow** | Quy trình thủ công 5 bước: Nhận tín hiệu Telematics/SOS -> Tra cứu vị trí và tình trạng pin -> Đánh giá trạm sạc khả dụng -> Tìm xe cứu hộ pin lưu động phù hợp -> Soạn tin nhắn hướng dẫn an toàn và phái xe cứu hộ. |
| **3. Bottleneck** | **Bước 3 & Bước 4 (mất 12 phút):** Đánh giá rủi ro cạn pin vật lý và tìm xe sạc lưu động khả dụng, kết hợp soạn văn bản hướng dẫn dừng đỗ an toàn, dẫn đến chậm trễ phát lệnh khẩn cấp. |
| **4. Business Impact** | Mỗi ngày có trung bình 60-80 vụ cảnh báo pin nguy cấp (< 5%) tại Hà Nội và TP.HCM. Chậm trễ phát lệnh cứu hộ khiến xe cạn kiệt pin giữa ngã tư hoặc trên cầu, gây ùn tắc giao thông, tốn chi phí xe kéo chuyên dụng (~1.500.000đ/lần), làm suy giảm tuổi thọ pin do xả sâu về 0% và ảnh hưởng nghiêm trọng đến hình ảnh thương hiệu taxi xanh thông minh. |
| **5. Success Metric** | 1. **Thời gian (Efficiency):** Giảm thời gian từ lúc nhận cảnh báo pin < 5% đến khi phát lệnh xe sạc di động từ 18 phút xuống **dưới 2.5 phút**.<br>2. **Độ an toàn (Zero Violation):** **Triệt tiêu 100%** trường hợp AI gợi ý di chuyển đến trạm sạc xa quá 5km khi pin < 5%.<br>3. **Chất lượng phục vụ:** 100% tài xế nhận được hướng dẫn đỗ xe an toàn và thời gian dự kiến xe sạc tiếp cận trong vòng 3 phút. |
| **6. Operational Boundary** | **AI được phép:** Tự động gom dữ liệu GPS, SoC xe, lọc trạm sạc trống và vị trí xe sạc lưu động; tự động soạn bản thảo lệnh điều phối và tin nhắn tài xế.<br>**Ranh giới cấm tuyệt đối:**<br>1. Tuyệt đối KHÔNG tự động phái xe cứu hộ hoặc gửi tin ra ngoài mà không có Dispatcher click duyệt (bắt buộc gắn thẻ `[DRAFT_ONLY]` ở đầu mọi phản hồi).<br>2. Tuyệt đối KHÔNG đề xuất di chuyển đến trạm sạc xa > 5km khi pin < 5%; bắt buộc phản hồi hành động điều xe sạc di động: `{"action": "dispatch_mobile_charger", "reason": "..."}`. |

---

## 3. AI Fit & Future-State Flow

### 3.1. Phân tích AI-Fit Matrix:
* **Rule-based Engine thuần túy:** Phù hợp để kiểm tra điều kiện `Pin < 5%`, nhưng không đủ linh hoạt để tổng hợp vị trí địa lý, thông tin tài xế và soạn thảo văn bản hướng dẫn mang tính trấn an, rõ ràng bằng tiếng Việt theo ngữ cảnh thời gian thực.
* **Autonomous Multi-Agent:** Rủi ro rất cao nếu Agent tự động gửi lệnh phái xe cứu hộ hoặc điều hướng xe mà không có sự kiểm duyệt của con người, có thể gây lãng phí nguồn lực xe cứu hộ nếu phát sinh báo động giả từ cảm biến.
* **Lựa chọn tối ưu:** **LLM Feature (Dispatcher Co-pilot)** kết hợp cơ chế kiểm soát ranh giới phủ định (Negative Constraints) và **Human-in-the-loop (HITL)**.

### 3.2. Future-State Flow (Quy trình tương lai tích hợp AI):

```text
[🚗 Telematics Alert: Pin SoC < 5% & Tọa độ GPS]
                       │
                       ▼
    [⚡ Rule Gateway: Kiểm tra ngưỡng an toàn]
      ├─ Pin < 5%: Cấm trạm sạc xa > 5km
      └─ Bật cờ yêu cầu Mobile Fast-Charger
                       │
                       ▼
[🔵 AI Feature (Gemini 2.5): Tổng hợp & Soạn Draft có tag [DRAFT_ONLY]]
      │
      ├─► Trường hợp Pin < 5%:
      │   Output JSON: {"action": "dispatch_mobile_charger", "reason": "..."}
      │
      └─► Kèm bản thảo tin nhắn hướng dẫn tài xế dừng đỗ an toàn
                       │
                       ▼
[🟢 HITL: Điều phối viên 1-Click xem trước, phê duyệt & phái xe cứu hộ]
                       │
                       ▼ (Nếu LLM timeout / lỗi phản hồi)
[↩️ Fallback: Hệ thống chuyển sang bảng điều khiển thủ công truyền thống]
```

---

## 4. Phase 5 — EVALUATE (Đánh giá độ sẵn sàng & Quyết định)

### AI Readiness Checklist:
1. [x] **Dữ liệu sẵn sàng (Data Readiness):** Nền tảng IoT Telematics của VinFast đã kết nối thời gian thực với toàn bộ đội xe Xanh SM (vị trí GPS, SoC pin, trạng thái khóa xe). Đội xe sạc pin di động (Mobile Charger) đã được trang bị thiết bị định vị riêng.
2. [x] **Rủi ro kiểm soát được (Risk Mitigation):** 
   - Ranh giới an toàn được bảo vệ đa lớp: Rào chắn thẻ `[DRAFT_ONLY]` đảm bảo con người luôn là mắt xích phê duyệt cuối cùng.
   - Cơ chế chặn cứng tự động chuyển sang JSON `dispatch_mobile_charger` khi pin < 5% ngăn chặn triệt để rủi ro xe cạn pin giữa đường.
   - Có cơ chế Fallback quay về quy trình thủ công ngay lập tức khi hệ thống AI gặp sự cố.
3. [x] **Stakeholders đồng thuận (Stakeholder Alignment):** Cả Ban Giám đốc GSM và Đội ngũ Điều phối viên đều ủng hộ mạnh mẽ do hệ thống giúp giải tỏa trực tiếp áp lực điều vận giờ cao điểm và giảm thiểu tổn thất vận hành.

### Quyết định cuối cùng của Ban Giám Đốc Vin Smart Future:
* [x] **GO (Bắt đầu xây dựng Prototype)**
* [ ] **NOT YET (Cần tích lũy thêm dữ liệu / xác lập baseline)**
* [ ] **NO-GO (Không khả thi / Rule-based tốt hơn)**

### Justification (Lý giải quyết định dựa trên bằng chứng kỹ thuật và chi phí):
* **Bằng chứng kỹ thuật:** Bản mẫu prompt prototype đã được lập trình và thử nghiệm thành công với Gemini 2.5 Flash SDK (`starter-code/prompt_prototype.py`). Mô hình vượt qua 100% các bài kiểm thử biên (Adversarial Testing) khi người dùng cố tình tạo áp lực ép gửi tin nhắn trực tiếp hoặc đòi đi trạm sạc xa.
* **Hiệu quả kinh tế & vận hành:** Giảm 85% thời gian xử lý sự cố (từ 18 phút xuống dưới 2.5 phút), cứu hàng chục nghìn lượt xe tránh khỏi nguy cơ chết máy mỗi năm, tiết kiệm hàng tỷ đồng chi phí xe kéo và bảo toàn độ bền của khối pin xe điện VinFast. Chi phí hạ tầng LLM API ở mức không đáng kể so với lợi ích thu được.