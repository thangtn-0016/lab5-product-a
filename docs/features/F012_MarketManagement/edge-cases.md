# Edge Cases — F012_MarketManagement

Nguồn: [SCR037](../../screens/SCR037_MarketManagement/spec.md) mục 4, 5, 9. Phủ F12-01…F12-04. Cột "Trạng thái": toàn bộ F012 là **chỉ đặc tả** (chưa xây). Mã lỗi `P0013`, `P0015`, `P0016`, `P0017`, `P0069`, `P0071` là đề xuất của bộ screen-spec, chưa cài; `P0005` đã cài nghĩa "sai chức danh".

| Scenario | What Happens | User-Facing Message | Trạng thái |
|----------|--------------|---------------------|------------|
| Ô bắt buộc để trống | Chặn gửi ở form | "Mục này bắt buộc." | Chỉ đặc tả |
| Mã chợ sai định dạng (chữ in hoa, số, gạch ngang, 2–12 ký tự) | Chặn ở form | "Mã chợ gồm chữ in hoa, số, gạch ngang, 2–12 ký tự." | Chỉ đặc tả |
| Mã chợ đã tồn tại | `P0069` (đề xuất; không dùng `P0005` vì đã cài nghĩa "sai chức danh") | "Mã chợ「{code}」đã được dùng." | Chỉ đặc tả |
| Lý do thay đổi dưới 10 ký tự | Chặn ở form | "Lý do thay đổi phải từ 10 ký tự." | Chỉ đặc tả |
| Khung giờ vận hành sai định dạng | Chặn ở form | "Khung giờ theo dạng HH:MM-HH:MM." | Chỉ đặc tả |
| Nội dung quy tắc thiếu trường theo loại | Chặn ở form | "Thiếu「{field}」cho loại quy tắc「{rule_key}」." | Chỉ đặc tả |
| Đã có 4 bản đã duyệt trong năm (theo ngày hiệu lực) | Từ chối `P0015` ở cả lưu nháp và duyệt; dữ liệu đã gõ giữ nguyên | "Quy tắc「{rule_key}」đã đổi 4 lần trong năm {year} — mức tối đa." | Chỉ đặc tả |
| Ngày hiệu lực không ở tương lai hoặc trùng bản đã duyệt | `P0016` | "Ngày hiệu lực phải ở tương lai và không trùng bản đã duyệt." | Chỉ đặc tả |
| Người lập tự duyệt bản của mình | Giao diện ẩn nút; gọi thẳng RPC vẫn bị `P0013` | "Bạn không duyệt được bản do chính mình lập." / "Người duyệt phải khác người lập và phải có thẩm quyền duyệt." | Chỉ đặc tả |
| Hai người cùng duyệt một bản | Người sau thấy bản không còn nháp; RPC từ chối `P0013` (đề xuất) | "Bản này không còn ở trạng thái nháp." (đề xuất) | Chỉ đặc tả |
| Duyệt tháng 12 cho hiệu lực tháng 1 | Tính vào năm sau; không đụng trần năm nay | — | Chỉ đặc tả |
| Giữa lúc duyệt và ngày hiệu lực | Tra theo ngày trong khoảng vẫn trả bản cũ | — | Chỉ đặc tả |
| Tra quy tắc khi chợ chưa có phiên bản nào được duyệt | Hàm raise lỗi `P0017` (không trả rỗng, để không hiểu nhầm "không hạn chế"); "không còn bản trước" là `P0071`, mã riêng | "Chợ chưa có quy chế hiệu lực tại ngày này." | Chỉ đặc tả |
| Quay lui khi không còn bản đã duyệt trước đó | `P0071` | "Không còn phiên bản trước để quay lại." | Chỉ đặc tả |
| Chỉnh bản đã duyệt/đã thay thế/đã quay lui | Mọi ô chỉ đọc; chỉ tạo bản mới | — | Chỉ đặc tả |
| `draft` bị quên lọc trong hàm tra | Bản nháp chi phối giao dịch thật (lỗi triển khai) — phải có kiểm thử VER-F012-01 | — | Chỉ đặc tả |
| Mất kết nối khi lưu | Banner + nút thử lại; dữ liệu đã gõ giữ nguyên | "Mất kết nối. Dữ liệu đã nhập vẫn được giữ." | Chỉ đặc tả |
| `trading`/`receiving` mở `/markets` | Route chặn về `/403`; RLS chặn tầng dữ liệu | "Bạn không có quyền mở trang này." | Chỉ đặc tả |
| Muốn xoá chợ | Không có đường xoá; chỉ chuyển ngừng hoạt động (đề xuất, rủi ro Thấp) | — | Chỉ đặc tả |
