<!-- Tên nhóm : safe-drive
Thành viên :
Nguyễn Đức Phát : nguyenducphat.edu@gmail.com
Đỗ Thành Đạt : dothanhdat10x@gmail.com
Nguyễn Ngọc Minh : nnminh433@gmail.com
Phạm Quang Đạt : phamqdat99@gmail.com
Lâm Hoàng Phúc : lamhoangphuc2003st@gmail.com
Chử Trần Phương Nam : tranphuongnam932004@gmail.com -->
# Phase 3 — DEEP-DIVE: Xanh SM Xử lý sự cố sạc pin thực địa

> **Bối cảnh có nguồn:**
> - Xanh SM chiếm **54.51% thị phần** taxi công nghệ Việt Nam Q1/2026, vận hành **30.000+ xe điện** tại 34 tỉnh thành, phục vụ **100+ triệu lượt khách** *(Mordor Intelligence, GSM)*
> - Pin xe điện VinFast thực tế thấp hơn công bố do điều kiện đường, thời tiết, tốc độ, tải trọng *(VinFast engineer tại Hà Nội, vietnam.vn 2026)*
> - VF5 dùng cổng Type 2, VF8/VF9 dùng CCS2 — dispatcher phải phân biệt khi tra trạm *(VinFast technical specs)*

---

## 3.1. Current-State Workflow Mapping

```
┌──────────────────┐   ┌──────────────────┐   ┌──────────────────┐   ┌──────────────────┐
│    BƯỚC 1        │   │    BƯỚC 2        │   │    BƯỚC 3 🔴     │   │    BƯỚC 4 🔴     │
│                  │   │                  │   │                  │   │                  │
│  Tài xế gọi     │   │  Dispatcher tra  │   │  Tra thủ công    │   │  Soạn tin nhắn  │
│  hotline báo    │──▶│  vị trí xe GPS   │──▶│  trạm V-Green    │──▶│  hướng dẫn      │
│  hết pin        │   │  trên bản đồ     │   │  còn trụ trống   │   │  đường đi, gửi  │
│                  │   │  nội bộ          │   │  + khớp loại     │   │  qua App tài xế │
│  Công cụ: Phone │   │  Công cụ: Map    │   │  cổng sạc        │   │  Công cụ: App   │
│                  │   │  🔄 Handoff:     │   │  Công cụ:        │   │  tài xế GSM     │
│                  │   │  Tài xế →        │   │  Dashboard thủ   │   │  🔄 Handoff:    │
│                  │   │  Dispatcher      │   │  công            │   │  Dispatcher →   │
└──────────────────┘   └──────────────────┘   └──────────────────┘   │  Tài xế          │
                                                                       └────────┬─────────┘
                                                                                │
                                                                                ▼
                                                                       ┌──────────────────┐
                                                                       │    BƯỚC 5        │
                                                                       │  Điều xe cứu hộ  │
                                                                       │  pin di động     │
                                                                       │  (nếu pin < 5%)  │
                                                                       └──────────────────┘

🔴 Bottleneck: Bước 3 & 4 — tra trạm + soạn tin thủ công
🔄 Handoff: 2 điểm (Tài xế→Dispatcher; Dispatcher→Tài xế qua App)
⚠️  Thời gian xử lý thực tế: cần đo baseline tại trung tâm điều vận
    Ước tính có căn cứ: với 5 bước, mỗi bước đơn giản nhất ~2 phút,
    riêng bước tra trạm + soạn tin có thể 5-10 phút/lượt tùy tải.
```

---

## 3.2. Problem Statement (6-field)

| Field | Nội dung chi tiết |
|---|---|
| **1. Actor / Operator** | Điều phối viên (Dispatcher) tại Trung tâm Điều vận Xanh SM — vận hành mạng lưới 30.000+ xe điện VinFast trên 34 tỉnh thành. Tốc độ mở rộng đội xe tỉ lệ thuận với tần suất sự cố pin mà dispatcher phải xử lý. |
| **2. Current Workflow** | Khi tài xế báo hết pin, dispatcher thực hiện 5 bước thủ công: tra GPS xe, mở Dashboard V-Green tìm trụ sạc trống phù hợp loại cổng (CCS2/Type 2), soạn tin nhắn chỉ đường bằng tiếng Việt, gửi qua App tài xế GSM. Toàn bộ thủ công, không tự động hóa bước nào. |
| **3. Bottleneck** | Bước 3 & 4: Tra thủ công trạm V-Green còn trụ trống *và* khớp đúng loại cổng sạc với dòng xe (VF5/VF8/VF9), sau đó soạn tin nhắn hướng dẫn tự nhiên bằng tiếng Việt. Đây là 2 bước tốn thời gian nhất và dễ sai nhất (chỉ sai loại cổng sạc là tài xế không sạc được). |
| **4. Business Impact** | Xanh SM vận hành 30.000+ xe. Ngay cả khi chỉ **0.3% đội xe** gặp sự cố pin mỗi ngày = ~90 sự cố/ngày toàn quốc *(ước tính bảo thủ có căn cứ từ quy mô đội xe)*. Mỗi sự cố tài xế không đón khách được trong thời gian chờ = mất doanh thu trực tiếp. Với 30.000 xe và thị phần 54.51%, mỗi phút tối ưu được nhân với quy mô lớn có tác động đáng kể đến năng lực vận hành. |
| **5. Success Metric** | 1. **Thời gian xử lý:** Giảm thời gian dispatcher xử lý mỗi sự cố *(baseline cần đo thực tế trước khi triển khai)*. 2. **Độ chính xác:** Tỉ lệ tin hướng dẫn đúng loại cổng sạc + đúng trạm trống đạt ≥ 98%. 3. **Tải dispatcher:** Số ca sự cố 1 dispatcher xử lý được trong 1 giờ tăng lên. |
| **6. Operational Boundary** | **Được phép:** Tra API định vị xe GPS, API trạm V-Green (trụ trống, loại cổng, khoảng cách), soạn tin nháp `[DRAFT_ONLY]`. **TUYỆT ĐỐI CẤM:** Tự gửi tin khi chưa có dispatcher phê duyệt (HITL bắt buộc); đề xuất trạm > 5km khi pin < 5% (thay bằng dispatch mobile charger); chỉ trạm sai loại cổng sạc. |

