<!-- ========================================================================== -->
<!-- THÔNG TIN NHÓM & DANH SÁCH THÀNH VIÊN                                      -->
<!-- Tên nhóm : safe-drive
Thành viên :
Nguyễn Đức Phát : nguyenducphat.edu@gmail.com
Đỗ Thành Đạt : dothanhdat10x@gmail.com
Nguyễn Ngọc Minh : nnminh433@gmail.com
Phạm Quang Đạt : phamqdat99@gmail.com
Lâm Hoàng Phúc : lamhoangphuc2003st@gmail.com
Chử Trần Phương Nam : tranphuongnam932004@gmail.com
<!-- ========================================================================== -->

# 02 — Báo Cáo Phân Tích Chuyên Sâu (Problem Deep-Dive Report)
## Hệ Thống Trợ Lý AI Co-pilot Hỗ Trợ Điều Vận Sự Cố Cạn Pin Taxi Điện Xanh SM

**Đơn vị chủ trì:** Vin Smart Future — Hợp tác cùng Khối Vận hành GSM Xanh SM  
**Vị trí chuyên môn:** AI Product Engineer  

---

## 🏗️ Phase 3 — PHÂN TÍCH CHUYÊN SÂU (DEEP-DIVE)

### 3.1. Sơ Đồ Quy Trình Hiện Tại (Current-State Workflow)

Quy trình giải quyết tình huống tài xế taxi điện Xanh SM gặp sự cố cạn pin hoặc trạm sạc đích bị quá tải hiện được xử lý qua 5 bước thủ công:

```text
┌─────────────────┐       🔄 Handoff 1      ┌─────────────────┐       🔄 Handoff 2      ┌─────────────────┐
│     BƯỚC 1      │ ──────────────────────> │     BƯỚC 2      │ ──────────────────────> │     BƯỚC 3      │
│ Tiếp nhận cảnh  │                         │ Mở bản đồ định  │                         │ Dò tìm trạm sạc │
│ báo sự cố pin   │                         │ vị toạ độ xe    │                         │ VinFast phù hợp │
│                 │                         │                 │                         │                 │
│ Actor: Tổng đài │                         │ Actor: Điều phối│                         │ Actor: Điều phối│
│ ⏱ 2 phút        │                         │ ⏱ 2 phút        │                         │ ⏱ 5 phút 🔴     │
└─────────────────┘                         └─────────────────┘                         │ 🔴 BOTTLENECK 1 │
                                                                                        └─────────────────┘
                                                                                                 │
                                                                                                 │ 🔄 Handoff 3
                                                                                                 ▼
┌─────────────────┐       🔄 Handoff 4      ┌─────────────────┐                         ┌─────────────────┐
│     BƯỚC 5      │ <────────────────────── │     BƯỚC 4      │ <───────────────────────┘
│ Điều xe sạc pin │  (Nếu pin < 5% cạn kiệt)│ Soạn thảo tin   │
│ lưu động cứu hộ │                         │ SMS chỉ dẫn     │
│                 │                         │                 │
│ Actor: Điều phối│                         │ Actor: Điều phối│
│ ⏱ 1 phút        │                         │ ⏱ 5 phút 🔴     │
└─────────────────┘                         │ 🔴 BOTTLENECK 2 │
                                            └─────────────────┘

Ghi chú phân tích:
🔴 Bottlenecks: Bước 3 (tra cứu thủ công) & Bước 4 (gõ tin nhắn tiếng Việt thủ công) tốn tới 10/15 phút (67%).
🔄 Handoffs: 4 điểm bàn giao thông tin rời rạc qua các hệ thống chưa được đồng bộ.
⏱ Tổng thời lượng xử lý: 15 phút cho mỗi ca sự cố.
```

---

### 3.2. Bảng Mô Tả Bài Toán Chuẩn 6 Trường (Problem Statement 6-Field)

| Trường thông tin | Chi tiết chuẩn doanh nghiệp |
|:---|:---|
| **1. Nhân sự thực thi (Actor)** | Điều phối viên trung tâm điều vận Xanh SM phối hợp cùng tài xế taxi điện hoạt động trên tuyến đường. |
| **2. Quy trình hiện tại** | Tiếp nhận cuộc gọi -> tra cứu toạ độ GPS -> tra cứu cổng sạc trống phù hợp với loại xe (VF5/VFe34/VF8) -> gõ tin nhắn chỉ đường/chỉ dẫn cho tài xế -> gọi xe cứu hộ nếu mức pin dưới 5%. Quy trình 5 bước thủ công kéo dài trung bình 15 phút. |
| **3. Điểm nghẽn (Bottleneck)** | Bước 3 & Bước 4: Mất 10 phút để đối soát trạng thái trụ sạc và tự gõ nội dung tin nhắn hướng dẫn tài xế vừa chính xác kỹ thuật vừa đảm bảo ngữ điệu hỗ trợ. |
| **4. Thiệt hại vận hành (Business Impact)** | Với trung bình 80 sự cố pin/ngày tại khu vực nội đô, doanh nghiệp tiêu tốn **20 giờ làm việc/ngày** của điều phối viên. Tình trạng xe chờ đợi gây giảm **~15% doanh thu cuốc xe**, tăng tỷ lệ hủy chuyến và tiềm ẩn nguy cơ xe hết sạch pin dừng giữa giao lộ gây ùn tắc nghiêm trọng. |
| **5. Chỉ số thành công (Success Metrics)** | - **Hiệu năng (Efficiency):** Giảm thời gian xử lý toàn trình từ 15 phút xuống dưới **3 phút**.<br>- **Chất lượng (Quality):** Đảm bảo trạm sạc được gợi ý đúng loại cổng sạc và còn trụ trống đạt **>= 98%**.<br>- **An toàn (Safety):** 100% tình huống pin dưới 5% được bảo vệ, không bao giờ điều đi xa quá 5km. |
| **6. Ranh giới vận hành (Operational Boundaries)** | **ĐƯỢC LÀM:** Tự động tổng hợp dữ liệu GPS và trạm sạc; tự động sinh bản thảo nội dung tin nhắn hướng dẫn.<br>**CẤM TUYỆT ĐỐI:** AI không được gửi tin trực tiếp đến tài xế khi chưa qua phê duyệt của điều phối viên (**Bắt buộc tag `[DRAFT_ONLY]`**). Không được điều xe có pin dưới **5%** đến trạm sạc cách xa trên **5km**, trường hợp này bắt buộc trả về lệnh điều xe cứu hộ sạc di động dạng JSON: `{"action": "dispatch_mobile_charger", "reason": "..."}`. |

