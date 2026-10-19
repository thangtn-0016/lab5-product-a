# Business Context — F010_Notifications

## Why It Matters

Hiện nay việc cần chú ý được truyền bằng điện thoại và FAX: điện thoại không để lại dấu vết tập trung, FAX chậm và khó tra lại. Hệ quả điển hình là 14 cuộc gọi mỗi ngày cao điểm chỉ để hỏi trạng thái giao hàng, và người tham gia hết hạn chỉ bị phát hiện khi đã tới bàn giao dịch. Nhóm này đưa bốn loại việc cần chú ý vào hai kênh duy nhất khách chấp nhận (trong ứng dụng và email), và bảo đảm không có thông báo nào mất im lặng.

Phạm vi trong bộ thiết kế này gồm cả hàng chờ đồng bộ khi mất mạng ở hiện trường, vì đó là nơi người nhập liệu biết bản ghi của mình đã vào sổ hay chưa. Trạng thái hiện tại: phần thông báo chưa xây trong prototype LAB-3 (chỉ có trang chủ theo chức danh hiện số liệu thống kê); hàng chờ đồng bộ cũng chưa xây vì prototype không chạy được khi mất mạng.

Tính năng này nằm trong luồng lô, giao dịch, giao hàng, đối chiếu (flow [FL-01](../../flows/FL-01-lot-trade-delivery-reconciliation.md)): các sự kiện sinh thông báo đến từ giao hàng ứ đọng, tranh chấp, hồ sơ tư cách sắp hết hạn và lô xuất kế toán lỗi.

## Who Uses It

- **Mọi nhân viên đã đăng nhập** — đọc lại thông báo của chính mình, lọc theo loại, đánh dấu đã đọc. Trạng thái đã đọc gắn với người, không gắn với máy, vì máy dùng chung theo ca.
- **Vận hành hệ thống (ops_admin, đề xuất — chưa có trong hệ thống)** — đặt người nhận, ngưỡng và thời điểm gửi; theo dõi hàng cảnh báo khi gửi lỗi; nhận xử lý, thử lại, đóng cảnh báo.
- **Bộ phận đối chiếu (reconcile)** — xem cấu hình và hàng cảnh báo ở chế độ chỉ xem, vì lỗi batch trước 11:00 là việc của họ.
- **Nhân viên tiếp nhận và giao hàng (receiving)** — nhập khi mất mạng và theo dõi hàng chờ đồng bộ; quyết định bỏ bản ghi hoặc gửi tiếp khi nghi trùng.
- **Quản trị hệ thống (sys_admin)** — chỉ xem cấu hình và cảnh báo để hỗ trợ.

## What They Do

1. Hệ thống phát sinh một trong bốn loại sự kiện: hồ sơ sắp hết hạn, tranh chấp đang mở, lỗi xuất dữ liệu, giao hàng ứ đọng — và tạo một thông báo riêng cho từng người nhận.
2. Người nhận thấy số chưa đọc trên trang chủ, mở hộp thông báo, bấm vào một dòng để đi tới đối tượng liên quan; dòng được đánh dấu đã đọc trong cùng thao tác.
3. Thông báo quan trọng phải gửi xong trong 5 phút; email lỗi được thử lại tối đa 3 lần, sau đó vào hàng cảnh báo vận hành.
4. Người vận hành mở hàng cảnh báo, xem kết quả từng lần thử, nhận xử lý để tránh hai người cùng làm, thử lại hoặc đóng kèm lý do.
5. Người vận hành đổi người nhận, ngưỡng hoặc thời điểm gửi; thông báo đã gửi và đang chờ gửi không bị mất, cấu hình mới chỉ áp cho sự kiện phát sinh sau đó.
6. Khi mạng ở khu tiếp nhận hoặc cầu cảng mất vài phút, nhân viên vẫn nhập lô và ghi lần giao; bản ghi được giữ trên máy, hiện rõ là đang chờ, và tự gửi khi có mạng lại.
7. Nếu bản ghi nhập bù có vẻ trùng, hoặc ngày nghiệp vụ của nó đã chốt sổ, hệ thống không tự quyết: người xử lý chọn bỏ, gửi tiếp hoặc lập phiếu điều chỉnh.

## Unresolved Questions

- **Phạm vi vận hành khi mất mạng (Q-01)**: đã chốt hẹp — chỉ nhập lô và ghi lần giao chạy được khi mất mạng (PREMISE C-02); khách chưa xác nhận. Nếu khách đòi rộng hơn cần thêm màn xử lý mâu thuẫn khi đồng bộ; chênh lệch ước tính 80 đến 120 ngày công.
- **Gửi FAX (Q-15)**: khách xác nhận FAX vẫn là thao tác thủ công ngoài hệ thống, không có kết nối với FAX hay SMS?
- **Thế nào là thông báo "quan trọng"**: tài liệu khách cam kết 5 phút cho việc quan trọng nhưng không nói việc nào là quan trọng.
- **Người tham gia bên ngoài**: có nhận thông báo không, và nếu có thì chỉ qua email hay cũng có hộp thông báo trong ứng dụng?
- **Thời hạn lưu thông báo và cảnh báo**: chưa có yêu cầu lưu trữ riêng cho loại dữ liệu này.
- **Cảnh báo của chính hàng cảnh báo**: nếu kênh email hỏng thì báo bằng đường nào ngoài băng?
- **Một chợ hay nhiều chợ**: quyết định cấu hình thông báo là chung hay theo từng chợ.
- **Ngưỡng nghi trùng và thời gian giữ bản chờ**: 30 phút và 24 giờ là đề xuất của bộ thiết kế, chưa có nguồn từ khách.
