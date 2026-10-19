# Edge Cases — F009_ReportsAndExport

Mã lỗi P0040–P0049 là đề xuất của bộ thiết kế, chưa cài; P0005 đã cài (sai chức danh). Mọi tình huống dưới đây là [ĐẶC TẢ — chưa xây]. Nguồn: [SCR028](../../screens/SCR028_ReportCatalog/spec.md) · [SCR029](../../screens/SCR029_ReportViewExport/spec.md) · [SCR030](../../screens/SCR030_AccountingBatch/spec.md) mục 4, 9. Cột "Trạng thái": cả ba màn chỉ đặc tả.

| Scenario | What Happens | User-Facing Message | Trạng thái |
|----------|--------------|---------------------|------------|
| Gõ thẳng mã báo cáo không thuộc 12 mã (P0040) | Chuyển về danh mục, banner đỏ | "Mã báo cáo {report_code} không có trong danh mục. Danh mục gồm đúng 12 báo cáo cố định." | Chỉ đặc tả |
| Chức danh không có quyền xem báo cáo (P0044) | Chặn ở RPC (route chỉ chặn theo tiền tố chung) | "Chức danh của bạn không được xem báo cáo {report_code}." | Chỉ đặc tả |
| Danh mục rỗng với một chức danh | Hành vi đúng của phân quyền theo báo cáo; không fallback hiện tất cả | "Không có báo cáo nào khớp bộ lọc." | Chỉ đặc tả |
| Tham số lạ hoặc sai định dạng (P0041) | Từ chối; tham số đã nhập giữ nguyên | "Tham số {param} không dùng được cho báo cáo {report_code}." | Chỉ đặc tả |
| Báo cáo chỉ-sau-chốt trên ngày chưa chốt (P0042) | Dừng, không chạy | "Ngày nghiệp vụ {business_date} chưa chốt sổ. Báo cáo {report_code} chỉ chạy được sau khi chốt." | Chỉ đặc tả |
| Thiếu `as_of` hoặc SQL báo cáo tham chiếu giờ hiện tại (P0043) | Chặn để không ra số sai; là mã phòng thủ | "Báo cáo thiếu thời điểm căn cứ quy tắc — không chạy để tránh ra số sai." | Chỉ đặc tả |
| Báo cáo tra cứu vận hành trên ngày chưa chốt | Chạy, gắn cờ tạm; CSV có hậu tố `-provisional` | "Số liệu tạm — ngày chưa chốt" | Chỉ đặc tả |
| Chạy rỗng (0 dòng) | Vẫn ghi lần chạy và hiện mã lần chạy | "Không có dòng nào khớp tham số." | Chỉ đặc tả |
| Xuất CSV quá 50.000 dòng (P0049) | Không chặn; chuyển hàng đợi nền, theo dõi ở màn batch | "Kết quả {row_count} dòng vượt ngưỡng xuất trực tiếp. Đã chuyển sang chạy nền — theo dõi ở màn Trạng thái batch." | Chỉ đặc tả |
| In lại lần chạy cũ sau khi đổi quy chế | Dùng `params` và `as_of` đã lưu, không dùng tham số đang hiển thị; checksum phải trùng | None — xử lý im lặng, số giữ nguyên | Chỉ đặc tả |
| Tạo lô kế toán khi ngày chưa chốt (P0045) | Từ chối | "Ngày nghiệp vụ {business_date} chưa chốt sổ — chưa tạo được lô xuất kế toán." | Chỉ đặc tả |
| Tạo lô `full` lần hai cho cùng ngày đã gửi (P0046) | Từ chối; nút mở phiếu điều chỉnh | "Ngày {business_date} đã gửi lô {batch_code}. Thay đổi sau khi gửi phải đi qua phiếu điều chỉnh (F07) rồi xuất lô delta." | Chỉ đặc tả |
| Bấm "Chạy lại" khi batch không ở trạng thái thất bại (P0047) | Từ chối | "Chỉ chạy lại được batch đang ở trạng thái thất bại." | Chỉ đặc tả |
| Batch thử 3 lần vẫn lỗi (P0048) | Tắt nút, đẩy cảnh báo sang hàng vận hành | "Batch đã thử 3 lần không thành công. Cần người vận hành xử lý trước khi chạy lại." | Chỉ đặc tả |
| Batch quá 11:00 chưa xong | Hiện `late`, đẩy cảnh báo; không tự huỷ batch đang chạy | "Quá hạn {n} phút" | Chỉ đặc tả |
| `sys_admin` gọi tạo lô hoặc chạy lại (P0005) | Từ chối ở RPC | "Chức danh của bạn chỉ được xem màn này." | Chỉ đặc tả |
| Mất mạng khi gửi lệnh | Trạng thái batch trên màn có thể đã cũ | "Mất kết nối — chưa gửi được lệnh. Trạng thái batch trên màn có thể đã cũ, tải lại trước khi thử tiếp." | Chỉ đặc tả |
| Hai người cùng bấm tạo lô cho một ngày | Khoá idempotency theo `batch_code`; người sau nhận P0046 (suy luận) | "Ngày {business_date} đã gửi lô {batch_code}…" | Chỉ đặc tả |
| Đổi sang API hai chiều sau này (C-03 chốt CSV một chiều; khách chưa xác nhận Q-04) | Thiết kế SCR030 phải làm lại, không chỉ sửa thêm | Không áp dụng — quyết định dự án, không phải thông báo người dùng | Chỉ đặc tả |