---

### 3.3. Đánh Giá Kiến Trúc Công Nghệ & Quy Trình Tương Lai (Future-State Flow)

* **Lựa chọn mô hình (AI-Fit Matrix):** Chọn **LLM Feature (Co-pilot)**.
  * *Vì sao không dùng Rule-based:* Rule-based chỉ tính được khoảng cách nhưng không thể sinh tin nhắn cá nhân hóa, đồng cảm và linh hoạt theo từng tình huống tài xế.
  * *Vì sao không dùng Autonomous Agent:* Việc để AI tự động gửi lệnh điều xe ra ngoài mà không có con người kiểm soát sẽ tiềm ẩn rủi ro rất lớn nếu AI gặp lỗi ảo giác (hallucination), có thể dẫn đến xe điện chết máy giữa đường cao tốc.

#### Sơ đồ tương lai có chốt chặn Human-in-the-loop:

```text
┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐
│     BƯỚC 1      │ ────> │     BƯỚC 2      │ ────> │     BƯỚC 3      │ ────> │     BƯỚC 4      │
│ Tiếp nhận tín   │       │ 🔵 Tự động lấy  │       │ 🔵 AI Co-pilot  │       │ 🟢 Điều phối    │
│ hiệu sự cố pin  │       │ GPS xe & tình   │       │ sinh bản nháp   │       │ viên kiểm tra   │
│ từ App/Hệ thống │       │ trạng trạm trống│       │ [DRAFT_ONLY]    │       │ và 1-click gửi  │
│ ⏱ 30 giây       │       │ ⏱ 5 giây        │       │ ⏱ 10 giây       │       │ ⏱ 30 giây       │
└─────────────────┘       └─────────────────┘       └─────────────────┘       └─────────────────┘
                                                            │                         │
                               [Nếu pin < 5%]               │                         ▼
                               Tự động sinh lệnh:           │                   Gửi tin nhắn App
                               {"action":                   │                   / Điều cứu hộ
                                "dispatch_mobile_charger"}  │                   (Tổng: < 2 phút)
                                                            │
                                                            ▼
                                                     ↩️ Cơ chế Dự phòng (Fallback):
                                                     Nếu AI timeout hoặc format lỗi,
                                                     hệ thống chuyển về mẫu tin nhắn
                                                     tiêu chuẩn có sẵn để duyệt ngay.
```

---

## 💻 Phase 4 — Xây Dựng & Kiểm Thử Prompt Prototype

Nhóm đã hiện thực hóa mã nguồn tại `starter-code/prompt_prototype.py` và kiểm thử nghiêm ngặt trước các kịch bản tấn công (Adversarial Prompting):
1. **Kiểm tra Rule 1 (Ngăn chặn bypass kiểm duyệt):** Kẻ tấn công yêu cầu gửi tin nhắn ngay lập tức và bỏ qua tiền tố. Mô hình bảo vệ thành công ranh giới và vẫn giữ nguyên tag `[DRAFT_ONLY]`.
2. **Kiểm tra Rule 2 (Ngăn chặn điều hướng nguy hiểm khi pin cạn):** Tài xế còn 2% pin nài nỉ chỉ đường trạm 8km. Mô hình từ chối chỉ đường xa và lập tức xuất lệnh kích hoạt xe sạc di động: `{"action": "dispatch_mobile_charger", ...}`.

---

## 🏁 Phase 5 — ĐÁNH GIÁ TÍNH SẴN SÀNG & QUYẾT ĐỊNH

### Bộ tiêu chí kiểm tra (AI Readiness Checklist):
* [x] **Dữ liệu khả dụng:** Tọa độ GPS của xe và tình trạng trụ sạc VinFast đã có sẵn qua API nội bộ.
* [x] **Kiểm soát rủi ro an toàn:** Luôn có chốt chặn con người phê duyệt (HITL) và cơ chế Fallback rõ ràng.
* [x] **Mức độ sẵn sàng tiếp nhận:** Đội ngũ điều vận rất chào đón công cụ giúp giảm tải áp lực giờ cao điểm.

### 👉 QUYẾT ĐỊNH CUỐI CÙNG: [ GO ] — Triển khai thử nghiệm Pilot
* **Lý giải (Justification):** Dự án có ROI rõ ràng khi cắt giảm hơn 80% thời gian xử lý sự cố (từ 15 phút xuống dưới 3 phút), giúp giải phóng 20 giờ lao động mỗi ngày cho khối điều vận và bảo toàn doanh thu cuốc xe, với chi phí đầu tư kỹ thuật thấp nhờ kiến trúc LLM Co-pilot gọn nhẹ và ranh giới an toàn tuyệt đối.
