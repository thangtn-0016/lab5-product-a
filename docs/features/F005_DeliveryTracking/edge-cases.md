# Edge Cases — F005_DeliveryTracking

Nguồn: [SCR014](../../screens/SCR014_DeliveryEntry/spec.md) · [SCR015](../../screens/SCR015_DeliveryList/spec.md) · [SCR016](../../screens/SCR016_DeliveryDetail/spec.md) · [SCR017](../../screens/SCR017_DeliveryException/spec.md) mục 4, 9, 10; `app/(app)/deliveries/actions.ts`; `0007_rpc_functions.sql:114-165`; kết quả kiểm chứng LAB-3 ở (tài liệu gốc trong repo, không nộp) `verification-checklist.md` §2, §3, §6. `P0063`–`P0065` do SCR016/SCR017 đề xuất, chưa cài.

| Scenario | What Happens | User-Facing Message | Trạng thái |
|----------|--------------|---------------------|------------|
| Thiếu mã giao dịch hoặc số lượng; số lượng không dương | Chặn định dạng ở client | "Vui lòng nhập đủ mã giao dịch và số lượng." / "Số lượng phải là số dương." | Đã xây |
| Chọn trạng thái "ngoại lệ" mà để trống lý do | Chặn ở client | "Vui lòng nhập lý do khi trạng thái là ngoại lệ." | Đã xây |
| Mã giao dịch không tồn tại | `P0001` trước RPC | "Không tìm thấy mã giao dịch đã nhập. Kiểm tra lại mã này." | Đã xây |
| Giao vượt số còn lại | `P0006` trong RPC; **không** ghi dòng nào (đã kiểm: số dòng `deliveries` không đổi) | "Giao dịch {mã} đã giao {x} {đơn vị}, chỉ còn {y} {đơn vị} — không đủ cho yêu cầu {z} {đơn vị}." | Đã xây |
| Người nhận hết hiệu lực khi ghi lần giao | `P0003` — **tư cách người nhận cũng phải còn hạn ở khâu giao hàng** | "Người nhận {tên} đã hết hạn tham gia từ {ngày}, căn cứ {căn cứ}. Liên hệ {phòng} để xử lý." | Đã xây |
| Ngày nghiệp vụ đã chốt sổ | `P0002`; Server Action gọi **riêng** `log_blocked_write_attempt` để ghi lần thử vào nhật ký (ngoài transaction đã rollback) | "Ngày nghiệp vụ {ngày} đã chốt sổ lúc {giờ}, không thể ghi thêm lần giao." | Đã xây (đã kiểm: dòng nhật ký xuất hiện) |
| Ghi trực tiếp vào bảng giao hàng của ngày đã chốt (bỏ qua RPC) | Trigger `P0009`, chỉ xảy ra khi có người ghi tay | "Ngày nghiệp vụ đã chốt sổ, không thể sửa" (lỗi hạ tầng) | Đã xây |
| Chức danh khác `receiving` (kể cả `trading`) mở `/deliveries/new` | Lớp chặn route → `/403`; Server Action và RPC từ chối `P0005` | "Chức danh của bạn không được phép thực hiện thao tác này." | Đã xây |
| Mất mạng lúc ghi | Banner lỗi; **dữ liệu form giữ nguyên**, chưa có dòng nào | "Mất kết nối mạng — không thể gửi lần giao. Kiểm tra lại kết nối rồi thử lại. Chưa có bản ghi nào được tạo." | Đã xây |
| Giao dịch bị chia từ lần thứ 3 | Banner cảnh báo, **không chặn**; ngưỡng 3 là giả định | "Giao dịch này đã chia {n} lần giao — vượt ngưỡng cảnh báo {ngưỡng} lần. Đây chỉ là cảnh báo tham khảo, vẫn có thể tiếp tục ghi nếu đúng thực tế hiện trường." | Đã xây (ngưỡng là giả định) |
| Hai người cùng ghi lần giao cho một giao dịch | Dòng giao dịch khoá `FOR UPDATE`; `seq` cấp = max + 1 trong khoá; ràng buộc UNIQUE `(trade_id, seq)` là lưới cuối | Người đến sau thấy số còn lại đã đổi → có thể `P0006` | Đã xây |
| Trả hàng hoặc huỷ một phần được ghi qua màn ghi lần giao | **Số "đã giao" tăng** thay vì giảm (view cộng mọi dòng ngoại lệ) — lỗi B-01 trong README của screen-specs, đã xác minh bằng `pg_get_viewdef` | Không có (sai ngầm) | Đã xây — **lỗi thật, cần sửa trước khi làm SCR017** |
| Giao đủ cho một giao dịch nhưng lô bán cho nhiều giao dịch | Lô bị lật sang `引渡し完了` sớm; vô hiệu hoá phép kiểm tư cách của bước chốt hoàn tất. **Đặc tả (PREMISE G3):** lô chỉ sang `引渡し完了` khi mọi giao dịch `confirmed` của lô đã hoàn tất | Không có (sai ngầm) | Đã xây — **lỗi thật** (lỗi B-02 trong README của screen-specs) |
| Giao thừa (`over`) | Không ghi được bằng đường hiện có vì `P0006`; chỉ vào qua phiếu đã duyệt | "Giao dịch… không đủ cho yêu cầu…" | Đã xây (thiếu tính năng, đường đúng là SCR017) |
| Ngoại lệ lệch số lượng ghi thẳng ở SCR014 mà chưa có phiếu duyệt | `P0064`, dẫn sang màn lập phiếu | "Ngoại lệ「{loại}」làm lệch {x} {đơn vị}, phải lập phiếu ngoại lệ có duyệt." | Chỉ đặc tả (đề xuất) |
| Người duyệt trùng người lập phiếu ngoại lệ, hoặc tài khoản `reconcile` không có cờ `can_approve` | `P0065` cho cả hai trường hợp (`details.reason` = `self_approve` hoặc `no_can_approve`, cùng cách `P0013` ở SCR036); nút duyệt ẩn sẵn với phiếu của chính mình và với tài khoản thiếu cờ `can_approve` | "Phiếu do chính bạn lập — người duyệt phải khác người đề nghị." | Chỉ đặc tả (đề xuất) |
| Chốt hoàn tất khi số lượng chưa khớp quy tắc thanh toán | `P0063` kèm số liệu và phiên bản quy tắc | "Đã giao {x}/{y} {đơn vị}, còn lại {z} — vượt dung sai {t} của quy tắc thanh toán. Chưa chốt hoàn tất được." | Chỉ đặc tả (đề xuất; dung sai = 0 là giả định) |
| Chốt hoàn tất khi còn lần giao `chờ giao`/`đang giao` | `P0063` nhánh b, nêu các lần chưa kết thúc | "Lần giao thứ {n} chưa kết thúc. Cập nhật trạng thái các lần đó trước khi chốt." | Chỉ đặc tả |
| Phiếu ngoại lệ đang chờ duyệt | **Không** tính vào "đã giao" cho tới khi duyệt | Không có | Chỉ đặc tả |
| Danh sách giao hàng rất dài | Mặc định lọc theo ngày; bỏ lọc phải có giới hạn | "Không tải được danh sách giao hàng…" khi lỗi truy vấn | Chỉ đặc tả |
| Mất mạng khi ghi lần giao ở cầu cảng (mục tiêu: vẫn ghi được) | LAB-3: thất bại kèm lỗi rõ. Đặc tả: lưu tạm + tự gửi + chống trùng; đồng bộ về ngày đã chốt → `P0067`, bản ghi **ở lại hàng đợi** chờ phiếu điều chỉnh | Chỉ báo "đang chờ đồng bộ" | Chỉ đặc tả (MOCK-2) |
