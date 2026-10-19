# Edge Cases — F001_AdminAndPermissions

Nguồn: [SCR002](../../screens/SCR002_Login/spec.md) · [SCR003](../../screens/SCR003_AccountList/spec.md) · [SCR004](../../screens/SCR004_AccountDetail/spec.md) mục 4, 5, 9 và `lib/supabase/proxy.ts`. Cột "Trạng thái" phân biệt hành vi **đã có ở LAB-3** với **đặc tả chưa xây**.

| Scenario | What Happens | User-Facing Message | Trạng thái |
|----------|--------------|---------------------|------------|
| Gửi form đăng nhập khi ô email hoặc mật khẩu rỗng | Server Action từ chối trước khi gọi dịch vụ xác thực; hai ô giữ nguyên | "Vui lòng nhập đầy đủ email và mật khẩu." | Đã xây |
| Sai email, sai mật khẩu, hoặc email không tồn tại | Cả ba trường hợp trả **cùng một** thông báo, không tiết lộ email có tồn tại hay không | "Email hoặc mật khẩu không đúng." | Đã xây |
| Chưa đăng nhập mà mở đường dẫn khác `/login` | Lớp chặn route chuyển hướng về `/login` | Không có (chuyển hướng) | Đã xây |
| Đã đăng nhập mà mở lại `/login` | Cho qua thẳng, không đá về trang chủ — dễ bị tưởng là lỗi | Form đăng nhập hiện lại | Đã xây (chưa quyết có nên đổi — khách chưa nêu yêu cầu) |
| Chức danh sai mở route của chức danh khác (ví dụ `receiving` mở `/accounts`) | Chuyển sang `/403` ở lớp chặn route; nếu lọt qua lớp này thì Server Action và RPC vẫn từ chối `P0005` | "Chức danh của bạn không được phép thực hiện thao tác này." | Đã xây (ba lớp) |
| Tài khoản bị chuyển `suspended` khi đang có phiên | Request kế tiếp: lớp chặn route đăng xuất và chuyển về `/login?reason=suspended` | "Tài khoản đã bị vô hiệu hoá. Liên hệ quản trị hệ thống." | Đã xây |
| Tài khoản có phiên nhưng **thiếu dòng trong bảng `app_users`** | Cũng bị đăng xuất và nhận thông báo "bị vô hiệu hoá" — **thông báo gây hiểu lầm**, thực chất là thiếu dữ liệu | "Tài khoản đã bị vô hiệu hoá. Liên hệ quản trị hệ thống." | Đã xây (lỗi diễn đạt, ghi nhận) |
| Quản trị viên tự đổi quyền của chính mình | RPC `change_user_role` từ chối `P0007`; Server Action **không** tự kiểm điều này, toàn bộ chặn ở cơ sở dữ liệu | "Không thể tự đổi quyền của chính mình." | Đã xây |
| Gửi `newRole` ngoài 4 giá trị hợp lệ (DOM bị can thiệp) | Server Action từ chối với mã `generic`, không chạm RPC | "Có lỗi xảy ra, vui lòng thử lại." | Đã xây |
| Mở `/accounts/[id]` với `id` không tồn tại | `notFound()` → trang 404 mặc định của Next.js | Trang 404 mặc định | Đã xây |
| Bộ lọc `role`/`status` có giá trị lạ trên URL | Bỏ qua lặng lẽ, coi như không lọc | Không có | Đã xây |
| Danh sách rỗng sau khi lọc | Hiện trạng thái rỗng | "Không có tài khoản nào khớp bộ lọc." | Đã xây |
| Mất mạng đúng lúc gửi đăng nhập hoặc đổi quyền | **Chưa được định nghĩa** — không có `try/catch` quanh lời gọi, không có `error.tsx` | Chưa có | Chỉ đặc tả (khoảng trống, hỏi lại ở [technical-spec](./technical-spec.md#unresolved-questions)) |
| Đăng nhập sai nhiều lần liên tiếp | Không có khoá tạm; chỉ dựa vào mặc định của dịch vụ xác thực. NFR-SEC-03 đòi tạm khoá | Chưa có | Chỉ đặc tả (đề xuất, rủi ro: thiếu thì không đạt NFR-SEC-03) |
| Quản trị viên cần khoá/mở khoá tài khoản | Chưa có giao diện; thao tác tay trên cơ sở dữ liệu. Màn chi tiết chỉ đổi được chức danh | Chưa có | Chỉ đặc tả |
| Người dùng cần xác thực hai lớp (quản trị, người duyệt) | Không có — chỉ email + mật khẩu (MOCK-1 của LAB-3; tài liệu gốc trong repo, không nộp: `lab3-scope-lock.md`) | Chưa có | Chỉ đặc tả |
