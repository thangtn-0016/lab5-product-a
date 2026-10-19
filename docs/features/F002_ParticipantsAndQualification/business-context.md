# Business Context — F002_ParticipantsAndQualification

## Why It Matters

Ai được phép mua bán tại chợ là câu hỏi mà chợ phải trả lời **đúng tại thời điểm chốt**. Hiện nay hồ sơ giấy phép và chấp thuận nằm rời khỏi điểm giao dịch, nên một người đã hết hạn vẫn giao dịch được — rủi ro chính khách hàng tự ghi nhận, vì chợ về sau phải xử lý tranh chấp. Tính năng này lập hồ sơ người tham gia theo **bốn căn cứ tham gia khác nhau**, quản lý trạng thái hiệu lực, và để hệ thống tự kiểm tư cách mỗi lần chốt giao dịch hay giao hàng.

Điểm khách hàng nhấn mạnh nhất: giấy phép (許可) và chấp thuận (承認) là hai căn cứ **khác bản chất**, không được gộp thành một quy tắc chung; gộp là điều kiện bị loại trực tiếp (RFP 12-06). Phần đã chạy ở LAB-3 mới làm được việc *kiểm hết hạn*; việc tách hai luồng xử lý vi phạm còn là thiết kế (xem câu hỏi chưa giải quyết).

Tính năng này là một phần của [vòng đời tư cách tham gia](../../flows/FL-03-participant-qualification-lifecycle.md).

## Who Uses It

- **Quản lý người tham gia** (chức danh đề xuất `party_admin`, chưa có trong hệ thống đang chạy) — lập hồ sơ, đề nghị đổi trạng thái tư cách. Là người lập phiếu, không tự duyệt phiếu của mình.
- **Người có thẩm quyền duyệt tư cách** (quản lý người tham gia có cờ `can_approve` — đã chốt, PREMISE C-09) — duyệt hoặc từ chối phiếu tạm dừng, xét lại. Phải khác người lập phiếu.
- **Nhân viên đối chiếu** (`reconcile`) — tra danh sách để giải thích số liệu; chỉ xem.
- **Nhân viên giao dịch** (`trading`) — tra tư cách người mua *trước* phiên (đề xuất), để khỏi bị từ chối giữa lúc chốt; chỉ xem.
- **Nhân viên giao dịch và giao hàng** gặp tính năng này gián tiếp: mỗi lần chốt giao dịch, chốt kết quả đấu giá hoặc ghi lần giao, hệ thống tự kiểm tư cách người mua và từ chối nếu hết hiệu lực.

## What They Do

1. Quản lý người tham gia chọn loại đối tác (bán buôn, trung gian, mua bán được chấp thuận, người mua tại chợ) — hệ thống suy ra căn cứ tham gia tương ứng và dựng đúng mẫu hồ sơ của căn cứ đó.
2. Quản lý nhập số hiệu căn cứ, cơ quan cấp, ngày hiệu lực, ngày hết hạn — hồ sơ ở trạng thái "có hiệu lực".
3. Khi gia hạn, quản lý lập **dòng tư cách mới** thay vì sửa đè dòng cũ — để báo cáo kỳ cũ vẫn tra được tư cách tại thời điểm đó.
4. Khi phát hiện vi phạm, quản lý lập phiếu đề nghị chuyển trạng thái: giấy phép sang "hạn chế có điều kiện" (vẫn giao dịch được trong giới hạn), chấp thuận sang "phải xét lại" (dừng giao dịch tới khi duyệt lại).
5. Người duyệt khác người lập mở phiếu, quyết định duyệt hay từ chối — trạng thái tư cách chỉ đổi sau khi có quyết định, và đổi từ ngày hiệu lực ghi trên phiếu.
6. Nhân viên giao dịch bấm chốt cho một người mua đã hết hạn — hệ thống từ chối, nêu **hết hạn từ ngày nào, thuộc căn cứ nào, liên hệ phòng nào**, và không tạo bản ghi giao dịch nào.
7. Quản lý theo dõi danh sách hồ sơ sắp hết hạn với ngưỡng cảnh báo cấu hình được.

## Unresolved Questions

- **"Hạn chế có điều kiện" cụ thể hạn chế gì?** Trần số lượng, trần giá trị, cấm đấu giá, hay do chợ tự định? Tài liệu khách chỉ có cụm này. Chưa trả lời thì chưa có điều kiện nào để hệ thống kiểm. Đây là câu hỏi ưu tiên cao nhất.
- **Ai duyệt "phải xét lại", trong bao lâu?** Chưa rõ thuộc chức danh quản trị quy chế hay một vai riêng.
- **Căn cứ đăng ký chợ và đăng ký tại điểm giao dịch có hạn hiệu lực không?** Bảng của khách chỉ cho số đăng ký, nhưng cùng tài liệu lại yêu cầu hiển thị ngày hiệu lực, ngày hết hạn.
- **Hai căn cứ còn lại có luồng vi phạm riêng không**, hay hoàn toàn theo quy chế chợ?
- **Một người tham gia có giữ đồng thời hai căn cứ không?** Ảnh hưởng cách hệ thống chọn căn cứ khi kiểm tư cách.
- **Khi điều kiện hạn chế hết hạn** thì tự về trạng thái có hiệu lực hay tự thu hồi? Hai cách đọc hợp lý và hệ quả ngược nhau.
- **Người tham gia có bao giờ bị xoá** hay chỉ thu hồi? Ảnh hưởng báo cáo kỳ cũ.
