# ADR-002 — Bảng `trades` và `deliveries` không có policy INSERT/UPDATE/DELETE

| | |
|---|---|
| **Trạng thái** | Đã triển khai |
| **Ngày ghi** | 2026-10-04 |
| **Phạm vi** | RLS, RPC `SECURITY DEFINER`, quyền EXECUTE |
| **Liên quan** | [architecture.md §C2, §C3, §2.2, §5](../system/architecture.md) · [F004_TradeRecording](../features/F004_TradeRecording/technical-spec.md) · [SCR010_NegotiatedTradeConfirm](../screens/SCR010_NegotiatedTradeConfirm/spec.md) · [FL-01](../flows/FL-01-lot-trade-delivery-reconciliation.md) |

## Bối cảnh

Ranh giới cứng của đề: hai phép kiểm khi chốt giao dịch (tư cách người mua còn hiệu lực + lô còn đủ số lượng) phải chạy **trong một transaction phía server**, không được kiểm ở client.

Nhưng Supabase phơi PostgREST ra ngoài. Bất kỳ ai có publishable key — vốn công khai theo thiết kế, nằm trong JavaScript của mọi trang — đều gọi thẳng REST API được. Nếu bảng có policy INSERT, họ ghi thẳng mà không đi qua phép kiểm nào.

## Lựa chọn

Hai bảng này **không có policy ghi**. Chỉ có policy SELECT. Đường ghi duy nhất là RPC `SECURITY DEFINER`: `confirm_negotiated_trade`, `record_auction_result`, `record_delivery`.

**Bằng chứng triển khai:** `0009_rls_policies.sql:53-61` (chỉ `trades_select`, `deliveries_select`); `0007_rpc_functions.sql:41-168` (`_confirm_trade`, `confirm_negotiated_trade`, `record_auction_result`, `record_delivery`; dòng kiểm `current_app_role()` ở đầu mỗi hàm → `P0005`); `0011_lockdown_public_execute.sql` và `0041_revoke_execute_new_signatures.sql` (khoá EXECUTE).

## Phương án đã bỏ và vì sao

**Policy INSERT kèm điều kiện kiểm trong `WITH CHECK`.** Bỏ vì `WITH CHECK` đánh giá trên từng dòng sắp ghi, không khoá được dòng lô để chống hai người chốt đồng thời, và không diễn đạt được phép kiểm trải trên nhiều bảng.

**Kiểm ở tầng ứng dụng rồi mới insert.** Bỏ vì đó chính là thứ đề cấm, và vì giữa lúc kiểm và lúc ghi có khoảng trống cho người khác chen vào.

## Hệ quả

**Được:** biến quy ước "không kiểm ở client" thành **ràng buộc vật lý**. Không ai bỏ qua được, kể cả khi viết code sai. Kiểm chứng: anon gọi REST tới `confirm_negotiated_trade` nhận `42501 permission denied`, đọc mọi bảng đều trả rỗng.

**Phải chịu:** logic nghiệp vụ dồn vào PL/pgSQL. Khó test hơn TypeScript, không có kiểu tĩnh, lỗi chỉ lộ lúc chạy. Và đã lộ thật — xem [ADR-004](./ADR-004-invariants-at-database-layer.md).

**Phải chịu:** mỗi lần thêm một đường ghi mới là phải viết một RPC mới, không chỉ thêm một policy. Chi phí thay đổi cao hơn.

**Một vết đã gặp:** `0030` tạo lại `record_auction_result` với **chữ ký sáu tham số** thay vì bốn. Trong PostgreSQL đó là một **hàm khác**, nên lệnh `REVOKE` ở `0008` không còn áp — anon vào được tận thân hàm, chỉ bị chặn bởi dòng kiểm chức danh bên trong. Vá ở `0041`. Bài học: mọi migration đổi chữ ký hàm phải revoke lại, và phải kiểm bằng `has_function_privilege` chứ không tin lệnh đã viết.

**Bài học EXECUTE bổ sung** (tài liệu gốc trong repo, không nộp: `db-schema.md` §7.1, §11): `REVOKE ... FROM anon` không đủ vì `PUBLIC` được cấp EXECUTE khi `CREATE FUNCTION`; `REVOKE ... FROM PUBLIC` cũng không đủ cho hàm tạo mới vì project có default privilege cấp EXECUTE trực tiếp cho `anon`. Phải revoke **cả hai** rồi xác minh bằng `has_function_privilege`.
