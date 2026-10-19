# Edge Cases — F002_ParticipantsAndQualification

Nguồn: [SCR005](../../screens/SCR005_ParticipantList/spec.md) · [SCR006](../../screens/SCR006_ParticipantProfile/spec.md) · [SCR036](../../screens/SCR036_QualificationReview/spec.md) mục 4, 9, 10; `0007_rpc_functions.sql:12-37`. Mã lỗi `P0010`–`P0019` là **dải đề xuất** của bộ thiết kế (README screen-specs), chưa cài. Chỉ `P0003` đã chạy.

| Scenario | What Happens | User-Facing Message | Trạng thái |
|----------|--------------|---------------------|------------|
| Người mua hết hiệu lực khi chốt giao dịch / kết quả đấu giá / lần giao | RPC từ chối `P0003` trong cùng transaction, **không tạo bản ghi nào**; kèm `expired_on`, `basis_type`, `basis_label`, `contact_office` | "Người mua {tên} đã hết hạn tham gia từ {ngày}, căn cứ {căn cứ}. Liên hệ {phòng} để xử lý." | Đã xây |
| Người mua không có dòng tư cách nào còn `active` trong khoảng ngày | Cùng nhánh `P0003`; thông báo dùng dòng có `valid_to` muộn nhất để nêu ngày hết hạn | như trên | Đã xây |
| Người mua có **nhiều** dòng tư cách (gia hạn tạo dòng mới) | Phép kiểm chỉ cần **một** dòng `active` trong khoảng ngày; nếu danh sách chọn nhầm dòng mới nhất theo `created_at` sẽ hiện sai trạng thái | Không có | Đã xây (kiểm); danh sách chọn dòng hiệu lực tại `as_of` là đặc tả |
| Phép kiểm dùng `current_date`, không dùng ngày nghiệp vụ của giao dịch | Giao dịch của ngày nghiệp vụ trước nhưng ghi lúc đã sang ngày mới được kiểm theo ngày hôm nay | Không có | Đã xây — **có thể sai nghiệp vụ**, chờ khách ([SCR016](../../screens/SCR016_DeliveryDetail/spec.md) mục 10) |
| Người mua ở trạng thái "hạn chế có điều kiện" vượt điều kiện (giấy phép) | Từ chối `P0010`; **không** dùng `P0003` vì người đó vẫn giao dịch được trong giới hạn | "Vượt điều kiện hạn chế của giấy phép…" (câu chữ đề xuất của spec này — chưa có trong screen spec nào, rủi ro thấp: chỉ đổi lời) | Chỉ đặc tả (đề xuất) |
| Người mua ở trạng thái "phải xét lại" (chấp thuận) | Từ chối `P0011` ngay cả khi mọi điều kiện khác hợp lệ | "Chấp thuận đang được xét lại, chưa giao dịch được." (câu chữ đề xuất của spec này — chưa có trong screen spec nào, rủi ro thấp: chỉ đổi lời) | Chỉ đặc tả (đề xuất) |
| Lập phiếu chuyển trạng thái không thuộc máy trạng thái của căn cứ | `P0012` | "Không chuyển được từ「{từ}」sang「{sang}」với căn cứ「{căn cứ}」." | Chỉ đặc tả (đề xuất) |
| Người duyệt trùng người lập, hoặc không có cờ được duyệt | `P0013`; nút duyệt ẩn sẵn ở phiếu của chính mình | "Người duyệt phải khác người đề nghị và phải có thẩm quyền duyệt." | Chỉ đặc tả (đề xuất) |
| Phiếu chuyển sang "hạn chế có điều kiện" thiếu nội dung hoặc hạn điều kiện | `P0014` | "Trạng thái「条件付き」phải có nội dung điều kiện và hạn điều kiện." | Chỉ đặc tả (đề xuất) |
| Hai người cùng xử lý một tư cách; trạng thái đã đổi trước | RPC đọc lại trạng thái trong cùng transaction; phiếu lập trên trạng thái cũ bị từ chối | "Trạng thái đã thay đổi từ lúc bạn mở phiếu. Hãy tải lại." | Chỉ đặc tả — mã `P0070` (đề xuất; cấp lại khi nộp lại 2026-10-06: trước đây trùng `P0005` "sai chức danh" đã cài) |
| Loại đối tác không khớp căn cứ gửi lên | `P0018` | "Căn cứ「{căn cứ}」không áp dụng cho loại đối tác「{loại}」." | Chỉ đặc tả (đề xuất) |
| Căn cứ chấp thuận hoặc đăng ký tại điểm giao dịch mà thiếu chợ | `P0019` | "Căn cứ「{căn cứ}」phải gắn với một chợ." | Chỉ đặc tả (đề xuất) |
| Mã người tham gia đã tồn tại | UNIQUE bị vi phạm; `P0069` (đề xuất, chưa cài) | "Mã「{mã}」đã được dùng." | Chỉ đặc tả |
| Ngưỡng cảnh báo hết hạn: chợ chưa có quy tắc hiệu lực | `P0017`; suy giảm êm — vẫn xem được danh sách, dùng mặc định 30 ngày | "Chưa có quy tắc hiệu lực cho ngưỡng cảnh báo. Đang dùng mặc định 30 ngày." | Chỉ đặc tả (đề xuất; 30 ngày là đề xuất) |
| Lọc "hạn chế có điều kiện" nhưng căn cứ đang để "tất cả" | Tự đặt căn cứ = giấy phép, hiện banner nhẹ, không chặn | "Trạng thái「条件付き」chỉ tồn tại ở căn cứ「許可」. Đã tự chuyển bộ lọc căn cứ." | Chỉ đặc tả |
| Truy cập trái chức danh (`receiving`, `sys_admin` mở danh sách người tham gia) | Lớp chặn route và RLS; hiện RLS `participants` cho **mọi role đã đăng nhập đọc** — phải siết khi xây màn | "Chức danh của bạn không được phép thực hiện thao tác này." | Chỉ đặc tả; siết RLS là thay đổi phá tương thích với màn chốt giao dịch nếu màn đó đang đọc tên người mua |
| Dữ liệu cũ (`active`/`expired`/`revoked`) khi tách thành ba nguồn | Không suy ra được "hạn chế" hay "xét lại" từ dữ liệu cũ; chỉ backfill `active→有効`, `expired→失効`, `revoked→取消` | Không có | Chỉ đặc tả (đề xuất) |
