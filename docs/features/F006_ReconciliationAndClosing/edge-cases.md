# Edge Cases — F006_ReconciliationAndClosing

Nguồn: [SCR018](../../screens/SCR018_DailyReconciliation/spec.md) · [SCR019](../../screens/SCR019_DisputeList/spec.md) · [SCR020](../../screens/SCR020_DisputeDetail/spec.md) · [SCR021](../../screens/SCR021_MarketCalendar/spec.md) mục 4, 5, 9, 10; `0008_rpc_functions_admin.sql:6-41`; `0010_closed_day_guard.sql`; kết quả LAB-3 ở (tài liệu gốc trong repo, không nộp) `verification-checklist.md` §5, §6. Mã `P0055`–`P0059` do SCR020/SCR021 đề xuất, chưa cài.

| Scenario | What Happens | User-Facing Message | Trạng thái |
|----------|--------------|---------------------|------------|
| Chốt sổ khi còn giao dịch chưa giao xong | Hộp thoại **cảnh báo**, nút xác nhận **không bị khoá**; RPC vẫn chốt và lưu danh sách vào nhật ký (ADR-005) | "Hệ thống vẫn cho chốt dù còn điều kiện chưa đạt…" | Đã xây (chủ ý, chờ khách xác nhận) |
| Chốt lần hai một ngày đã chốt | `P0002` | "Ngày nghiệp vụ {ngày} đã được chốt sổ trước đó, không thể chốt lại." | Đã xây |
| Ngày nghiệp vụ không tồn tại | `P0001` | "Không tìm thấy ngày nghiệp vụ này." | Đã xây |
| Chức danh khác `reconcile` chốt sổ | `P0005` ở Server Action rồi ở RPC | "Chức danh của bạn không được phép chốt sổ." | Đã xây |
| Mất mạng lúc chốt | Hộp thoại giữ nguyên, banner lỗi; không chốt (chốt sổ cần mạng) | "Mất kết nối mạng — không thể chốt sổ. Kiểm tra lại kết nối rồi thử lại." | Đã xây |
| Ghi lần giao cho ngày đã chốt | `P0002` ở RPC giao hàng; lần thử được ghi vào nhật ký | "Ngày nghiệp vụ {ngày} đã chốt sổ lúc {giờ}, không thể ghi thêm lần giao." | Đã xây (đã kiểm ở LAB-3) |
| Ghi trực tiếp vào bảng lô/giao dịch/giao hàng của ngày đã chốt (bỏ qua RPC) | Trigger từ chối `P0009`; **lần thử không được ghi vào nhật ký** vì transaction bị huỷ và người gọi không phải Server Action | "Ngày nghiệp vụ đã chốt sổ, không thể sửa" | Đã xây (nhật ký chỉ phủ đường ghi lần giao) |
| Thử sửa ngày đã chốt từ các màn khác (nhập lô, chốt giao dịch) | Bị chặn nhưng **không gọi** nhật ký lần thử; nhập lô còn rơi vào thông báo "lỗi kết nối" chung | Xem [F003](../F003_LotsAndInspection/edge-cases.md) | Đã xây (khoảng trống FR-CORR-03 "chặn **và ghi log**") |
| Ngày đã chốt mà người xem muốn sửa số | Không có đường nào ở SCR018 dẫn sang phiếu điều chỉnh | Không có | Đã xây (khoảng trống) — đặc tả dẫn sang [F007](../F007_PostCloseCorrection/edge-cases.md) |
| Người chốt hiển thị "—" dù đã có thời điểm chốt | RLS `app_users` chỉ cho tự xem mình + `sys_admin`; khi có nhiều tài khoản đối chiếu, người khác thấy "—" | "—" | Đã xây (rủi ro khi có nhiều tài khoản) |
| Tổng tiền của ngày có giao dịch giao nhiều lần | **(đã xác nhận trên DB thật 2026-10-06, lỗi B-05)** view cộng `amount` trên tích giao dịch × lần giao nên nhân theo số lần giao (2026-09-23: 53250.00 so với 38250.00 thật); số giao dịch dùng `count(distinct)` thì đúng | Không có (sai ngầm) | Đã xây — **lỗi số liệu** B-05: chưa sửa; có bản vá nháp 0042 chưa áp. Xem [technical-spec](./technical-spec.md#algorithms) |
| Bộ lọc ngày để trống | Hiện toàn bộ ngày; không phân trang | Không có | Đã xây |
| Danh sách tranh chấp lọc theo người đã nghỉ việc | Vẫn trả dòng — tranh chấp không được "biến mất" | Không có | Chỉ đặc tả |
| Tranh chấp quá hạn đã đóng muộn | Không còn tính là "quá hạn" (`is_overdue` loại trừ đã giải quyết/đã đóng) | Không có | Chỉ đặc tả |
| Chuyển trạng thái tranh chấp ngoài 6 chuyển hợp lệ | `P0055` | "Không chuyển từ「{từ}」sang「{sang}」được." | Chỉ đặc tả (đề xuất) |
| Chuyển sang "đã giải quyết" thiếu loại kết quả hoặc kết quả cuối | `P0056` | "Phải ghi loại kết quả và kết quả cuối trước khi đóng vụ." | Chỉ đặc tả (đề xuất) |
| Hạn xử lý rơi vào ngày nghỉ chợ | `P0057` (dùng chung với lịch nghỉ) | "Ngày {ngày} là 休市日 của chợ, không phải 業務日." | Chỉ đặc tả (đề xuất) |
| Dời hạn mà không nêu lý do (< 10 ký tự) | Chặn ở form | "Dời ngày dự kiến phải nêu lý do từ 10 ký tự." | Chỉ đặc tả |
| Kết quả tranh chấp làm đổi số liệu ngày đã chốt | Màn **không sửa số**; chỉ dẫn sang phiếu điều chỉnh. Nếu ngày **chưa** chốt, không có gì chặn — đây mới là chỗ nguy hiểm | Không có | Chỉ đặc tả |
| Vụ tranh chấp đã đóng có tình tiết mới | Không mở lại; lập vụ mới trỏ về vụ cũ | Không có | Chỉ đặc tả |
| Đặt ngày nghỉ cho ngày đã có dữ liệu hoặc đã chốt | `P0058` | "Ngày {ngày} đã có dữ liệu nghiệp vụ (hoặc đã chốt sổ), không đặt thành 休市日 được." | Chỉ đặc tả (đề xuất) |
| Lịch ở chế độ đồng bộ nguồn ngoài mà vẫn cố ghi | `P0059` | "Lịch nghỉ của chợ này đồng bộ từ nguồn ngoài — màn hình chỉ xem." | Chỉ đặc tả (đề xuất) |
| Thêm ngày nghỉ đã tồn tại | Idempotent: không làm gì, không ghi dòng mới | "Ngày {ngày} đã là 休市日." | Chỉ đặc tả |
| Chuỗi ngày nghiệp vụ sắp cạn mà không ai để ý | Sáng không mở được ngày nghiệp vụ mới trong khi ca bắt đầu 02:00; cần cảnh báo khi còn dưới 30 ngày | Cảnh báo ở lưới lịch | Chỉ đặc tả (kênh cảnh báo chưa quyết) |
