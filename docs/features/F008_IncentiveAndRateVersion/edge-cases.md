# Edge Cases — F008_IncentiveAndRateVersion

Nguồn: [SCR025](../../screens/SCR025_IncentiveResult/spec.md) · [SCR026](../../screens/SCR026_RateVersionList/spec.md) · [SCR027](../../screens/SCR027_RateVersionApproval/spec.md) mục 4, 5, 9. `P0005` **đã cài** (sai chức danh); `P0030`–`P0038` là mã của bộ thiết kế, **chưa cài** (không cấp mã mới). Hiển thị theo `error.code`, không so khớp chuỗi `message`. Cột "Trạng thái": cả ba màn chỉ đặc tả (chưa xây).

## Kết quả tiền khuyến khích (SCR025)

| Scenario | What Happens | User-Facing Message | Trạng thái |
|----------|--------------|---------------------|------------|
| Chưa chọn kỳ | Không tải bảng | "Chọn kỳ trước khi xem kết quả." | Chỉ đặc tả |
| Kỳ chưa chạy tính lần nào | Bảng rỗng, nút "Tính kỳ này" chỉ với `reconcile` | "Kỳ này chưa có kết quả tính." | Chỉ đặc tả |
| Bấm tính trên kỳ đã chốt (`P0035`, đề xuất) | Từ chối; kết quả kỳ cũ không đổi | "Kỳ {period} đã chốt — không tính lại. Điều chỉnh sẽ sinh chênh lệch ở kỳ {next_period}." | Chỉ đặc tả |
| Không có bản 料率 hiệu lực tại ngày nghiệp vụ (`P0034`, đề xuất) | Không tính, thay vì ngầm coi tỷ lệ bằng 0 | "Không có phiên bản 料率 hiệu lực cho ngày {business_date}." | Chỉ đặc tả |
| Kỳ còn ngày nghiệp vụ chưa chốt (`P0038`, đề xuất) | Cảnh báo, **vẫn cho chạy**; kết quả đánh dấu tạm, lần chạy lại ghi đè | "Kỳ {period} còn {n} ngày nghiệp vụ chưa chốt — tính ra số tạm." | Chỉ đặc tả |
| Chức danh khác `reconcile` gọi thẳng RPC tính (`P0005`) | Từ chối | "Bạn không có quyền chạy tính cho kỳ này." | Chỉ đặc tả |
| Thanh toán quá hạn | `incentive_amount = 0`, `zero_reason = overdue`; không giảm dần, không gia hạn | Nguyên nhân bằng 0 hiện ở cột riêng (không phải lỗi) | Chỉ đặc tả |
| Chưa có ngày thanh toán xong tại lúc tính | `incentive_amount = 0`, `zero_reason = unpaid` | Nguyên nhân bằng 0 hiện ở cột riêng | Chỉ đặc tả |
| Số thập phân sau hệ số thuế | Làm tròn **xuống** theo JPY (không làm tròn thường) | Cột "Số trước làm tròn" hiện để giải thích chênh lệch | Chỉ đặc tả |
| Dưới ngưỡng tối thiểu (đề xuất, rủi ro Trung bình — nếu khách xác nhận có ngưỡng) | `zero_reason = below_threshold` | Nguyên nhân bằng 0 hiện ở cột riêng | Chỉ đặc tả |
| Điều chỉnh sau chốt kỳ | Không sửa dòng cũ; kỳ kế tiếp có dòng `delta` kèm kỳ gốc và lý do | Dòng "Chênh lệch" có nhãn cảnh báo | Chỉ đặc tả |
| Mất mạng | Bộ lọc và vị trí cuộn giữ nguyên | "Không tải được dữ liệu. Thử lại." (suy luận từ mẫu lỗi mạng của bộ spec) | Chỉ đặc tả |
| Chức danh `rule_admin` hoặc kỳ đã chốt | Ẩn nút "Tính kỳ này"; bảng vẫn xem và xuất CSV được | None — chế độ chỉ đọc | Chỉ đặc tả |

## Danh sách và quay lui (SCR026)

