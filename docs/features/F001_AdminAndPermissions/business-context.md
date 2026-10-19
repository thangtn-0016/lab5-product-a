# Business Context — F001_AdminAndPermissions

## Why It Matters

Chợ đầu mối thủy sản chạy trên **terminal dùng chung theo ca**: người ca sau ngồi đúng cái máy ca trước vừa rời, lúc 02:00 sáng. Nếu ai cũng mở được mọi màn hình thì ranh giới trách nhiệm giữa người nhận hàng, người chốt giao dịch và người đối chiếu biến mất ở đúng chỗ cần nó nhất. Tính năng này cho mỗi nhân viên đúng một chức danh, và đảm bảo chức danh đó quyết định họ được mở màn nào, đọc dòng dữ liệu nào.

Khách hàng nêu rõ nguyên tắc **tách trách nhiệm** (責任分離, RFP 02-08) và yêu cầu mọi lần đổi quyền phải để lại dấu vết: quyền cũ, quyền mới, ai đổi, lúc nào. Chức danh quản trị hệ thống cố ý **không** mở được màn nghiệp vụ — để phân quyền là một hạn chế thật, không phải "quản trị viên thấy tất cả".

## Who Uses It

- **Quản trị hệ thống** (`sys_admin`) — xem danh sách tài khoản nội bộ, đổi chức danh cho nhân viên, xem lịch sử đổi quyền. Là chức danh duy nhất mở được hai màn quản trị tài khoản và không mở được bất kỳ màn nghiệp vụ nào.
- **Nhân viên tiếp nhận & giao hàng** (`receiving`), **nhân viên giao dịch** (`trading`), **nhân viên đối chiếu** (`reconcile`) — đăng nhập đầu ca, chỉ thấy phần việc của chức danh mình. Bị đẩy sang trang "không đủ quyền" nếu mở nhầm đường dẫn.
- **Năm chức danh nghiệp vụ mới** (đề xuất, đã chốt ở PREMISE C-09: quản lý người tham gia `party_admin`, quản trị quy chế `rule_admin`, quản trị vận hành `ops_admin`, quản lý chợ `market_manager`, kiểm toán nội bộ `internal_audit`) — chưa có trong hệ thống đang chạy; việc duyệt dùng cờ `can_approve` trên tài khoản. Nhãn "(đề xuất)" giữ vì chưa có trong DB.

## What They Do

1. Nhân viên mở hệ thống bằng bất kỳ đường dẫn nào — hệ thống đưa họ về trang đăng nhập nếu chưa có phiên làm việc.
2. Nhân viên nhập email và mật khẩu nội bộ — hệ thống xác thực, rồi đưa về trang chủ đúng chức danh.
3. Quản trị viên mở danh sách tài khoản, lọc theo chức danh hoặc trạng thái, chọn một người — màn chi tiết hiện chức danh hiện tại và lịch sử đổi quyền.
4. Quản trị viên chọn chức danh mới, ghi chú lý do (tuỳ chọn), xác nhận — quyền đổi **có hiệu lực ngay ở lần thao tác kế tiếp** của người đó, và một dòng lịch sử mới xuất hiện.
5. Quản trị viên cố tự đổi quyền của chính mình — hệ thống từ chối, vì tự khoá mình khỏi màn quản trị là sự cố không có đường cứu.
6. Tài khoản bị chuyển sang trạng thái ngưng — ở lần thao tác kế tiếp người đó bị đăng xuất và thấy thông báo tài khoản bị vô hiệu hoá.

## Unresolved Questions

- **Xác thực hai lớp**: hướng đã chốt (PREMISE C-04): MFA phân tầng — bắt buộc cho tài khoản quản trị và người duyệt nội bộ (NFR-SEC-01), người bên ngoài một yếu tố + gắn thiết bị. Chưa chốt thiết bị token (SEC-IAM-02) và ảnh hưởng lên terminal dùng chung. Prototype chỉ có email + mật khẩu.
- **Chức danh thứ 5–9 — đã chốt (PREMISE C-09):** thêm năm role đề xuất **và** cờ `can_approve` trên tài khoản. Khách chưa xác nhận giả định này; nếu đổi thì đổi cả ma trận quyền và nguyên tắc người lập khác người duyệt ở các tính năng sau (xem [F007](../F007_PostCloseCorrection/business-context.md)).
- **Khoá và mở khoá tài khoản**: tài liệu kiến trúc gọi đây là thao tác nhạy cảm, nhưng chưa có màn hình nào làm; hiện phải sửa tay trong cơ sở dữ liệu.
- **Chính sách phiên làm việc**: thời gian tự thoát khi không dùng, số lần sai mật khẩu trước khi tạm khoá — khách nêu yêu cầu, chưa có số cụ thể.
