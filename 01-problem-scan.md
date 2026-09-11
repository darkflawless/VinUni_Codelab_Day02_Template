<!-- ========================================================================== -->
<!-- THÔNG TIN NHÓM & DANH SÁCH THÀNH VIÊN                                      -->
<!-- Tên nhóm: [ĐIỀN TÊN NHÓM CỦA BẠN VÀO ĐÂY]                                   -->
<!-- Danh sách thành viên:                                                      -->
<!-- 1. [Họ và tên Trưởng nhóm] - [Email]                                       -->
<!-- 2. [Họ và tên Thành viên 2] - [Email]                                      -->
<!-- 3. [Họ và tên Thành viên 3] - [Email]                                      -->
<!-- 4. [Họ và tên Thành viên 4] - [Email]                                      -->
<!-- 5. [Họ và tên Thành viên 5 (nếu có)] - [Email]                              -->
<!-- 6. [Họ và tên Thành viên 6 (nếu có)] - [Email]                              -->
<!-- ========================================================================== -->

# 01 — Báo Cáo Quét Cơ Hội & Thẻ Bài Toán Vận Hành (Vin Smart Future)

**Đơn vị nghiên cứu:** Vin Smart Future — Khối Công nghệ Tập đoàn Vingroup  
**Vị trí công tác:** AI Product Engineer  

---

## 🧭 Tổng quan khảo sát vận hành tại Vingroup
Với sứ mệnh tối ưu hóa hiệu năng vận hành và trải nghiệm khách hàng xuyên suốt các đơn vị thành viên Vingroup, nhóm kỹ sư AI Product Engineer đã tiến hành khảo sát thực địa các luồng tác nghiệp tại: Xanh SM (GSM), VinFast, Vinhomes và Vinpearl. Mục tiêu là bóc tách các "điểm nghẽn" (bottlenecks) tốn nhiều nhân lực để áp dụng các giải pháp trí tuệ nhân tạo phù hợp.

---

# 🔍 Phase 1 — SCAN: Bảng Quét Cơ Hội (Áp dụng 4 Lenses)

| # | Đơn vị thành viên | Lens phân tích | Mô tả chi tiết bài toán vận hành & Điểm nghẽn |
|:---:|:---|:---|:---|
| **1** | **Xanh SM (GSM)** | **Tốn thời gian** | **Điều phối khẩn cấp sự cố cạn pin xe taxi điện:** Khi tài xế báo pin xuống dưới ngưỡng an toàn trên lộ trình chở khách, điều phối viên mất từ 12-15 phút để tra cứu GPS thủ công, kiểm tra tình trạng trụ sạc VinFast còn trống và soạn thảo tin nhắn chỉ dẫn/gọi xe cứu hộ. |
| **2** | **VinFast** | **AI-upgrade** | **Chẩn đoán sơ bộ sự cố xe qua mô tả của tài xế:** Tiếp nhận các phản ánh kỹ thuật (như tiếng kêu gầm xe, lỗi sạc chậm) qua văn bản tiếng Việt để tự động trích xuất mã lỗi kỹ thuật ban đầu, thay vì để nhân viên kỹ thuật đọc từng phản ánh. |
| **3** | **Vinhomes** | **Lặp lại** | **Phân loại và gán nhãn phản ánh của cư dân:** Mỗi ngày hàng nghìn phản ánh về tiện ích căn hộ gửi về App Vinhomes Resident cần nhân sự phân loại thủ công chuyển về các tổ bảo trì, vệ sinh, an ninh. |
| **4** | **Vinpearl** | **Pain từ người khác** | **Lọc và cảnh báo phàn nàn khẩn cấp từ review du khách:** Đọc và phân loại tự động hàng nghìn bình luận trên TripAdvisor, Google Reviews để cảnh báo ngay cho quản lý khách sạn các sự cố nghiêm trọng (phòng ẩm, thái độ phục vụ kém). |
| **5** | **VinFast** | **Lặp lại** | **Đối chiếu số liệu phiên sạc điện ngoài hệ thống:** Kế toán phải dò soát thủ công các hóa đơn điện năng giữa trạm sạc đối tác và dữ liệu viễn thông trên xe VinFast theo chu kỳ tuần. |

---

# 🃏 Phase 2 — QUICK-ASSESS: 3 Thẻ Đánh Giá Nhanh (Problem Cards)

---

