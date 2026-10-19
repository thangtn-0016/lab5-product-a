# ADR-004 — Bất biến dữ liệu giữ ở tầng database, không đẩy lên tầng ứng dụng

| | |
|---|---|
| **Trạng thái** | Đã triển khai — mất 3 vòng migration (`0033` → `0034` → `0035`) |
| **Ngày ghi** | 2026-10-04 |
| **Phạm vi** | Ràng buộc `auction_session_at` / `auction_confirmed_by_name` trên `trades`; nguyên tắc chung cho mọi bất biến |
| **Liên quan** | [architecture.md §3.6](../system/architecture.md) · [SCR011_AuctionResultEntry](../screens/SCR011_AuctionResultEntry/spec.md) · [F004_TradeRecording](../features/F004_TradeRecording/technical-spec.md) · [ADR-007](./ADR-007-market-rules-as-versioned-data.md) |

## Bối cảnh

`SC-11` ghi kết quả phiên đấu giá cần hai trường nhập tay bắt buộc: thời điểm phiên và người xác nhận. Yêu cầu: giao dịch loại `auction` không bao giờ được thiếu hai trường này.

Nhưng `record_auction_result` dựng dòng qua hàm dùng chung `_confirm_trade` (không biết hai cột mới, để `NULL`), rồi mới `UPDATE` điền vào — **cùng một transaction**.

## Lựa chọn

Giữ bất biến ở tầng database bằng `CONSTRAINT TRIGGER ... DEFERRABLE INITIALLY DEFERRED`, và trong thân trigger **`SELECT` lại dòng theo `id`** thay vì tin `NEW`.

**Bằng chứng triển khai:** `0035_auction_fields_trigger_refetch.sql:24-39` (`check_auction_fields_complete()` đọc lại `v_row` theo `new.id`, raise `P0008`); `0034_auction_fields_deferred_check.sql:38` (`trg_auction_fields_complete … deferrable initially deferred`); `0030_auction_manual_fields.sql:45-60` (`record_auction_result` 6 tham số: gọi `_confirm_trade` rồi `UPDATE` hai cột).

## Phương án đã bỏ và vì sao

**`CHECK` constraint thông thường.** Đã thử ở `0030`. Hỏng: `CHECK` bắn ngay tại `INSERT`, chặn luôn dòng vừa chèn với hai cột `NULL`, trước khi `UPDATE` kịp chạy. `SC-11` hỏng hoàn toàn, trả `23514`.

**Chuyển phép kiểm lên Server Action.** Đã làm ở `0033` với lý lẽ: `trades` không có policy INSERT nên đường ghi duy nhất là RPC, ứng dụng kiểm là đủ. Lý lẽ đúng ở hiện tại — nhưng nó biến một **bất biến dữ liệu** thành một **quy ước code**. Quy ước thì người sau không biết, và một RPC mới viết sau này sẽ không có phép kiểm đó. Bỏ.

**`CONSTRAINT TRIGGER DEFERRABLE` tin vào `NEW`.** Đã thử ở `0034`. Vẫn hỏng, vì lý do tinh vi: trigger hoãn bắn **theo sự kiện** — một lần cho `INSERT`, một lần cho `UPDATE` — và mỗi lần dùng ảnh `NEW` mà câu lệnh đó chụp, không đọc lại trạng thái hiện tại lúc commit. Lần bắn gắn với `INSERT` vẫn thấy hai cột `NULL`. *"Deferred" chỉ hoãn **khi nào** kiểm, không đổi **kiểm cái gì**.*

## Hệ quả

**Được:** bất biến nằm đúng tầng thấp nhất. Kiểm chứng hai chiều: chèn thẳng SQL một dòng `auction` thiếu trường → bị từ chối `P0008` lúc commit, không sót dòng rác; luồng ứng dụng thật → thành công, hai trường lưu đúng.

**Phải chịu — và đây là giá thật:** mất **ba vòng migration** (`0033` → `0034` → `0035`) mới đúng. Hai lần đầu đều *trông như* đã giải quyết. Nếu không kiểm bằng cách chèn thẳng SQL bỏ qua ứng dụng thì cả hai đã trôi qua.

**Phải chịu:** `CONSTRAINT TRIGGER` chạy một truy vấn phụ mỗi lần ghi vào `trades`. Nhỏ, nhưng không miễn phí.

**Bài học mang đi:** khi một phép kiểm trải qua nhiều câu lệnh trong một transaction, `DEFERRABLE` là cần nhưng chưa đủ — thân trigger phải đọc lại trạng thái, không tin ảnh chụp.

**Hệ quả phụ đã gặp:** seed (`supabase/seed.sql`) từng không chạy lại được sau `0030–0035` vì insert thẳng 3 dòng `auction` thiếu hai cột → `P0008`; đã sửa ở Phase 6 (tài liệu gốc trong repo, không nộp: `db-schema.md` §11.4).
