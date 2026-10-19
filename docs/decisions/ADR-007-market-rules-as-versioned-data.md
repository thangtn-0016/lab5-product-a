# ADR-007 — Quy tắc vận hành là dữ liệu có phiên bản theo chợ, không phải hằng số trong code

| | |
|---|---|
| **Trạng thái** | **ĐỀ XUẤT — chưa xây.** Prototype không có thực thể chợ (sai lệch D-02) |
| **Ngày ghi** | 2026-10-04 |
| **Phạm vi** | `markets`, `market_rules`, `resolve_market_rule`, mọi truy vấn nghiệp vụ |
| **Liên quan** | [architecture.md §C6, §3.2](../system/architecture.md) · [entities.md D-02](../generated/entities.md#d-02--không-có-khái-niệm-chợ-nên-không-có-quy-tắc-riêng-từng-chợ) · [F012_MarketManagement](../features/F012_MarketManagement/technical-spec.md) · [SCR037_MarketManagement](../screens/SCR037_MarketManagement/spec.md) · [F008_IncentiveAndRateVersion](../features/F008_IncentiveAndRateVersion/technical-spec.md) |

## Bối cảnh

Đề bài LAB-4 nêu đích danh ràng buộc: *"quy tắc vận hành riêng của từng chợ"*. Tài liệu khách tham chiếu *"mua theo quy chế chợ"* cho nhóm `売買参加者`, và căn cứ `market_registration` có `violation_action = "Theo quy chế chợ"`.

Prototype không có bảng nào về chợ. Truy vấn `information_schema` tìm cột hoặc bảng chứa `market`: **0 kết quả**. Mọi quy tắc đang là toàn cục.

Đáng chú ý: **LAB-1 cũng không có màn hình nào cho việc này** — gần nhất là `SC-21` lịch nghỉ chợ. Đây là lỗ hổng từ khâu phân tích, không phải lỗi riêng của prototype.

## Lựa chọn

Thêm `markets` và `market_rules`. Quy tắc lưu dưới dạng **dữ liệu có ngày hiệu lực và phiên bản**, không phải hằng số trong code. Gắn `business_days`, `lots`, `participant_qualifications` về chợ.

Mô hình phiên bản mượn nguyên tắc `GOV-RULE-01` của khách — vốn đã quy định cho bảng tỷ lệ: tối đa 4 lần đổi mỗi năm, tách người lập và người duyệt, có ngày hiệu lực và đường quay lại bản trước.

Thiết kế dữ liệu (từ architecture §C6):

```
markets(id, code, name_ja, name_vi, service_window, timezone)
market_rules(id, market_id, rule_key, version, effective_from,
             payload jsonb, status, drafted_by, approved_by, superseded_by)
resolve_market_rule(p_market_id, p_rule_key, p_as_of date) → jsonb
```

Điểm sống còn: `resolve_market_rule` tra theo **`p_as_of` là ngày nghiệp vụ**, không theo `now()` — để in lại báo cáo kỳ cũ ra đúng con số cũ (06-04 #3, `FR-INC-03`). Phạm vi: hệ số thuế 110/100 và làm tròn xuống JPY (`BR-INC-01`) là quy tắc **quốc gia**, không đẩy vào `market_rules`.

## Phương án đã bỏ và vì sao

**Quy tắc viết thẳng vào code, mỗi chợ một nhánh `if`.** Bỏ vì thêm chợ là phải sửa code và deploy lại. Và vì quy tắc chợ thay đổi theo quyết định hành chính, không theo chu kỳ phát hành phần mềm.

**Một bảng cấu hình phẳng không có phiên bản.** Bỏ vì khi quy tắc đổi giữa kỳ, báo cáo kỳ cũ in lại sẽ ra số mới — mâu thuẫn với nguyên tắc append-only của [ADR-004](./ADR-004-invariants-at-database-layer.md) và với `FR-INC-03` (in lại báo cáo kỳ cũ phải ra đúng con số cũ).

## Hệ quả

**Được:** thêm chợ là thêm dữ liệu, không phải sửa code. Quy tắc đổi có ngày hiệu lực nên số liệu quá khứ không bị viết lại.

**Phải chịu:** mọi truy vấn nghiệp vụ phải mang theo ngữ cảnh chợ. Hiện tại không truy vấn nào có. Đây là thay đổi lan rộng, không khu trú — chạm vào cả ba view dẫn xuất và cả năm RPC.

**Phải chịu:** thêm một màn hình quản lý chợ và quy tắc, kèm luồng duyệt. Không có trong 35 màn của LAB-1.

**Phải chịu:** nếu khách thực tế chỉ vận hành một chợ, đây là trừu tượng hoá sớm. Rủi ro có thật. Đánh đổi: đề bài nêu đích danh ràng buộc này, và `violation_action = "Theo quy chế chợ"` đang trỏ tới một thực thể không tồn tại — nên dù chỉ một chợ thì mô hình vẫn phải có chỗ cho nó.

**Đường di trú (architecture §C6):** tạo `markets` với 1 dòng mặc định → backfill `market_id` → đổi `business_days` sang PK ghép `(market_id, business_date)`. Bước đổi PK là bước duy nhất phá tương thích; làm sau cùng. Chi phí chưa ước lượng — kéo theo mọi FK trỏ `business_date`.

## Câu hỏi còn mở

- Hệ thống thực tế phục vụ **một chợ hay nhiều chợ**? Quyết định này đổi mức độ ưu tiên của ADR-007 từ bắt buộc sang nên có. Nếu chỉ một chợ, C6 thu lại còn bảng quy tắc có `発効日` không cần `market_id`.
