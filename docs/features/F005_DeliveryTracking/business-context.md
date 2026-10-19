# Business Context — F005_DeliveryTracking

## Why It Matters

Một giao dịch ở chợ này thường **giao nhiều lần**, vì lịch xe gom và lịch hạ hàng không khớp nhau. Hôm nay bộ phận giao hàng cộng dồn số đã giao bằng tay trên bảng tổng hợp. Tính năng này cho họ ghi từng lần giao và thấy ngay *đã giao bao nhiêu, còn bao nhiêu*, để giao dịch đi tiếp vào đối chiếu mà không bị treo.

Có hai điểm khách hàng cố định. Thứ nhất, **quyền tham gia của người nhận phải còn hạn cả ở khâu giao hàng**, không chỉ ở khâu chốt giao dịch — dễ bị bỏ sót. Thứ hai, khi có tranh chấp về số lượng (giao thiếu, giao thừa, trả hàng, huỷ một phần), hệ thống **không tự quyết**: người xác nhận ghi lại kèm lý do, hệ thống chỉ lưu — đây là một trong bốn điểm cố ý giữ quyết định trong tay con người.

Việc ghi lần giao diễn ra ở cầu cảng, lúc mạng hay mất, nên tính năng này là một trong hai nơi khách muốn tiếp tục ghi được khi mất kết nối. Tính năng này là bước ba của [chuỗi lô → giao dịch → giao hàng → đối chiếu](../../flows/FL-01-lot-trade-delivery-reconciliation.md).

## Who Uses It

- **Nhân viên giao hàng** — tài liệu phân tích có chức danh riêng cho bộ phận giao hàng; hệ thống đang chạy gộp vào chức danh tiếp nhận (`receiving`) vì cùng ca, cùng máy, cùng cầu cảng; đã chốt giữ gộp (PREMISE C-10) và ghi là nợ. Khi làm thật nên tách lại, vì tách người nhận hàng và người giao hàng là một kiểm soát nội bộ.
- **Nhân viên đối chiếu** (`reconcile`) có cờ `can_approve` — duyệt phiếu ngoại lệ giao hàng (khác người lập; PREMISE C-09); `reconcile` nói chung xem danh sách giao hàng ứ đọng (đặc tả).
- **Nhân viên giao dịch** (`trading`) — chỉ cần biết giao dịch của mình đã giao đến đâu (quyền đọc, đề xuất).

## What They Do

1. Nhân viên giao hàng nhập mã giao dịch — hệ thống hiện ngay người nhận, mặt hàng, tổng số lượng, **đã giao**, **còn lại**, số lần đã giao, *trước khi* cho gõ số lượng lần giao mới.
2. Nhân viên nhập số lượng lần giao này, chọn trạng thái (mặc định "đã giao") và ghi — hệ thống kiểm tư cách người nhận và số còn lại trong một lần, rồi lưu thành **lần giao thứ n** của giao dịch.
3. Nếu giao vượt số còn lại, hệ thống từ chối và nói rõ đã giao bao nhiêu, còn bao nhiêu.
4. Khi giao dịch bị chia quá nhiều lần, màn hình hiện cảnh báo (không chặn).
5. Khi có sự cố (giao thiếu, giao thừa, trả hàng, huỷ một phần), nhân viên ghi ngoại lệ kèm lý do và người xác nhận; với ngoại lệ làm lệch số lượng, phiếu phải được nhân viên đối chiếu duyệt trước khi tính vào số liệu (đặc tả).
6. Khi giao đủ, nhân viên chốt hoàn tất giao hàng cho **giao dịch**: chỉ chốt được khi số lượng khớp quy tắc thanh toán và tư cách người nhận còn hiệu lực. Lô chỉ sang "đã giao xong" khi mọi giao dịch của lô đã chốt hoàn tất (đặc tả).
7. Nhân viên đối chiếu mở danh sách giao hàng, thấy các giao dịch ứ đọng và có ngoại lệ (đặc tả).
8. Khi mất mạng ở cầu cảng, nhân viên vẫn ghi lần giao vào máy; bản ghi tự gửi khi có mạng và không tạo bản trùng (đặc tả).

## Unresolved Questions

- **Giới hạn số lần chia và thời điểm ghi nhận vào thanh toán**: khách nói đây là điều kiện để chốt ước lượng nhưng chưa trả lời. Ngưỡng cảnh báo hiện là 3 lần — là **giả định** của bản thử nghiệm, không phải con số của khách.
- **"Khớp quy tắc thanh toán" nghĩa là gì?** Phải giao đúng đủ, hay có dung sai theo tỷ lệ hoặc theo mặt hàng? Đang giả định không có dung sai. Là một trong bốn điểm chặn ước lượng của khách.
- **Tư cách người nhận xét theo ngày nào?** Ngày ghi lần giao hay ngày nghiệp vụ của giao dịch? Giao hàng kéo dài qua ngày làm hai mốc này lệch nhau.
- **Chốt hoàn tất rồi có mở lại được không?** Đang chọn không, mọi sửa đi qua phiếu điều chỉnh.
- **Ai được chốt hoàn tất khi đang có tranh chấp mở?** Chặn cứng hay cảnh báo?
- **Danh sách lý do ngoại lệ dựng sẵn**: khách yêu cầu nhưng chưa cung cấp danh mục.
- **Ngưỡng duyệt ngoại lệ**: mọi lệch khác 0 đều phải duyệt, hay có dung sai? Một cấp duyệt hay nhiều cấp?
- **Phạm vi làm việc khi mất mạng** (chung với tính năng lô hàng): chỉ ghi lần giao, hay rộng hơn?
- **Ngưỡng "giao hàng ứ đọng"** là bao nhiêu ngày, đếm theo ngày lịch hay ngày nghiệp vụ? Phải thống nhất với báo cáo giao hàng ứ đọng.
- **Ba lỗi đã phát hiện ở bản thử nghiệm** (nêu ở tài liệu kỹ thuật): số "đã giao" tăng khi có hàng trả về; lô bị đánh dấu "đã giao xong" khi mới giao đủ cho một trong nhiều giao dịch của lô; và lô cũng bị đánh dấu như vậy khi lần giao vừa ghi mới ở trạng thái "Chờ giao" (mới đọc code, chưa tái hiện). Thêm một lỗi ở bảng đối chiếu ngày: tổng tiền bị cộng nhiều lần khi giao dịch có nhiều lần giao (đã xác nhận trên dữ liệu thật). Luật đúng đã chốt (PREMISE G3): hoàn tất ghi ở cấp giao dịch, lô chỉ sang "đã giao xong" khi mọi giao dịch của lô đã hoàn tất. Còn cần quyết định thời điểm sửa code trước khi làm màn chốt hoàn tất.
