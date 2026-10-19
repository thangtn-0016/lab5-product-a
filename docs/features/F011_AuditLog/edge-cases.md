# Edge Cases — F011_AuditLog

Nguồn: [SCR034](../../screens/SCR034_AuditLogSearch/spec.md) mục 4, 5, 9; `0002`, `0009`, `0040`, `lib/domain/reconciliation.ts:140-159`. Mã lỗi: `P0001`–`P0009` đã cài; `P0068` là đề xuất của bộ screen-spec, chưa cài. Cột "Trạng thái": phân biệt hành vi **đã có ở LAB-3** với **đặc tả chưa xây**.

| Scenario | What Happens | User-Facing Message | Trạng thái |
|----------|--------------|---------------------|------------|
| Tra theo mã giao dịch không tồn tại | Kết quả rỗng, không lỗi | "Không có vết nào khớp điều kiện tra cứu." | Chỉ đặc tả |
| Tra theo mã người tham gia không tồn tại | Kết quả rỗng, không lỗi | "Không có vết nào khớp điều kiện tra cứu." | Chỉ đặc tả |
| Ngày bắt đầu sau ngày kết thúc | Chặn ngay ở form | "Ngày bắt đầu phải trước ngày kết thúc." | Chỉ đặc tả |
| Khoảng > 31 ngày, chưa chọn giao dịch/người tham gia/người thực hiện | Từ chối `P0068` (đề xuất, chưa cài), không chạy truy vấn, giữ bộ lọc | "Khoảng tra cứu {days} ngày là quá rộng khi chưa chọn giao dịch, người tham gia hay người thực hiện. Thu hẹp còn {max_days} ngày hoặc thêm một điều kiện." | Chỉ đặc tả |
| Truy vấn lỗi mạng | Khối lỗi, điều kiện giữ trên đường dẫn | "Không tải được nhật ký kiểm toán. Kiểm tra kết nối rồi tra lại." | Chỉ đặc tả |
| `receiving`/`trading` mở `/audit` | Route chặn về `/403`; RLS trả 0 dòng | "Bạn không có quyền mở trang này." | Chỉ đặc tả |
| `reconcile` tra cứu khi RLS chưa nới | `audit_log` chỉ cho `sys_admin` đọc (`0009:28-29`) → rỗng im lặng | (hiện rỗng — rủi ro: tưởng không có vết; phải nới RLS trước khi làm giao diện) | Đã xây |
| Tài khoản người thực hiện không tra được tên (RLS `app_users`) | Hiện `actor_id` thô, không hiện "—" | (hiện mã người dùng) | Chỉ đặc tả |
| Dòng chốt sổ / lần thử bị chặn không có `record_id` | Không nối được tới giao dịch qua `record_id`; ngày nghiệp vụ lấy từ nội dung cũ | "—" ở cột giao dịch | Chỉ đặc tả |
| Cột "Lý do" khi `audit_log` chưa có `reason` | Luôn "—" | "—" (mục tiêu 98% dấu vết đủ không đạt cho tới khi thêm cột) | Chỉ đặc tả |
| Ghi log lần thử bị chặn thất bại | Ngoại lệ (lỗi mạng) → `catch` ghi `console.error` (`reconciliation.ts:156-158`); lỗi RPC trả trong `{ error }` thì không ai đọc nên mất vết im lặng (khoảng trống FR-CORR-03, xem [technical-spec](./technical-spec.md)). Không ném lại; lỗi gốc vẫn hiện | Thông báo lỗi gốc (ví dụ ngày đã chốt) — lần thử không có vết | Đã xây (có khoảng trống) |
| Gọi `log_blocked_write_attempt` khi chưa đăng nhập | RPC raise `P0005` `0040:32-35` | (không hiện cho người dùng; `{ error }` của lời gọi không được đọc nên lỗi bị bỏ qua) | Đã xây |
| Hai nguồn lệch tên hành động | Lọc theo nhãn thiết kế không khớp giá trị thật (`close_business_day` so với `close_day`) | Không có kết quả dù có dữ liệu — cần thống nhất trước LAB-5 | Chỉ đặc tả |
| Muốn sửa/xoá một dòng nhật ký | Không có đường ghi: không policy UPDATE/DELETE, không nút trên màn `0002:26,39` | "Nhật ký chỉ đọc." | Đã xây |
| Xuất CSV hàng loạt chứa dữ liệu cá nhân | Chưa chốt giới hạn/duyệt; việc xuất được ghi vết (đề xuất, rủi ro Trung bình) | (chưa có thông điệp) | Chỉ đặc tả |