---

## 3.3. Future-State Flow & AI Fit

**AI Fit:** `LLM Feature` — Không cần Agentic Loop vì quy trình có cấu trúc cố định. Nếu AI tự động gửi tin sai trạm sạc khi xe gần hết pin, hậu quả là xe cạn kiệt giữa đường — rủi ro an toàn thực địa không thể chấp nhận.

```
┌──────────────────┐   ┌──────────────────┐   ┌──────────────────┐   ┌──────────────────┐
│    BƯỚC 1        │   │    BƯỚC 2 🔵     │   │    BƯỚC 3 🔵     │   │    BƯỚC 4 🟢     │
│                  │   │                  │   │                  │   │                  │
│  Tài xế gọi     │   │  Auto-pull:      │   │  AI soạn tin     │   │  Dispatcher      │
│  hotline báo    │──▶│  vị trí GPS +    │──▶│  nháp [DRAFT_    │──▶│  review, click   │
│  hết pin        │   │  mức pin +       │   │  ONLY] hướng     │   │  xác nhận,       │
│                  │   │  loại cổng sạc + │   │  dẫn đến trạm    │   │  gửi cho tài xế  │
│                  │   │  trạm V-Green    │   │  phù hợp nhất    │   │  qua App GSM     │
│                  │   │  trống gần nhất  │   │                  │   │                  │
└──────────────────┘   └──────────────────┘   └──────────────────┘   └──────────────────┘

                              ⚠️  PHÂN NHÁNH — Pin < 5%:
                              AI không soạn tin chỉ trạm sạc.
                              Trả về: {"action": "dispatch_mobile_charger",
                                       "reason": "Pin dưới ngưỡng an toàn,
                                                  không thể đến trạm xa"}
                              Dispatcher điều xe cứu hộ pin di động.

                              ↩️  FALLBACK:
                              Nếu AI draft lỗi / không đủ tự tin,
                              dispatcher tự soạn thủ công như quy trình cũ.

🔵 AI Step  |  🟢 Human-in-the-loop (HITL bắt buộc)  |  ↩️ Fallback
```

---

# Phase 5 — EVALUATE

## AI Readiness Checklist

| # | Tiêu chí | Đánh giá |
|---|---|---|
| 1 | Có sẵn dữ liệu mẫu/logs sạch để test? | ✅ Có — hệ thống điều vận GSM đã có GPS tracking xe thời gian thực, log cuộc gọi sự cố, API trạm sạc V-Green. Không cần thu thập dữ liệu mới, chỉ cần trích xuất logs để đo baseline trước khi triển khai. |
| 2 | Rủi ro khi AI sai nằm trong tầm kiểm soát? | ✅ Có — HITL bắt buộc (dispatcher phê duyệt trước khi gửi), rule cứng pin < 5% → mobile charger được kiểm chứng qua adversarial test trong prototype Python, fallback về thủ công khi AI lỗi. |
| 3 | Stakeholders sẵn sàng thay đổi quy trình? | ✅ Có — dispatcher đang xử lý lượng sự cố tỉ lệ thuận với quy mô 30.000+ xe đang tăng. Công cụ hỗ trợ giảm tải trực tiếp, không thay thế vai trò dispatcher. |

## Quyết định cuối cùng

**[x] GO — Bắt đầu xây dựng Prototype với scope hẹp.**

**Justification:**

Ba điều kiện GO đều đạt:

1. **Dữ liệu sẵn có, không cần thu thập mới.** GPS, API trạm V-Green, log sự cố đều đã tồn tại trong hệ thống vận hành GSM. Việc cần làm trước khi triển khai là đo baseline thực tế (thời gian xử lý/sự cố) tại 1 trung tâm điều vận để có số so sánh rõ ràng.

2. **Rủi ro kiểm soát được qua thiết kế.** LLM Feature + HITL bắt buộc + fallback thủ công là lớp bảo vệ đủ mạnh. Rule cứng pin < 5% đã được kiểm chứng trong prototype. Giải pháp không thay thế dispatcher mà làm giảm tải cho họ.

3. **Quy mô đội xe tạo business case rõ ràng.** Xanh SM đang vận hành 30.000+ xe và chiếm 54.51% thị phần taxi công nghệ *(Mordor Intelligence Q1/2026)*. Bất kỳ cải thiện nào về tốc độ xử lý sự cố đều nhân với quy mô lớn — ngay cả cải thiện nhỏ cũng tạo ra tác động vận hành đáng kể.

**Scope khởi đầu:** Triển khai thử nghiệm tại 1 Trung tâm Điều vận Hà Nội, đo baseline trước và sau 2 tuần, quyết định rollout dựa trên số thực.
