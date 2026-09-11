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

# 📝 03-ai-log.md — AI Thought-Partner Reflection Log

---

## 🏛️ Thông tin Nhóm & Cá nhân
* **Tên nhóm:** safe-drive
* **Thành viên:**
  1. Nguyễn Đức Phát (nguyenducphat.edu@gmail.com)
  2. Đỗ Thành Đạt (dothanhdat10x@gmail.com)
  3. Nguyễn Ngọc Minh (nnminh433@gmail.com)
  4. Phạm Quang Đạt (phamqdat99@gmail.com)
  5. Lâm Hoàng Phúc (lamhoangphuc2003st@gmail.com)
  6. Chử Trần Phương Nam (tranphuongnam932004@gmail.com)
* **Đơn vị:** Vin Smart Future (Vingroup Technology Division)
* **Buổi Lab:** Day 02 — AI Product Scoping & Prompt Guardrail Prototyping

---

# 🤖 1. Phản ánh quá trình dùng AI làm Thought Partner

Trong buổi lab này, nhóm safe-drive đã sử dụng AI (Gemini / LLM) đóng vai trò làm **Thought Partner (Đối tác phản biện & Brainstorm)** để quét bài toán vận hành cho Vingroup, xây dựng ranh giới an toàn (Guardrails) và stress-test kịch bản tấn công prompt.

## 1.1. Prompts chính đã sử dụng

### 📌 Prompt 1: Brainstorm danh sách bài toán Vingroup
> *"Tôi là AI Engineer tại Vin Smart Future (Vingroup). Tôi đang tìm kiếm các pain point vận hành cụ thể có thể tối ưu bằng AI cho các mảng VinFast, Xanh SM, Vinhomes, Vinmec, Vinpearl. Hãy gợi ý cho tôi 5 quy trình nghiệp vụ thủ công tốn thời gian kèm con số ước tính tổn thất."*

### 📌 Prompt 2: Stress-test Quick Problem Card
> *"Đây là một thẻ bài toán vận hành tôi đề xuất cho Vin Smart Future: Xử lý sự cố cạn pin tài xế Xanh SM. Hãy đóng vai trò là một CFO và Trưởng phòng Vận hành khắt khe, chỉ ra 3 điểm yếu về logic, metric và giải thích vì sao rule-based thông thường có thể giải quyết bài toán này tốt hơn là dùng AI."*

### 📌 Prompt 3: Thiết lập System Prompt Guardrails
> *"Hãy hỗ trợ tôi thiết kế một System Prompt cho trợ lý AI Điều phối Xanh SM. Yêu cầu bắt buộc: 1) Output luôn có thẻ [DRAFT_ONLY] ở đầu. 2) Khi pin dưới 5%, cấm khuyến nghị trạm xa > 5km và phải phát lệnh dispatch_mobile_charger."*

---

## 1.2. Đánh giá sự hỗ trợ của AI (AI Recommendations vs Human Edits)

| AI Đề xuất | Quyết định của nhóm safe-drive | Lý do điều chỉnh |
|---|---|---|
| Đề xuất tạo một Autonomous Agent tự động điều hướng xe và gửi SMS trực tiếp cho tài xế Xanh SM. | **Từ chối (Reject)** -> Đưa về **LLM Feature + HITL (Dispatcher duyệt)**. | Rủi ro xe hết pin giữa đường gây tắc nghẽn rất cao nếu AI gợi ý sai trạm sạc. Cần giữ Human-in-the-loop để đảm bảo an toàn tuyệt đối. |
| Đề xuất xử lý cả bài toán khiếu nại dịch vụ tài xế Xanh SM bằng AI. | **Từ chối (Reject)** cho giai đoạn này. | Bài toán khiếu nại chưa có taxonomy rõ ràng, cần chuẩn hóa rule trước khi áp dụng AI. |
| Đề xuất cấu trúc System Prompt nghiêm ngặt phân tách thẻ `[DRAFT_ONLY]` và kịch bản `dispatch_mobile_charger`. | **Chấp nhận (Accept)** & tinh chỉnh thêm định dạng JSON. | AI gợi ý cấu trúc Guardrail rất chặt chẽ, giúp script vượt qua 100% các bài test tấn công prompt. |

---

# 💡 2. Bài học kinh nghiệm thu được

1. **AI không phải chiếc đũa thần (No Silver Bullet):** Không phải bài toán nào cũng cần AI tự trị (Agent). Việc kết hợp LLM để soạn thảo ngôn ngữ tự nhiên với Rule-based Engine để kiểm soát logic cứng (pin < 5%) đem lại hiệu quả vận hành cao nhất và an toàn nhất.
2. **Tầm quan trọng của Operational Boundaries:** Một System Prompt chuẩn sản phẩm phải định nghĩa rõ không chỉ **AI được làm gì** mà quan trọng hơn là **AI TUYỆT ĐỐI KHÔNG ĐƯỢC LÀM GÌ** (Guardrails).
3. **Phản biện đa chiều:** Sử dụng AI đóng vai CFO/Trưởng phòng Vận hành khắt khe giúp nhóm nhận ra những góc khuất về chi phí và rủi ro thực tế mà kỹ sư công nghệ thường bỏ qua.
