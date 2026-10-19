# Edge Cases — F007_PostCloseCorrection

Nguồn: [SCR022](../../screens/SCR022_CorrectionRequestCreate/spec.md) · [SCR023](../../screens/SCR023_PendingRequestList/spec.md) · [SCR024](../../screens/SCR024_RequestDetailApproval/spec.md) mục 4, 5, 9; `0010_closed_day_guard.sql`, `lib/domain/reconciliation.ts:140-159`. Mã `P0001`/`P0002`/`P0005`/`P0009` **đã cài**; `P0020`–`P0027` là mã của bộ thiết kế, **chưa cài** (không cấp mã mới). Hiển thị theo `error.code`, không so khớp chuỗi `message`. Cột "Trạng thái": ba màn SCR022–SCR024 chỉ đặc tả; phần bảo vệ ngày đã chốt đã xây ở LAB-3.

## Lập phiếu (SCR022)

| Scenario | What Happens | User-Facing Message | Trạng thái |
|----------|--------------|---------------------|------------|
| Mã bản ghi đích không tồn tại (`P0001`) | Không tra ra bản ghi; form giữ nguyên | "Không tìm thấy bản ghi「{target_code}」." | Chỉ đặc tả |
| Ngày của bản ghi chưa chốt (`P0020`, đề xuất) | Từ chối lập phiếu; dẫn về sửa trực tiếp | "Ngày {business_date} chưa chốt sổ — hãy sửa trực tiếp, không cần phiếu điều chỉnh." | Chỉ đặc tả |
| Bản ghi đang có phiếu chờ duyệt (`P0021`, đề xuất, có thể bị khách bác) | Chặn phiếu thứ hai để tránh phải cộng dồn nhiều phiếu | "Bản ghi này đang có phiếu {request_code} chờ duyệt." | Chỉ đặc tả |
| Số sau bằng số trước (V-05 phía client; `P0022` phía server) | Không tạo phiếu | "Chưa có gì thay đổi — không cần lập phiếu." / "Phiếu không tạo ra thay đổi nào." | Chỉ đặc tả |
| Lý do dưới 10 ký tự | Chặn gửi, hiện dưới ô | "Lý do phải từ 10 ký tự — phiếu này là chứng cứ kiểm toán." | Chỉ đặc tả |
| Thiếu chứng cứ | Chặn gửi | "Phải đính kèm chứng cứ — không có chứng cứ thì phiếu không duyệt được." | Chỉ đặc tả |
| Số lượng hoặc đơn giá ≤ 0 | Chặn gửi | "Số lượng và đơn giá phải lớn hơn 0." | Chỉ đặc tả |
| Ngày ghi nhận điều chỉnh vừa bị chốt giữa chừng (`P0002`) | Từ chối; tải lại để tra ngày mới | "Ngày ghi nhận {business_date} vừa được chốt sổ. Hãy tải lại trang." | Chỉ đặc tả |
| Không còn ngày nghiệp vụ mở nào | Không có chỗ ghi điều chỉnh; không phiếu nào lập được | "Chưa có ngày nghiệp vụ đang mở để ghi nhận điều chỉnh." (`P0026`, đề xuất) | Chỉ đặc tả |
| Chức danh không phải `reconcile` (`P0005`) | Chặn ở route (đẩy về trang 403) và ở RPC | "Chức danh của bạn không được phép lập phiếu điều chỉnh." | Chỉ đặc tả |
| Mất mạng khi gửi | Dữ liệu đã gõ giữ nguyên, có nút thử lại — mất một phiếu soạn 10 phút là lỗi không chấp nhận được | "Mất kết nối — chưa gửi được phiếu. Thử lại." (suy luận từ mẫu lỗi mạng của bộ spec) | Chỉ đặc tả |
| Người dùng mở form để tab nhiều giờ | `snapshot_before` có thể cũ; không chặn lúc lập, chỉ đối chiếu lại lúc duyệt | Không có thông báo lúc lập (xử lý ở `P0024`) | Chỉ đặc tả |

## Danh sách chờ duyệt (SCR023)

| Scenario | What Happens | User-Facing Message | Trạng thái |
|----------|--------------|---------------------|------------|
| Phiếu `pending` do chính người xem lập | Ẩn hẳn nút duyệt, giữ dòng để theo dõi | "Bạn không duyệt được phiếu do chính mình lập." | Chỉ đặc tả |
| Tất cả phiếu `pending` đều của chính người xem | Rỗng biến thể — phân biệt với rỗng thật để không tưởng hệ thống hỏng | "Chỉ còn phiếu do bạn lập — cần người khác duyệt." | Chỉ đặc tả |
| Người xem không có `can_approve` | Xem được danh sách, cột hành động chỉ có liên kết "Xem" | "Bạn xem được danh sách nhưng không có quyền duyệt." | Chỉ đặc tả |
| Không có phiếu khớp bộ lọc | Danh sách trống, gợi ý xoá lọc | "Không có phiếu nào chờ duyệt." | Chỉ đặc tả |
| Cột người 起票 hiện "—" do RLS `app_users` | Người duyệt không biết phiếu của ai (lỗi nghiệp vụ, cần nới RLS) | "—" (chưa có thông báo; phải xử lý trước khi triển khai) | Chỉ đặc tả |
| Phiếu treo quá 3 ngày làm việc (đề xuất, rủi ro Thấp) | Tông cảnh báo ở cột thời gian chờ; không tự huỷ | "{n} ngày" tô nổi | Chỉ đặc tả |
| Tải danh sách thất bại | Bộ lọc nằm trên query string nên không mất | "Không tải được danh sách phiếu. Kiểm tra kết nối rồi thử lại." | Chỉ đặc tả |