| Scenario | What Happens | User-Facing Message | Trạng thái |
|----------|--------------|---------------------|------------|
| Đang có một bản nháp hoặc chờ duyệt | Chặn lập bản mới | "Đang có bản v{n} chưa xử lý xong. Hoàn tất hoặc huỷ bản đó trước." | Chỉ đặc tả |
| Đã đủ 4 bản duyệt trong năm (`P0030`, đề xuất) | Từ chối lập, duyệt hoặc quay lui | "料率 đã đổi 4 lần trong năm {year} — mức tối đa theo quy chế." | Chỉ đặc tả |
| Quay lui trên dòng không phải bản đã thay thế | Nút bị vô hiệu | "Chỉ quay lui được về phiên bản đã từng có hiệu lực." | Chỉ đặc tả |
| Không còn bản trước để quay lại (`P0033`, đề xuất) | Từ chối | "Không còn phiên bản trước để quay lại." | Chỉ đặc tả |
| Người quay lui không có `can_approve` (`P0032`, đề xuất) | Từ chối | "Quay lui cần thẩm quyền 承認." | Chỉ đặc tả |
| Ngày hiệu lực của bản quay lui không ở tương lai hoặc trùng bản đã duyệt (`P0031`, đề xuất) | Từ chối trong hộp thoại | "発効日 phải ở tương lai và không trùng bản đã duyệt." | Chỉ đặc tả |
| Quay lui bị tính vào trần | Quay lui tạo bản `approved` mới nên tốn một suất (đề xuất, chờ khách xác nhận) | Hộp thoại nêu số lần còn lại trong năm (suy luận) | Chỉ đặc tả |
| Chức danh `reconcile`/`sys_admin` | Ẩn "Lập bản mới" và "Quay lui" | None — chế độ chỉ đọc | Chỉ đặc tả |
| Chưa có phiên bản nào | Rỗng, nút "Lập bản đầu tiên" chỉ với `rule_admin` | "Chưa có phiên bản 料率 nào." | Chỉ đặc tả |

## Lập và duyệt phiên bản (SCR027)

| Scenario | What Happens | User-Facing Message | Trạng thái |
|----------|--------------|---------------------|------------|
| Tỷ lệ ngoài (0, 1] hoặc quá 4 chữ số thập phân | Chặn lưu | "交付率 phải lớn hơn 0, không quá 1, tối đa 4 chữ số thập phân." | Chỉ đặc tả |
| Tỷ lệ bằng bản đang hiệu lực | Chặn lưu — bản mới không đổi gì là vô nghĩa | "交付率 không đổi so với bản v{n} — không cần bản mới." | Chỉ đặc tả |
| Lý do thay đổi dưới 10 ký tự | Chặn lưu | "Lý do thay đổi phải từ 10 ký tự." | Chỉ đặc tả |
| Số ngày tới hạn thanh toán ngoài [0, 180] | Chặn lưu | "Số ngày tới 支払期日 là số nguyên từ 0 đến 180." | Chỉ đặc tả |
| Ngày hiệu lực không ở tương lai hoặc trùng bản đã duyệt (`P0031`, đề xuất) | Từ chối | "発効日 phải ở tương lai và không trùng bản đã duyệt." | Chỉ đặc tả |
| Người duyệt là người lập hoặc không có thẩm quyền (`P0032`, đề xuất) | Ẩn hẳn nút duyệt với người lập; RPC vẫn từ chối khi gọi thẳng | "Người duyệt phải khác người lập và phải có thẩm quyền 承認." / "Bạn không duyệt được bản do chính mình lập." | Chỉ đặc tả |
| Người khác vừa xử lý bản này (`P0036`, đề xuất) | Từ chối | "Bản v{n} đã được xử lý — trạng thái hiện tại là {status}." | Chỉ đặc tả |
| Giá trị tỷ lệ vượt khoảng khi server kiểm lại (`P0037`, đề xuất) | Từ chối | "交付率 {value} nằm ngoài khoảng cho phép." | Chỉ đặc tả |
| Duyệt bản khi trần đã đạt giữa hai thời điểm (có bản khác được duyệt chen vào) | `P0030` lúc duyệt dù lúc lưu nháp còn chỗ | "料率 đã đổi 4 lần trong năm {year} — mức tối đa theo quy chế." | Chỉ đặc tả |
| Bản bị trả lại | Về `draft`, lý do hiện trong khối dấu vết; người lập sửa và gửi lại; không xoá dòng | "Đã trả lại v{n} cho người lập." | Chỉ đặc tả |
| Chỉ có một tài khoản `rule_admin` | Luồng không chạy được — đúng thiết kế | None (vấn đề dữ liệu demo) | Chỉ đặc tả |
| Mất mạng | Dữ liệu đã gõ giữ nguyên, có nút thử lại | "Mất kết nối. Dữ liệu đã nhập vẫn được giữ." (suy luận từ mẫu lỗi mạng của bộ spec) | Chỉ đặc tả |
