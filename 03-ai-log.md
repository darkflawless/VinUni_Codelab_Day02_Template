<!-- Tên nhóm : safe-drive
Thành viên :
Nguyễn Đức Phát : nguyenducphat.edu@gmail.com
Đỗ Thành Đạt : dothanhdat10x@gmail.com
Nguyễn Ngọc Minh : nnminh433@gmail.com
Phạm Quang Đạt : phamqdat99@gmail.com
Lâm Hoàng Phúc : lamhoangphuc2003st@gmail.com
Chử Trần Phương Nam : tranphuongnam932004@gmail.com -->

# Phase 6 — AI Log & Reflection: Nhật ký tương tác AI

> *Phản ánh trung thực về quá trình dùng AI (Gemini/Kiro) làm thought-partner trong buổi lab hôm nay.*

---

## AI giúp gì trong buổi lab?

### 1. Brainstorm bài toán (Phase 1)
Khi chưa nghĩ ra đủ 5 bài toán, dùng prompt:
> *"Tôi là AI Engineer tại Vin Smart Future. Gợi ý 5 pain point vận hành cụ thể tại Xanh SM có thể tối ưu bằng AI."*

AI trả về danh sách có hướng đúng, bao gồm bài toán sự cố pin tài xế — bài toán nhóm chọn để Deep-Dive. Tiết kiệm được thời gian blank-page ban đầu.

### 2. Cấu trúc Problem Statement
AI giúp sắp xếp Problem Statement 6 trường theo thứ tự logic: Actor → Workflow → Bottleneck → Impact → Metric → Boundary. Đặc biệt hữu ích khi nhắc đến việc phải phân biệt loại cổng sạc CCS2/Type 2 giữa các dòng xe VinFast — chi tiết kỹ thuật quan trọng mà nếu bỏ qua sẽ làm solution sai ngay từ đầu.

### 3. Viết System Prompt (Phase 4)
AI giúp cấu trúc System Prompt theo thứ tự: vai trò → phạm vi được phép → quy tắc cứng → định dạng output. Sau đó tự chạy adversarial test để kiểm chứng ranh giới giữ vững.

### 4. Tra cứu số liệu có nguồn
AI tìm được số liệu thực từ Mordor Intelligence (thị phần 54.51% Q1/2026), GSM (100 triệu lượt khách, 30.000+ xe), và xác nhận thông tin thực tế về pin xe điện VinFast từ kỹ sư VinFast tại Hà Nội *(vietnam.vn 2026)*. Những số liệu này có thể dùng trực tiếp trong báo cáo mà không cần tự bịa.

---

## AI sai ở đâu / hallucination?

### Lần 1 — Bịa số liệu vận hành nội bộ
Khi hỏi "Xanh SM có bao nhiêu sự cố pin mỗi ngày?", AI trả lời con số cụ thể (~120 sự cố/ngày) mà không có nguồn. Đây là hallucination — AI không có dữ liệu nội bộ của Xanh SM nhưng vẫn đưa ra con số như thật.

**Cách phát hiện:** Hỏi lại "nguồn từ đâu?" — AI thừa nhận không có nguồn.

**Cách xử lý:** Không dùng con số đó. Thay bằng ước tính có logic rõ ràng: *"0.3% đội xe 30.000 xe = ~90 sự cố/ngày"* — con số tương tự nhưng có căn cứ và người đọc biết đây là ước tính, không phải số thực đo được.

### Lần 2 — Đề xuất kiến trúc thiếu HITL
Khi yêu cầu thiết kế Future-State Flow, AI ban đầu đề xuất AI tự động gửi tin cho tài xế không cần dispatcher duyệt — lý do là "nhanh hơn". Đây là sai về operational boundary: trong tình huống an toàn thực địa (xe hết pin giữa đường), tin sai = xe cạn pin = nguy hiểm thực sự.

**Cách xử lý:** Thêm constraint rõ ràng vào System Prompt: `[DRAFT_ONLY]` bắt buộc, dispatcher phải phê duyệt trước khi gửi. Chạy adversarial test để verify AI giữ đúng rule này.

### Lần 3 — Viết business impact quá lạc quan
AI viết "rò rỉ doanh thu ~15%" mà không có căn cứ. Con số nghe có vẻ hợp lý nhưng thực ra là đoán mò.

**Cách xử lý:** Bỏ con số đó. Thay bằng logic định tính: "Mỗi phút tài xế chờ = không đón được khách mới = mất doanh thu trực tiếp" — không có số cụ thể nhưng không bịa.

---

## Mình sửa prompt ra sao?

| Vấn đề phát hiện | Cách sửa prompt |
|---|---|
| AI bịa số liệu | Thêm: *"Chỉ dùng số liệu có nguồn rõ ràng. Nếu không có nguồn, ghi rõ đây là ước tính và nêu cơ sở tính."* |
| AI bỏ HITL | Thêm vào System Prompt: `[DRAFT_ONLY]` bắt buộc, không tự gửi. Verify bằng adversarial test. |
| AI bỏ qua loại cổng sạc | Thêm context kỹ thuật: *"VF5 dùng Type 2, VF8/VF9 dùng CCS2 — bắt buộc khớp khi đề xuất trạm sạc."* |
| AI đề xuất kiến trúc phức tạp hơn cần thiết | Thêm constraint: *"Ưu tiên giải pháp đơn giản nhất đủ giải quyết bài toán. Không dùng Agentic Loop nếu LLM Feature là đủ."* |

---

## Tổng kết

AI hữu ích nhất ở **brainstorm ban đầu** và **tra cứu số liệu có nguồn** — giúp không mất thời gian tìm kiếm thủ công. Nhưng có 2 loại output của AI cần kiểm tra kỹ trước khi dùng:

- **Số liệu vận hành nội bộ** (sự cố/ngày, thời gian xử lý): AI không có, sẽ bịa nếu không hỏi rõ. Phải tự đo hoặc ước tính có logic.
- **Quyết định về ranh giới an toàn**: AI có xu hướng đề xuất tự động hóa tối đa. Người thiết kế phải tự đánh giá rủi ro và chủ động thêm HITL vào những điểm cần thiết.

> **Takeaway thực tế:** AI giúp tăng tốc giai đoạn đầu, nhưng chất lượng bài scoping phụ thuộc vào người dùng biết **verify số liệu**, **đặt đúng ranh giới an toàn**, và **không tin con số nào AI đưa ra mà không hỏi nguồn**.
