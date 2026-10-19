# Business Context — F004_TradeRecording

## Why It Matters

Đây là **luồng chính của chợ**: giao dịch thỏa thuận trực tiếp chiếm khoảng 90% giá trị, phiên đấu giá khoảng 10%. Hiện nay nhân viên giao dịch xác nhận qua điện thoại, ghi phiếu rồi chuyển người khác nhập lại. Tính năng này cho họ chốt ngay tại bàn điều hành, và chặn chốt khi người mua đã hết hạn tham gia hoặc lô không còn đủ hàng — hai lỗi mà chợ không sửa được sau khi đã xảy ra.

Hai luồng **không đối xứng, đúng như tài liệu khách mô tả**: giao dịch thỏa thuận có đủ tạo, chốt, huỷ; phiên đấu giá diễn ra **ngoài hệ thống** bằng ký hiệu tay nên hệ thống chỉ *ghi kết quả cuối do người vận hành nhập* — không đọc ký hiệu, không đọc ảnh, không gợi ý. Coi sản phẩm là "hệ thống chỉ có đấu giá" là điều kiện bị loại trực tiếp (RFP 12-06, 13-01); vì vậy luồng thỏa thuận mới là luồng được đầu tư bàn phím và kiểm tra ở máy chủ.

Bước chốt **bắt buộc có mạng**: hai phép kiểm cần trạng thái dùng chung ở máy chủ. Chốt trên dữ liệu cũ nghĩa là có thể bán vượt lô.

Tính năng này là bước hai của [chuỗi lô → giao dịch → giao hàng → đối chiếu](../../flows/FL-01-lot-trade-delivery-reconciliation.md).

## Who Uses It

- **Nhân viên giao dịch** (`trading`) — chốt giao dịch thỏa thuận, ghi kết quả phiên đấu giá, tra lại giao dịch đã chốt. Làm trên máy dùng chung theo ca.
- **Nhân viên đối chiếu** (`reconcile`) — xem danh sách giao dịch để đối chiếu cuối ngày; chỉ xem, muốn sửa phải lập phiếu điều chỉnh.
- **Người xác nhận kết quả đấu giá tại hiện trường** — không nhất thiết là người gõ máy; hệ thống lưu riêng *tên người xác nhận* (nhập tay) và *tài khoản thao tác*.

## What They Do

1. Nhân viên giao dịch nhập mã lô, mã người mua, số lượng, đơn giá và chốt — hệ thống kiểm tư cách người mua **và** số lượng lô còn lại trong cùng một lần.
2. Nếu thiếu một trong hai, hệ thống từ chối, nêu lý do cụ thể (hết hạn từ ngày nào, căn cứ gì, liên hệ phòng nào; hoặc lô còn bao nhiêu, đã bán cho giao dịch nào) và **không tạo bản ghi nào**.
3. Nếu đủ cả hai, giao dịch được chốt, có **mã giao dịch duy nhất**; số còn lại của lô giảm đúng bằng số vừa chốt; banner xác nhận ở nguyên đó, không tự biến mất.
4. Nhân viên nhập tiếp giao dịch kế — form đã xoá sạch (không để lại người mua của ca trước), con trỏ về ô mã lô.
5. Với phiên đấu giá, người vận hành nhập tay năm thông tin: người thắng, số lượng, đơn giá, thời điểm phiên, người xác nhận — hệ thống ghi lại, không tự suy luận.
6. Nhân viên tra danh sách giao dịch theo ngày nghiệp vụ, người mua, trạng thái, loại giao dịch.
7. Khi chốt nhầm trước khi sổ ngày được chốt, nhân viên huỷ giao dịch kèm lý do (đặc tả); nếu giao dịch đã có lần giao thì phải xử lý ngoại lệ giao hàng trước.

## Unresolved Questions

- **Giao dịch đã có lần giao thì huỷ thế nào?** Chặn cứng và đẩy sang xử lý ngoại lệ giao hàng, hay cho huỷ phần chưa giao? Hai đường chồng nhau, khách cần chọn một.
- **Năm trạng thái hay hai?** Tài liệu phân tích mô tả năm trạng thái (nháp, chờ xác nhận, đã chốt, huỷ, cải chính), nhưng yêu cầu từ chối chốt lại cấm tạo bản ghi ở trạng thái chờ. Thiết kế đang giữ hai trạng thái (đã chốt, đã huỷ); "cải chính" đi qua phiếu điều chỉnh và sinh bản ghi mới.
- **Lý do huỷ/sửa** phải lưu ở đâu: thêm cột lý do vào nhật ký kiểm toán hay quy ước trong nội dung? Ảnh hưởng cả màn tra cứu nhật ký.
- **Ai được xem lịch sử sửa của giao dịch?** Hiện chỉ quản trị hệ thống đọc được nhật ký; nhân viên giao dịch và đối chiếu cần quyền đọc có điều kiện.
- **Giao dịch có cần chặn khi lô chưa "mở bán"?** Hiện hệ thống không chặn.
- **Phân trang danh sách**: khách nêu 700–1.200 giao dịch mỗi ngày; chưa có ngưỡng.
