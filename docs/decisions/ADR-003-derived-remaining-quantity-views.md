# ADR-003 — Số lượng còn lại là view dẫn xuất, không phải cột lưu sẵn

| | |
|---|---|
| **Trạng thái** | Đã triển khai |
| **Ngày ghi** | 2026-10-04 |
| **Phạm vi** | Mô hình dữ liệu — `v_lot_availability`, `v_trade_delivery_status`, `v_daily_reconciliation` |
| **Liên quan** | [architecture.md §C4](../system/architecture.md) · [entities.md](../generated/entities.md#ba-view-dẫn-xuất) · [F003_LotsAndInspection](../features/F003_LotsAndInspection/technical-spec.md) · [F005_DeliveryTracking](../features/F005_DeliveryTracking/technical-spec.md) |

## Bối cảnh

`SC-08` phải hiện lô còn bao nhiêu. `SC-14` phải hiện giao dịch đã giao bao nhiêu, còn lại bao nhiêu. `SC-10` cần số còn lại để quyết định cho chốt hay từ chối.

Cách thông thường: giữ cột `remaining` trên bảng, cập nhật mỗi lần có giao dịch.

## Lựa chọn

Ba view dẫn xuất tính tại chỗ: `v_lot_availability`, `v_trade_delivery_status`, `v_daily_reconciliation`. Cột `lots.quantity` là số ban đầu, **bất biến sau khi tạo**.

**Bằng chứng triển khai:** `0006_derived_views.sql:6-62` (cả ba view, `security_invoker = true`); `0004_business_days_and_lots.sql:41-42` (comment cột `quantity` bất biến).

## Phương án đã bỏ và vì sao

**Cột `remaining` cập nhật bằng trigger.** Bỏ vì nó tạo ra hai nguồn sự thật cho cùng một con số. Khi chúng lệch — và chúng sẽ lệch, sau một lần rollback dở hoặc một migration quên cập nhật — không có cách nào biết cái nào đúng.

Đây cũng là thứ `BR-LOT-02` của khách cấm: *số lượng khả dụng không bao giờ được âm*. Một bất biến dễ giữ hơn nhiều khi con số được tính ra thay vì được lưu.

## Hệ quả

**Được:** không thể lệch. Số còn lại luôn bằng phép trừ từ dữ liệu gốc.

**Phải chịu:** mỗi lần đọc là một lần tính. Với bảng giao dịch lớn dần theo năm, đây là chỗ sẽ chậm trước tiên. Chưa gặp ở quy mô hiện tại nhưng là nợ đã biết.

**Phải chịu:** ba view bật `security_invoker = true` để RLS của bảng gốc vẫn áp dụng. Hệ quả phụ không lường trước: chức danh `receiving` không có quyền đọc `trades`, nên `v_trade_delivery_status` trả **0 dòng** cho họ — và `SC-14` không hiện được "đã giao / còn lại". Phải mở thêm quyền đọc ở `0013_trades_select_receiving.sql` (sai lệch **D-05**). Bảo mật ở tầng view kéo theo ràng buộc ở tầng bảng, không độc lập như tưởng.

**Lỗi đã phát hiện sau này (không thuộc quyết định, thuộc công thức):** B-01 — `v_trade_delivery_status` cộng cả `returned` và `partial_cancel` vào "đã giao" (`0006:42`). Xem [entities.md](../generated/entities.md#năm-lỗi-tầng-dữ-liệu-phát-hiện-khi-viết-spec).
