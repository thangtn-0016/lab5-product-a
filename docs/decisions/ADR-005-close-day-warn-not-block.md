# ADR-005 — Chốt sổ cảnh báo chứ không chặn khi còn giao dịch chưa giao xong

| | |
|---|---|
| **Trạng thái** | Đã triển khai — **cần khách xác nhận cách đọc `US-11`** |
| **Ngày ghi** | 2026-10-04 |
| **Phạm vi** | RPC `close_business_day`, SC-18 |
| **Liên quan** | [architecture.md §2.1](../system/architecture.md) · [FL-01](../flows/FL-01-lot-trade-delivery-reconciliation.md) · [SCR018_DailyReconciliation](../screens/SCR018_DailyReconciliation/spec.md) · [F006_ReconciliationAndClosing](../features/F006_ReconciliationAndClosing/technical-spec.md) · entities.md [D-04](../generated/entities.md#d-04--chốt-sổ-không-chặn-khi-còn-giao-dịch-chưa-giao-xong) |

## Bối cảnh

`US-11` của LAB-1: *"Hộp thoại chốt hiện danh sách **điều kiện phải đạt** trước khi chốt."* Đọc sát chữ thì "phải đạt" nghĩa là chặn.

Nhưng `F05` của cùng tài liệu mô tả: một giao dịch giao nhiều lần *"do lịch xe gom và lịch hạ hàng"*.

## Lựa chọn

`close_business_day` trả về `pending_conditions` để giao diện cảnh báo, nhưng **vẫn cho chốt**.

**Bằng chứng triển khai:** `0008_rpc_functions_admin.sql:27-39` — truy `v_trade_delivery_status` lấy các giao dịch có `remaining > 0` vào `v_pending`, cập nhật `business_days.status='closed'`, ghi `audit_log`, trả `{business_date, closed_at, pending_conditions}`; không có nhánh `raise` nào cho trường hợp còn tồn.

## Phương án đã bỏ và vì sao

**Chặn cứng bằng mã lỗi mới.** Bỏ vì giao hàng kéo dài qua ngày là bình thường trong nghiệp vụ này. Chặn cứng thì ngày nào cũng có giao dịch chưa giao xong, và **không ngày nào chốt sổ được** — hệ thống tự khoá chính nó.

**Chặn nhưng cho ghi đè kèm lý do.** Trung thành với cả hai cách đọc, nhưng tốn thêm luồng duyệt và một màn hình. Ngoài ngân sách prototype. Ghi lại như hướng mở.

## Hệ quả

**Được:** chốt sổ hàng ngày khả thi. Người đối chiếu vẫn thấy đầy đủ việc còn tồn trước khi quyết định.

**Phải chịu:** số liệu chốt của một ngày có thể chứa giao dịch chưa giao xong. Báo cáo kỳ đó không phản ánh trạng thái giao hàng cuối cùng. Chấp nhận được vì sau chốt vẫn còn đường điều chỉnh có kiểm soát (`F07`, [FL-02](../flows/FL-02-post-close-correction.md)), nhưng phải nói rõ với người dùng.

**Phải chịu:** đây là cách đọc khác với nghĩa đen của `US-11`. Là **sai lệch có ý thức**, đã ghi vào tài liệu gốc `db-schema.md` (trong repo, không nộp). Nếu khách hiểu "phải đạt" theo nghĩa chặn thì phải đổi, và lúc đó cần thêm luồng ghi đè.

## Câu hỏi còn mở

- "Điều kiện phải đạt" trong `US-11` nên hiểu là chặn hay cảnh báo? Hiện chọn cảnh báo, cần khách xác nhận.
