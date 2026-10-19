# Business Context — F011_AuditLog

## Why It Matters

Hiện nay chỉ khoảng 35% giao dịch còn đủ dấu vết thay đổi, vì việc sửa được làm trực tiếp trên bảng tính; tra lại một bản ghi cũ mất 15–30 phút. Khách đặt mục tiêu nâng tỷ lệ có đủ dấu vết lên từ 98% trở lên và rút thời gian tra cứu xuống không quá 2 phút. Nhật ký kiểm toán là chỗ chứng minh "ai làm gì, lúc nào, số cũ, số mới, vì sao" khi cơ quan quản lý hoặc kiểm toán nội bộ hỏi.

Tính năng này được LAB-1 yêu cầu xây cùng lúc với nhóm quản trị và phân quyền, không gắn thêm sau: mọi nhóm còn lại đều ghi vết vào đây.

Feature này tham gia luồng điều chỉnh sau chốt ([FL-02](../../flows/FL-02-post-close-correction.md)) và luồng lô → giao dịch → giao hàng → đối chiếu ([FL-01](../../flows/FL-01-lot-trade-delivery-reconciliation.md)): lần thử sửa vào ngày đã chốt được ghi lại và là bằng chứng cho đường điều chỉnh có duyệt.

## Who Uses It

- **Kiểm toán nội bộ** (`internal_audit`, đề xuất — chưa có trong hệ thống) — điều tra một con số của tháng trước bằng tra cứu thay vì phỏng vấn người còn nhớ.
- **Bộ phận đối chiếu** (`reconcile`) — chứng minh mình làm đúng; xem được những lần thử sửa vào ngày đã chốt.
- **Quản trị hệ thống** (`sys_admin`) — hiện là chức danh duy nhất đọc được nhật ký trong bản prototype; xem để hỗ trợ, không sửa.
- **Ban quản lý** — hưởng lợi gián tiếp: trả lời được câu hỏi của cơ quan quản lý.

## What They Do

1. Hệ thống tự ghi một dòng nhật ký mỗi khi có thao tác quan trọng (tạo, sửa, duyệt, chốt sổ, đổi quyền) — không ai phải nhập tay.
2. Người dùng thử sửa vào ngày nghiệp vụ đã chốt — hệ thống chặn, và ghi lại chính lần thử đó như một sự kiện nghiệp vụ.
3. Kiểm toán viên mở màn tra cứu, chọn một hoặc nhiều chiều: mã giao dịch, ngày nghiệp vụ, người tham gia, loại thao tác.
4. Hệ thống kiểm tra khoảng tra cứu không quá rộng khi chưa có điều kiện thu hẹp; quá rộng thì yêu cầu thu hẹp.
5. Kiểm toán viên đọc từng dòng với đủ năm thông tin: ai làm, lúc nào, số cũ, số mới, lý do; mở rộng dòng để xem nguyên nội dung trước và sau.
6. Với bản ghi tiền khuyến khích hoặc đối chiếu, người xem thấy ngay phiên bản quy chế đã áp dụng và ngày hiệu lực.
7. Kiểm toán viên xuất kết quả ra CSV; chính việc xuất này cũng được ghi vết.

## Unresolved Questions

- **Vai trò kiểm toán nội bộ**: đã chốt có role riêng `internal_audit` (đề xuất, chưa có trong hệ thống — PREMISE C-09), không dùng cờ `can_audit`; dùng chung với bộ phận đối chiếu thì kiểm toán và đối chiếu không còn tách được. Khách chưa xác nhận.
- **Mức mở dữ liệu cho bộ phận đối chiếu**: được đọc toàn bộ nhật ký hay chỉ phần liên quan lô, giao dịch, giao hàng, ngày nghiệp vụ (giấu phần tài khoản và đổi quyền)?
- **Lý do thay đổi**: khách đòi đủ năm thông tin, trong đó có "lý do". Mọi thao tác ghi vết đều phải có lý do bắt buộc, hay chỉ một số loại?
- **Hạn lưu và xuất hàng loạt**: dữ liệu tra cứu giữ 7 năm; "tra nhanh cả 7 năm" hay "tra được nhưng có thể chậm sau 2 năm" (câu hỏi Q-10 gửi khách, chưa có trả lời). Xuất CSV hàng loạt có cần giới hạn hoặc duyệt, vì chứa thông tin cá nhân của người tham gia?
