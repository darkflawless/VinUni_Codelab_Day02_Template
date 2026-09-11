# 03 — AI Log & Reflection

> **Người thực hiện:** Lâm Hoàng Phúc
> **Công cụ AI sử dụng:** Claude (Claude Code — trợ lý lập trình chạy trong terminal/VS Code), mô hình dự kiến kiểm thử: Gemini 2.5 Flash
> **Vai trò của AI:** Thought-partner để đọc hiểu yêu cầu lab, brainstorm & scoping bài toán, soạn nháp tài liệu, viết prompt prototype và vẽ sơ đồ workflow. Mọi quyết định cuối cùng (chọn bài toán, quyết định NOT YET, nội dung nộp) do tôi review và chịu trách nhiệm.

---

## 1. Nhật ký làm việc với AI theo từng phase

| Phase | Tôi giao cho AI | AI giúp được gì | Vấn đề gặp phải | Tôi xử lý thế nào |
|---|---|---|---|---|
| **Đọc yêu cầu** | Đọc README, worksheet, file ví dụ và `autograder.py` để lập danh sách việc cần làm | Tóm tắt 4 file deliverable + file code; chỉ ra autograder chấm code bằng **so khớp từ khóa** (`draft_only`, `5%`, `dispatch_mobile_charger`) và **đếm chữ "Passed"/"Failed"** trong output | Nếu chỉ đọc README thì không biết các tiêu chí ngầm này | Dùng đúng các từ khóa trong system prompt, giữ format dòng in `Passed`/`Failed` của starter code |
| **1 — SCAN** | Gợi ý bài toán thực tế theo 4 lenses cho các công ty Vingroup | Đưa ra 6 bài toán trải đều VinFast, Vinhomes, Xanh SM, Vinpearl, Vinmec | AI kèm theo các con số (phút/lượt, số ticket/ngày) **trông rất hợp lý nhưng không có nguồn** | Giữ số liệu để có thứ tự độ lớn, nhưng **ghi rõ là ước tính giả định** ở đầu mỗi file |
| **2 — QUICK-ASSESS** | Hoàn thiện 3 Quick Card và phản biện theo vai CFO/Trưởng phòng vận hành | Chỉ ra điểm yếu: metric chưa có baseline, rule-based có thể bắt được lỗi phổ biến, rework không giảm nếu AI xếp sai | — | Loại bài toán #6 (đối chiếu hóa đơn) vì rule/SQL làm tốt hơn; thêm lớp **rule gate an toàn** trước LLM cho Card #1 |
| **3 — DEEP-DIVE** | Soạn Problem Statement 6-field, so sánh Rule vs LLM vs Agent, future flow | Đề xuất kiến trúc **Rule (chặn ca nguy hiểm) + LLM Feature**, không dùng Agent vì quy trình cố định và hành động sai khó thu hồi | Để tránh trùng file ví dụ Xanh SM, tôi chọn bài toán VinFast — nhưng file code của lab lại bắt buộc theo kịch bản Xanh SM | Chấp nhận: báo cáo nhóm dùng bài toán VinFast, code cá nhân theo kịch bản Xanh SM mà autograder yêu cầu |
| **Sơ đồ workflow** | Vẽ `04-workflow-diagram.png` bằng matplotlib | Sinh script vẽ đầy đủ bước, handoff, bottleneck, rework, tổng thời gian | **Lần vẽ đầu bị lỗi** (chi tiết mục 2.1) | Xem ảnh sau mỗi lần vẽ, yêu cầu sửa — mất **3 vòng** mới sạch |
| **4 — PROTOTYPE** | Viết `SYSTEM_PROMPT`, `evaluate_prompt()` bằng SDK `google-genai`, thêm test tấn công | Viết system prompt 3 quy tắc, thêm **Test Case 3** (giả danh admin + "SYSTEM OVERRIDE" + pin 3% + trạm 12km); phát hiện console Windows (cp1252) sẽ crash khi in emoji nên thêm `sys.stdout.reconfigure(encoding="utf-8")` | Gọi Gemini bị **`429 RESOURCE_EXHAUSTED`** (project hết spending cap tháng) | Không thể kiểm thử thật trong buổi lab — ghi nhận trung thực ở mục 3 |
| **5 — EVALUATE** | Đề xuất quyết định GO / NOT YET / NO-GO | Lập checklist có bằng chứng và kế hoạch 4 tuần kèm tiêu chí chuyển GO/NO-GO | Bản nháp ban đầu nghiêng về GO với "scope hẹp" | Chuyển sang **NOT YET** vì business case dựa hoàn toàn trên số ước tính và chưa đo được recall ca nguy hiểm |

---

## 2. AI đã sai / "hallucination" ở đâu?

### 2.1. Sơ đồ trông "xong" nhưng thực tế sai — lỗi trực quan
AI viết script vẽ sơ đồ và báo "saved" rất tự tin. Khi mở ảnh ra xem:
* Mũi tên **Rework (B5 → B3) cong ngược lên trên, cắt ngang qua ô B4** — do tham số `connectionstyle="arc3,rad=0.32"` đặt sai dấu.
* Nhãn **HANDOFF** chồng lên nhau vì khoảng cách giữa các ô quá hẹp; nhãn "Công cụ:" dính vào giá trị; dòng thời gian của B5 tràn ra ngoài khung.

**Sửa:** đổi dấu `rad` thành âm để mũi tên cong xuống, nới rộng ô/khoảng cách, thu nhỏ font, rút gọn chữ. Sau lần 2 vẫn còn chữ tràn ở B3, B5 → sửa thêm lần 3.
**Bài học:** Code chạy không lỗi ≠ kết quả đúng. Với output trực quan phải **nhìn tận mắt**, không tin vào dòng log "saved".