## Duyệt / từ chối (SCR024)

| Scenario | What Happens | User-Facing Message | Trạng thái |
|----------|--------------|---------------------|------------|
| Người duyệt chính là người lập, đi vòng qua giao diện (`P0023`, đề xuất) | RPC từ chối; không ghi gì | "Người duyệt phải khác người 起票." | Chỉ đặc tả |
| Người duyệt không có `can_approve`, gọi thẳng RPC (`P0027`, đề xuất) | RPC từ chối | "Bạn không có thẩm quyền duyệt phiếu điều chỉnh." | Chỉ đặc tả |
| Số gốc đã đổi từ lúc lập phiếu (`P0024`, đề xuất) | Cảnh báo lệch hiện ngay khi mở màn; RPC từ chối duyệt dựa trên ảnh chụp cũ | "Số liệu gốc đã thay đổi từ lúc lập phiếu. Hãy tải lại và kiểm tra lại phiếu." | Chỉ đặc tả |
| Hai người duyệt cùng lúc (`P0025`, đề xuất) | Khoá dòng phiếu; người sau thấy phiếu đã xử lý; không sinh hai bộ bản ghi điều chỉnh | "Phiếu đã được {decided_by_name} xử lý lúc {decided_at}." | Chỉ đặc tả |
| Không có ngày nghiệp vụ mở để ghi (`P0026`, đề xuất) | Không duyệt được | "Chưa có ngày nghiệp vụ đang mở để ghi nhận điều chỉnh." | Chỉ đặc tả |
| Ngày ghi nhận vừa bị chốt giữa lúc duyệt (`P0002`) | Từ chối; tải lại | "Ngày ghi nhận {business_date} vừa được chốt sổ. Hãy tải lại." | Chỉ đặc tả |
| Từ chối mà ý kiến dưới 10 ký tự | Chặn gửi | "Từ chối phải nêu lý do từ 10 ký tự." | Chỉ đặc tả |
| Chưa chọn duyệt hay từ chối | Chặn gửi | "Phải chọn duyệt hoặc từ chối." | Chỉ đặc tả |
| `id` phiếu không tồn tại (`P0001`) | Banner đầu trang | "Không tìm thấy phiếu này." | Chỉ đặc tả |
| Chức danh ngoài `reconcile` (`P0005`) | Đẩy về trang 403 | "Chức danh của bạn không được phép mở phiếu điều chỉnh." | Chỉ đặc tả |
| Duyệt nhầm | Không có thao tác hoàn tác; chỉ lập phiếu điều chỉnh ngược | Hộp thoại xác nhận phải nói trước điều này (không có mã lỗi) | Chỉ đặc tả |
| Mất mạng lúc gửi quyết định | Hộp thoại giữ nguyên, ý kiến đã gõ còn nguyên | "Mất kết nối — chưa gửi được quyết định. Kiểm tra kết nối rồi thử lại." | Chỉ đặc tả |

## Bảo vệ chung (đã triển khai trong prototype)

| Scenario | What Happens | User-Facing Message | Trạng thái |
|----------|--------------|---------------------|------------|
| Ghi trực tiếp vào bản ghi của ngày đã chốt qua đường không có RPC (`P0009`) | Trigger `guard_closed_day` chặn INSERT/UPDATE/DELETE trên lô, giao dịch, lần giao; có thể dựng lại lần thử trong nhật ký (`blocked_write_attempt`) | "Ngày nghiệp vụ đã chốt sổ, không thể sửa." | Đã xây |
| RPC tự phát hiện ngày đã chốt trước khi ghi (`P0002`) | Từ chối kèm thời điểm chốt; Server Action gọi riêng RPC ghi vết | "Ngày nghiệp vụ đã chốt sổ" kèm ngày và giờ chốt | Đã xây |
| Ghi vết thất bại | Không ném lỗi — thông báo gốc vẫn hiện cho người dùng. Ngoại lệ (lỗi mạng) được `catch` và ghi `console.error`; còn lỗi do chính RPC trả về trong `{ error }` thì **không ai đọc** nên mất vết im lặng (khoảng trống FR-CORR-03, xem [technical-spec](./technical-spec.md#overview) và [F011](../F011_AuditLog/technical-spec.md)) | None — silent handling | Đã xây (có khoảng trống) |
