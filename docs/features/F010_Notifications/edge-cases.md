# Edge Cases — F010_Notifications

Mã P0050–P0054, P0066, P0067 là đề xuất của bộ thiết kế, chưa cài; P0001, P0003, P0005, P0006, P0009 đã cài. Mọi tình huống là [ĐẶC TẢ — chưa xây]. Nguồn: [SCR031](../../screens/SCR031_NotificationInbox/spec.md) · [SCR032](../../screens/SCR032_NotificationSettings/spec.md) · [SCR033](../../screens/SCR033_OpsAlertQueue/spec.md) · [SCR035](../../screens/SCR035_SyncQueue/spec.md) mục 4, 9. Cột "Trạng thái": cả bốn màn chỉ đặc tả.

| Scenario | What Happens | User-Facing Message | Trạng thái |
|----------|--------------|---------------------|------------|
| Đánh dấu đã đọc thông báo của người khác hoặc không tồn tại (P0050) | Cùng một lỗi cho cả hai để mã không thành kênh dò sự tồn tại; trạng thái đọc trên giao diện quay lại như cũ | "Không tìm thấy thông báo này." | Chỉ đặc tả |
| Lọc từ ngày sau đến ngày hoặc khoảng hơn 92 ngày (đề xuất) | Chặn lọc, giữ giá trị đã nhập | "Ngày bắt đầu phải trước hoặc bằng ngày kết thúc." / "Khoảng lọc tối đa 92 ngày." | Chỉ đặc tả |
| Bộ lọc không khớp | Không phải lỗi; có nút xoá lọc | "Không có thông báo nào khớp bộ lọc." | Chỉ đặc tả |
| Chọn gửi gộp cho mức critical | Chặn vì không giữ nổi 5 phút | "Mức quan trọng phải gửi ngay — cam kết gửi trong 5 phút không áp dụng cho gửi gộp." | Chỉ đặc tả |
| Ngưỡng ngoài khoảng (P0051) | Từ chối, giữ dữ liệu đã gõ | "Ngưỡng của「{event_type}」phải trong khoảng {min}–{max} {unit}." | Chỉ đặc tả |
| Bật loại sự kiện mà không có người nhận (P0052) | Từ chối | "Loại「{event_type}」đang bật nhưng chưa có người nhận nào." | Chỉ đặc tả |
| Hai người cùng sửa một cấu hình (P0053) | Không ghi gì, người dùng tải lại rồi sửa tiếp | "Cấu hình đã được「{updated_by}」sửa lúc {updated_at}. Tải lại để xem bản mới nhất trước khi lưu." | Chỉ đặc tả |
| Tắt một loại sự kiện | Hộp thoại xác nhận; hàng chờ hiện tại vẫn gửi theo cấu hình cũ | "Từ lúc lưu, hệ thống ngừng sinh thông báo loại này; thông báo đã sinh và đang chờ gửi không bị ảnh hưởng." | Chỉ đặc tả |
| Đổi chức danh người dùng ở màn tài khoản | Người nhận theo chức danh đổi ngay, không có bước xác nhận (rủi ro đã biết) | None — silent handling | Chỉ đặc tả |
| Thử lại khi cảnh báo đã xử lý, hộp thư không tồn tại hoặc đang có lần thử chạy dở (P0054) | Không sinh lần thử; ba lý do ba câu khác nhau | "Không thể gửi lại: {reason}." | Chỉ đặc tả |
| Chức danh chỉ xem gọi thẳng RPC ghi (P0005) | Từ chối ở RPC | "Chức danh của bạn không được phép thực hiện thao tác này." | Chỉ đặc tả |
| Đóng cảnh báo với lý do dưới 10 ký tự | Chặn, giữ ghi chú đã gõ | "Lý do đóng phải từ 10 ký tự." | Chỉ đặc tả |
| Hàng cảnh báo rỗng | Tông trung tính, không báo thành công, vì job sinh cảnh báo chết cũng cho màn rỗng | "Không có cảnh báo nào đang mở." | Chỉ đặc tả |
| Email hỏng và cảnh báo về email hỏng cũng bằng email | Cần đường ngoài băng; chưa chốt | Không áp dụng — câu hỏi thiết kế còn mở | Chỉ đặc tả |
| Mất mạng, bản ghi vào hàng chờ | Không tăng số lần thử; bản ghi an toàn trên máy | "Chưa gửi được — vẫn mất kết nối. Bản ghi vẫn an toàn trên máy này." | Chỉ đặc tả |
| Cùng mã bản ghi tạm nhưng nội dung khác (P0066) | Không ghi đè, dòng `rejected` | "Mã bản ghi tạm này đã được nhận lúc {stored_at} với nội dung khác ({diff_fields}). Không ghi đè." | Chỉ đặc tả |
| Nội dung gần giống một bản đã vào sổ (đề xuất) | Dừng, chờ người quyết; không tự bỏ | "Có thể trùng với {mã bản ghi} đã vào sổ lúc {time}… Bạn kiểm tra rồi quyết định." | Chỉ đặc tả |
| Đồng bộ về ngày đã chốt (P0067) | `needs_correction`, bản ghi ở lại hàng chờ, mở phiếu F007 | "Ngày nghiệp vụ {business_date} đã chốt sổ lúc {closed_at}. Bản ghi này phải đi qua phiếu điều chỉnh có duyệt — không chèn thẳng vào ngày đã chốt được." | Chỉ đặc tả |
| P0009 xuất hiện ở màn hàng chờ | Nghĩa là P0067 đã bị lọt; là lỗi cần sửa, không phải trạng thái bình thường | Dùng chung câu của P0067 | Chỉ đặc tả |
| Người nhận hết hạn tham gia lúc đồng bộ (P0003) | `rejected`, không vào sổ | "Người nhận {buyer_name} đã hết hạn tham gia từ {expired_on}. Bản ghi không vào sổ được." | Chỉ đặc tả |
| Lần giao vượt số còn lại sau khi mất mạng (P0006) | `rejected`, gợi ý ghi ngoại lệ | "Giao dịch {trade_code} chỉ còn {remaining} {unit}, không đủ cho {requested} {unit}." | Chỉ đặc tả |
| Bỏ bản ghi với lý do dưới 10 ký tự | Chặn | "Vui lòng nêu lý do bỏ bản ghi, tối thiểu 10 ký tự." | Chỉ đặc tả |
| Ca sau dùng máy, hàng chờ của ca trước còn kẹt | Mỗi mục mang tên người đã nhập; vết ghi theo người đó | None — hiển thị khác nền cho dòng của ca trước | Chỉ đặc tả |
| Máy hỏng hoặc xoá dữ liệu trình duyệt khi còn bản chờ | Mất bản ghi chưa gửi; xử lý bằng quy trình vận hành (kiểm hàng chờ trống trước hết ca) | "Kiểm tra lại kết nối trước khi hết ca" (khi bản chờ lâu từ 2 giờ) | Chỉ đặc tả |