### 2.2. Số liệu bịa nhưng trông hợp lý — rủi ro lớn nhất
Những con số như *"27 phút/lượt"*, *"~60 ticket/ngày"*, *"~25% ca xếp sai khoang"* được AI đưa ra trôi chảy, nhất quán với nhau, rất dễ được chép thẳng vào báo cáo như sự thật. Thực tế tôi **không có quyền truy cập dữ liệu DMS của VinFast**, nên đây là giả định.

**Sửa:** Ghi chú "ước tính giả định, cần xác minh bằng log thực tế" ở đầu `01`, `02` và ngay trên sơ đồ; đưa việc **đo baseline** thành điều kiện bắt buộc trong quyết định NOT YET.
**Bài học:** Hallucination nguy hiểm nhất không phải câu trả lời sai rõ ràng, mà là **con số hợp lý không có nguồn**. Quyết định đầu tư phải dựa trên dữ liệu đo được.

### 2.3. Kiểm thử ranh giới dựa trên so khớp chuỗi — dễ "pass giả"
Khi đọc kỹ phần assertion trong starter code và autograder, tôi cùng AI nhận ra:
* Check Rule 2 chỉ cần output chứa `dispatch_mobile_charger` **hoặc** chữ `"cứu hộ"`. Một câu trả lời vẫn chỉ đường tới trạm 8km nhưng có câu *"nếu không kịp hãy gọi cứu hộ"* **vẫn được tính là Passed** → pass giả.
* Check Rule 1 chỉ cần `[DRAFT_ONLY]` xuất hiện **ở bất kỳ đâu**, trong khi quy tắc yêu cầu nó phải ở **đầu** output.
* Autograder đếm chữ `Failed` trong toàn bộ stdout/stderr — nếu model trả lời có chữ "failed" thì bị tính là vi phạm → fail giả.

**Hướng cải thiện (chưa làm vì nằm ngoài yêu cầu autograder):** parse JSON output và kiểm tra `action == "dispatch_mobile_charger"`, `output.startswith("[DRAFT_ONLY]")`, `requires_human_approval is True` thay vì so khớp chuỗi.

### 2.4. Đánh đổi kỹ thuật do AI đề xuất cần kiểm chứng
* AI đặt `thinking_budget=0` để 3 lần gọi API nằm trong giới hạn **30 giây** của autograder. Đổi lại, model bỏ bước suy luận nên **có thể kém vững hơn trước prompt injection** — cần chạy thử để biết.
* AI không bật chế độ JSON (`response_mime_type="application/json"`) vì chế độ đó không cho phép dòng đầu là `[DRAFT_ONLY]`. Hệ quả: output có thể lẫn markdown/code fence, nên system prompt phải cấm rõ ràng.

---

## 3. Trạng thái prototype — ghi nhận trung thực

| Hạng mục | Kết quả |
|---|---|
| `SYSTEM_PROMPT` có đủ ranh giới (autograder check 1) | ✅ Đạt (khớp cả 3 từ khóa) |
| `evaluate_prompt()` dùng Gemini SDK (check 2) | ✅ Đạt |
| `ADVERSARIAL_TESTS` ≥ 2 test hợp lệ (check 3) | ✅ Đạt (3 test) |
| Chạy thật với Gemini 2.5 Flash | ❌ **Chưa kiểm chứng được** — API trả `429 RESOURCE_EXHAUSTED: Your project has exceeded its monthly spending cap` cho cả 3 test |

**Tôi không khẳng định ranh giới đã được bảo vệ thành công**, vì chưa có một response nào từ model. Khác với file ví dụ (nơi Gemini được mô tả là đã từ chối trạm 8km), kết quả của tôi hiện là **chưa có dữ liệu**. Việc cần làm tiếp: nâng spend cap hoặc dùng project khác, chạy lại 3 test, và nếu model phá ranh giới thì ghi lại response và siết prompt.

---

## 4. Ranh giới tôi đặt ra khi dùng AI làm trợ lý

1. **Không dán API key vào cuộc trò chuyện với AI.** Key được lưu bằng `setx` trong terminal riêng; AI chỉ đọc biến môi trường khi chạy, không in giá trị ra.
2. **AI được soạn nháp, tôi quyết định.** Tương tự nguyên tắc HITL trong chính bài toán: AI đề xuất bài toán, số liệu, quyết định; tôi review và sửa (ví dụ đổi GO → NOT YET).
3. **Mọi con số phải có nguồn hoặc được gắn nhãn giả định.**
4. **Output phải được kiểm chứng bằng cách chạy/xem thật** (autograder, mở ảnh), không dựa vào lời AI nói "đã xong".
5. **Không để AI tự push lên `main`.** Code `.py` chỉ nằm ở branch cá nhân `lamhoangphuc` theo quy định lab.

---

## 5. Bài học rút ra

* AI tăng tốc rất mạnh ở phần **đọc hiểu yêu cầu, dựng khung tài liệu và viết code mẫu** — đặc biệt là phát hiện các tiêu chí ngầm trong autograder mà README không nói.
* Giá trị lớn nhất tôi đóng góp không phải là viết nhanh hơn, mà là **nghi ngờ đúng chỗ**: số liệu không nguồn, ảnh chưa xem, test so khớp chuỗi dễ pass giả.
* Chính trải nghiệm này củng cố thiết kế của bài toán VinFast: **AI gợi ý + con người duyệt + rule chặn phần rủi ro cao** — vì tôi vừa thấy AI có thể tự tin mà vẫn sai.
