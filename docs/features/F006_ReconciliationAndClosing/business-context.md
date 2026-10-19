# Business Context — F006_ReconciliationAndClosing

## Why It Matters

Cuối mỗi ngày nghiệp vụ, bộ phận đối chiếu phải biến dữ liệu rải rác thành **số chính thức**. Hôm nay họ nhập lại từ năm nguồn khác nhau, sai lệch do nhập tay khoảng 1,8% số dòng, và khung 06:30–10:00 của họ bị nhập liệu nuốt hết. Tính năng này tự gom giao dịch, giao hàng và ngoại lệ của một ngày vào một bảng đối chiếu để họ chuyển từ *nhập liệu* sang *kiểm tra*, rồi **chốt sổ** để số liệu thành chính thức.

Sau khi chốt, **không ai được sửa thẳng** — mọi lần thử đều bị chặn và được ghi lại, vì chính nhật ký đó là nguồn của một trong mười hai báo cáo. Hôm nay việc sửa được làm bằng ghi đè trên bảng tính, nên chỉ khoảng 35% giao dịch có đủ dấu vết; mục tiêu của khách là từ 98% trở lên. Cũng ở đây, khách muốn quản lý **tranh chấp** (giao thiếu, giao thừa, trả hàng…) có người xử lý, hạn xử lý và kết quả cuối, và quản lý **lịch nghỉ chợ** để chuỗi ngày nghiệp vụ không có ngày nghỉ.

Một quyết định cần nói trước với người dùng: hộp thoại chốt sổ chỉ **cảnh báo** khi còn giao dịch chưa giao xong, không chặn. Giao hàng kéo dài qua ngày là chuyện bình thường; nếu chặn cứng thì không ngày nào chốt được. Hệ quả là số liệu chốt có thể chứa giao dịch chưa giao xong, vì sau chốt vẫn còn đường điều chỉnh có kiểm soát. Đã chốt là cảnh báo (PREMISE C-07); khách vẫn cần xác nhận cách đọc này.

Tính năng này là bước bốn của [chuỗi lô → giao dịch → giao hàng → đối chiếu](../../flows/FL-01-lot-trade-delivery-reconciliation.md) và là nơi dẫn sang [điều chỉnh sau chốt](../../flows/FL-02-post-close-correction.md).

## Who Uses It

- **Nhân viên đối chiếu** (`reconcile`) — xem bảng đối chiếu, chốt sổ, cập nhật tranh chấp. Làm sau 10:00 giờ Nhật. Hiện chỉ có một tài khoản đối chiếu trong dữ liệu mẫu.
- **Quản trị quy chế** (role đề xuất `rule_admin`, đã chốt ở PREMISE C-09; chưa có trong DB) — nhập và quản lý lịch nghỉ chợ.
- **Nhân viên giao dịch** (`trading`) và **nhân viên giao hàng** (gộp vào `receiving`, PREMISE C-10) — xem tranh chấp liên quan giao dịch của mình (đề xuất); chỉ xem.
- **Quản trị vận hành** (đề xuất) — đọc lịch để biết cửa sổ chạy batch.

## What They Do

1. Nhân viên đối chiếu mở bảng đối chiếu — mỗi dòng là một ngày nghiệp vụ với số giao dịch, tổng tiền, số lần giao, số ca ngoại lệ; hệ thống đã gom sẵn, không có ô nào để nhập.
2. Với ngày còn mở, nhân viên bấm "Chốt sổ" — hộp thoại liệt kê các giao dịch **chưa giao xong** cùng số còn lại, kèm dòng nhắc rằng hệ thống vẫn cho chốt.
3. Nhân viên xác nhận — ngày thành "đã chốt", ghi nhận người chốt và thời điểm, và ngày đó **không còn sửa trực tiếp được**.
4. Ai đó (lỡ tay hoặc cố ý) thử ghi vào ngày đã chốt — hệ thống từ chối, ghi lại lần thử vào nhật ký, và (đặc tả) dẫn người đó sang đường xin điều chỉnh.
5. Nhân viên đối chiếu mở danh sách tranh chấp đang mở, lọc theo trạng thái, người xử lý, quá hạn (đặc tả).
6. Nhân viên nhận xử lý một tranh chấp, ghi nguyên nhân, đặt ngày dự kiến, cập nhật qua các bước, và ghi **kết quả cuối** — kết quả chỉ là *kết luận*, không tự sửa số liệu; nếu phải đổi số của ngày đã chốt thì lập phiếu điều chỉnh (đặc tả).
7. Quản trị quy chế nhập ngày nghỉ chợ — hệ thống bỏ qua các ngày đó khi sinh chuỗi ngày nghiệp vụ; ngày đã có dữ liệu hoặc đã chốt không đặt thành ngày nghỉ được (đặc tả).

## Unresolved Questions

- **Chốt sổ cảnh báo hay chặn?** Đã chốt cảnh báo (PREMISE C-07). "Điều kiện phải đạt" trong yêu cầu của khách có thể đọc là chặn; chọn cảnh báo vì giao hàng kéo dài qua ngày; nếu khách muốn chặn thì phải thêm luồng ghi đè kèm lý do.
- **Tranh chấp đang mở có chặn chốt sổ không?** Quyết cùng câu trên.
- **Hạn xử lý tranh chấp (SLA) là bao nhiêu ngày?** Báo cáo tranh chấp nhắc "SLA xử lý" nhưng khách chưa cho số; mặc định 3 ngày nghiệp vụ là đề xuất.
- **Ai tạo tranh chấp, từ màn nào?** Tự động sinh từ ngoại lệ giao hàng, hay do người lập? Hai hướng cho hai thiết kế rất khác.
- **Năm trạng thái tranh chấp có đúng nghiệp vụ không?** Khách chưa liệt kê trạng thái nào; có thể chỉ cần "đang mở" và "đã đóng".
- **Ai được đóng vụ — người xử lý hay cấp trên?**
- **Lịch nghỉ chợ nhập tay hay đồng bộ từ nguồn ngoài?** Đã chốt nhập tay (`calendar_mode=manual`, PREMISE C-08); khách chưa xác nhận giả định. Hướng đồng bộ phát sinh hạng mục tích hợp chưa có trong ước lượng.
- **Hệ thống phục vụ một chợ hay nhiều chợ?** Nhiều chợ thì mỗi chợ một lịch riêng.
- **Hạn thanh toán đếm theo ngày lịch hay ngày nghiệp vụ?** Lịch nghỉ làm hai cách đếm cho ra hai ngày khác nhau, và nó chạm vào tiền khuyến khích.
- **Chứng cứ tranh chấp lưu 3 năm rồi chuyển lưu trữ nguội**, mất tối đa 2 ngày làm việc để phục hồi; cần nói trước với người dùng.
