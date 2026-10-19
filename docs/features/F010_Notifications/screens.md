# Screens — F010_Notifications

Chỉ trang chủ có trong prototype LAB-3 (và chưa có thông báo); bốn màn còn lại là [ĐẶC TẢ — chưa xây]. Nguồn: (tài liệu gốc trong repo, không nộp) `sc-01`, `sc-31`, `sc-32`, `sc-33`, `sc-35`. Xem [technical-spec.md](./technical-spec.md).

## Screen List

| Screen Name | SCR### | What User Sees | What User Can Do |
|-------------|--------|----------------|------------------|
| Trang chủ theo chức danh (ホーム, alias SC-01) — [ĐÃ TRIỂN KHAI] phần số liệu | [SCR001_RoleHome](../../screens/SCR001_RoleHome/spec.md) | Lời chào và các thẻ số liệu theo 4 chức danh. Hiện chưa có số thông báo chưa đọc; số chưa đọc và số bản chờ đồng bộ là (đề xuất, rủi ro Thấp) | Chỉ xem; điều hướng qua menu |
| Hộp thông báo (通知ボックス, alias SC-31) | [SCR031_NotificationInbox](../../screens/SCR031_NotificationInbox/spec.md) | Thông báo của chính mình: loại, mức, tiêu đề, nội dung, đối tượng liên quan, ngày nghiệp vụ, trạng thái đã đọc và trạng thái gửi email; mặc định chỉ chưa đọc, critical lên trước | Lọc theo loại và khoảng ngày; bấm dòng để đi tới đối tượng (đánh dấu đã đọc cùng lúc); đánh dấu một dòng hoặc tất cả đã đọc (có xác nhận) |
| Cấu hình thông báo (通知設定, alias SC-32) | [SCR032_NotificationSettings](../../screens/SCR032_NotificationSettings/spec.md) | 4 thẻ, mỗi loại sự kiện: bật/tắt, mức, người nhận, kênh, ngưỡng, thời điểm gửi, số thông báo đang chờ, người và lúc sửa cuối | Chỉ `ops_admin` sửa và lưu; chức danh khác chỉ xem. Tắt một loại cần xác nhận |
| Hàng cảnh báo vận hành (運用警告キュー, alias SC-33) | [SCR033_OpsAlertQueue](../../screens/SCR033_OpsAlertQueue/spec.md) | Cảnh báo mở và đã nhận, cũ nhất lên trước; mỗi dòng có nguồn, số lần thử, lỗi cuối và bảng kết quả từng lần | Nhận xử lý, thử lại thủ công (chỉ cho email), đóng kèm lý do; với batch/lô kế toán thì mở màn batch |
| Hàng chờ đồng bộ (同期待ちキュー, alias SC-35) | [SCR035_SyncQueue](../../screens/SCR035_SyncQueue/spec.md) | Bản ghi nhập lúc mất mạng chưa lên máy chủ, trạng thái, người nhập, lỗi gần nhất; thanh trạng thái kết nối và số bản chờ | Gửi lại, sửa, bỏ kèm lý do, lập phiếu điều chỉnh, chọn "đây là bản trùng" hoặc "không trùng — gửi tiếp" |

Chức danh: hộp thông báo cho mọi chức danh; cấu hình và cảnh báo cho `ops_admin` (đề xuất), `reconcile`, `sys_admin` (chỉ xem); hàng chờ đồng bộ cho `receiving`. Route `/notifications/settings` phải đặt trước `/notifications` trong bảng chặn route.

## User Journey

1. Người dùng đăng nhập và thấy trang chủ; khi tính năng có, số thông báo chưa đọc hiện cạnh tên.
2. Người dùng mở hộp thông báo, thấy các việc critical trước, bấm một dòng để tới đúng đối tượng.
3. Người vận hành vào hàng cảnh báo, mở rộng một dòng để xem từng lần thử, nhận xử lý rồi thử lại hoặc đóng.
4. Nếu thông báo nhiều hoặc ít quá, người vận hành sang màn cấu hình đổi ngưỡng; banner xác nhận cho biết các thông báo đang chờ không bị ảnh hưởng.
5. Ở hiện trường, khi mất mạng, nhân viên vẫn lưu được; màn hàng chờ cho biết từng bản đang chờ, bị từ chối, nghi trùng hay cần phiếu điều chỉnh.

```mermaid
journey
    title Thông báo và hàng chờ đồng bộ
    section Người nhận
      Thấy số chưa đọc: 4: Nhân viên
      Mở thông báo và đi tới việc: 5: Nhân viên
    section Vận hành
      Xem hàng cảnh báo: 3: Vận hành
      Thử lại hoặc đóng cảnh báo: 4: Vận hành
      Chỉnh cấu hình: 4: Vận hành
    section Hiện trường
      Lưu khi mất mạng: 5: Tiếp nhận
      Xử lý bản trùng hoặc ngày đã chốt: 3: Tiếp nhận
