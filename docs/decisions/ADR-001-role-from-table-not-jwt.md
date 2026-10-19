# ADR-001 — Đọc chức danh từ bảng `app_users`, không nhét vào JWT

| | |
|---|---|
| **Trạng thái** | Đã triển khai |
| **Ngày ghi** | 2026-10-04 (viết sau khi prototype LAB-3 chạy thật) |
| **Phạm vi** | Xác thực/phân quyền — `proxy.ts`, RLS, mọi RPC |
| **Liên quan** | [architecture.md §C1, §5](../system/architecture.md) · [permissions.md](../system/permissions.md) · [SCR004_AccountDetail](../screens/SCR004_AccountDetail/spec.md) · [F001_AdminAndPermissions](../features/F001_AdminAndPermissions/technical-spec.md) |

> Mỗi ADR đủ bốn phần: bối cảnh · lựa chọn · phương án đã bỏ · hệ quả (kể cả hệ quả xấu). ADR của bộ này viết **sau** khi prototype chạy thật, nên phần hệ quả phần lớn là quan sát được, không phải dự đoán.

## Bối cảnh

Mỗi request cần biết người dùng thuộc chức danh nào để quyết định cho vào route nào và cho đọc dòng nào. Supabase cung cấp **Custom Access Token Hook** — nhét chức danh thành claim trong JWT, đọc ra không tốn truy vấn. Gói Free dùng được.

Nhưng màn hình `SC-04` có chức năng đổi quyền, và `F01-04` của LAB-1 yêu cầu *"cấp, thu, tạm dừng, mở lại quyền — có lưu vết"*.

## Lựa chọn

Chức danh đọc trực tiếp từ bảng `app_users` ở mỗi request. Hàm `current_app_role()` (`SECURITY DEFINER`) là nguồn sự thật duy nhất, dùng chung cho cả `proxy.ts` lẫn mọi RLS policy.

**Bằng chứng triển khai:** `supabase/migrations/0002_core_identity.sql:43` (`current_app_role()`); `lib/auth/route-policy.ts` (`ROUTE_POLICY`, `canAccess`); `lib/supabase/proxy.ts` (kiểm `app_users.status`, `signOut()` khi `suspended`).

## Phương án đã bỏ và vì sao

**Custom Access Token Hook.** Bỏ vì JWT có thời hạn sống — sau khi quản trị viên đổi quyền ở `SC-04`, token cũ vẫn mang chức danh cũ cho tới khi hết hạn, khoảng một giờ.

Đó đúng là thứ `SC-04` phải chứng minh là làm thật. Một màn hình đổi quyền mà quyền không đổi ngay thì là màn hình trang trí.

## Hệ quả

**Được:** một nguồn sự thật, đổi quyền có hiệu lực tức thì, không có trạng thái hai nơi lệch nhau.

**Phải chịu:** thêm một round-trip database mỗi request đi qua proxy. Với tải của đề này — 60 người dùng đồng thời, 1.200 giao dịch ngày cao điểm — không đáng kể. Nhưng nếu tải tăng một bậc thì đây là chỗ đầu tiên phải xem lại, và lúc đó việc chuyển sang JWT claim sẽ kéo theo bài toán vô hiệu hoá token mà hiện chưa có lời giải.

**Rủi ro còn lại:** `current_app_role()` nằm trên đường đi của mọi truy vấn. Nó hỏng thì toàn hệ thống hỏng, không hỏng một phần.

## Câu hỏi còn mở

- Ngưỡng tải nào thì round-trip mỗi request thành vấn đề? Chưa đo, chỉ ước lượng từ con số 60 người dùng đồng thời của đề (RFP FIG-LOAD-01).