### 📇 QUICK PROBLEM CARD #1 — Xanh SM: Điều phối xử lý sự cố xe cạn kiệt pin

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #1                                                       │
│                                                                             │
│ Bài toán: Hỗ trợ điều phối viên phản ứng nhanh và chính xác khi tài xế taxi │
│ điện Xanh SM báo nguy cơ cạn sạch pin giữa đường.                           │
│ Đơn vị thành viên: [x] GSM (Xanh SM)                                        │
│                                                                             │
│ Ai đang gặp khó khăn (Actor)? Điều phối viên trung tâm điều vận, Tài xế.    │
│                                                                             │
│ Quy trình thủ công hiện tại:                                                │
│   1. Nhận báo cáo sự cố ──> 2. Mở định vị tìm xe                            │
│   ──> 3. Lọc trạm sạc VinFast còn trụ trống ──> 4. Soạn tin nhắn hướng dẫn  │
│   ──> 5. Kích hoạt xe sạc lưu động nếu pin chạm mức nguy cấp (< 5%)         │
│                                                                             │
│ Khâu tốn thời gian nhất? Bước 3 & 4 (mất ~10 phút/lượt).                    │
│ Vị trí AI can thiệp? Tự động đối soát GPS với trạm sạc và sinh bản thảo tin │
│ nhắn chỉ đường [DRAFT_ONLY] hoặc lệnh kích hoạt cứu hộ dạng JSON.            │
│                                                                             │
│ Chỉ số đo lường (Metrics):                                                  │
│ - Rút ngắn thời gian xử lý từ 15 phút xuống dưới 3 phút/sự cố.              │
│ - Độ chuẩn xác trong phân bổ trạm sạc tương thích đạt >= 98%.               │
│                                                                             │
│ Kiến trúc kỹ thuật đề xuất: [x] LLM Feature (Co-pilot kết hợp duyệt HITL)   │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### 📇 QUICK PROBLEM CARD #2 — VinFast: Trợ lý tiếp nhận phản ánh lỗi kỹ thuật

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #2                                                       │
│                                                                             │
│ Bài toán: Phân tích ngôn ngữ tự nhiên từ tin nhắn báo lỗi xe điện VinFast để│
│ tự động trích xuất mã linh kiện và mức độ ưu tiên xử lý cho xưởng dịch vụ.  │
│ Đơn vị thành viên: [x] VinFast                                              │
│                                                                             │
│ Ai đang gặp khó khăn (Actor)? Cố vấn dịch vụ xưởng VinFast, Chủ xe điện.    │
│                                                                             │
│ Quy trình thủ công hiện tại:                                                │
│   1. Nhận tin nhắn báo lỗi ──> 2. Đọc và hỏi thêm chi tiết                  │
│   ──> 3. Tra cứu catalog mã lỗi ──> 4. Xếp lịch hẹn sửa chữa                │
│                                                                             │
│ Khâu tốn thời gian nhất? Bước 2 & 3 (mất 20 phút/khách).                    │
│ Vị trí AI can thiệp? LLM trích xuất thực thể, triệu chứng và gợi ý mã lỗi.  │
│                                                                             │
│ Chỉ số đo lường (Metrics):                                                  │
│ - Tăng tốc độ phân loại phản ánh từ 20 phút xuống 2 phút.                   │
│ - Độ chính xác trích xuất bộ phận hỏng hóc đạt >= 90%.                      │
│                                                                             │
│ Kiến trúc kỹ thuật đề xuất: [x] LLM Feature (Information Extraction)        │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### 📇 QUICK PROBLEM CARD #3 — Vinhomes: Điều hướng phản ánh cư dân tự động

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #3                                                       │
│                                                                             │
│ Bài toán: Tự động phân luồng phản ánh từ App Vinhomes Resident đến đúng     │
│ bộ phận phụ trách (an ninh, cảnh quan, kỹ thuật tòa nhà).                   │
│ Đơn vị thành viên: [x] Vinhomes                                             │
│                                                                             │
│ Ai đang gặp khó khăn (Actor)? Nhân viên trực tổng đài ban quản lý khu đô thị│
│                                                                             │
│ Quy trình thủ công hiện tại:                                                │
│   1. Đọc nội dung ticket ──> 2. Xác định vị trí tòa nhà                     │
│   ──> 3. Chọn phòng ban phụ trách ──> 4. Gửi email thông báo                │
│                                                                             │
│ Khâu tốn thời gian nhất? Bước 1 & 3 (tồn đọng xử lý 4-8 tiếng).             │
│ Vị trí AI can thiệp? Phân loại Intent và điều hướng ticket tự động.         │
│                                                                             │
│ Chỉ số đo lường (Metrics):                                                  │
│ - Giảm độ trễ chuyển tiếp ticket từ 4 tiếng xuống dưới 1 phút.              │
│ - Tỷ lệ chuyển tiếp đúng phòng ban đạt >= 95%.                              │
│                                                                             │
│ Kiến trúc kỹ thuật đề xuất: [x] LLM Classifier                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Luận điểm chọn bài toán cho Deep-Dive

Nhóm thống nhất chọn **Quick Problem Card #1: Xanh SM — Điều phối xử lý sự cố xe cạn pin** để phân tích sâu vì:
1. **Tính cấp thiết thời gian thực (Real-time Criticality):** Sự cố hết pin giữa đường ảnh hưởng trực tiếp đến an toàn giao thông và hình ảnh thương hiệu xe điện VinFast.
2. **Hiệu quả kinh tế thấy ngay:** Giảm 80% thời gian xử lý thủ công giúp tiết kiệm hàng chục giờ công mỗi ngày và giảm tỷ lệ hủy chuyến của hành khách.
3. **Mô hình an toàn kiểu mẫu:** Đòi hỏi thiết lập ranh giới an toàn nghiêm ngặt (Operational Boundaries) bảo vệ an toàn cho phương tiện và bắt buộc có con người kiểm duyệt (Human-in-the-loop).
