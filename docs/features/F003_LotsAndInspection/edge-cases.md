# Edge Cases — F003_LotsAndInspection

Nguồn: [SCR007](../../screens/SCR007_LotEntry/spec.md) · [SCR008](../../screens/SCR008_LotList/spec.md) · [SCR009](../../screens/SCR009_LotDetail/spec.md) mục 4, 5, 9, 10; `app/(app)/lots/actions.ts`; migration `0004`, `0009`, `0010`, `0020`, `0021`. Mã `P0060`/`P0061` do SCR009 đề xuất, chưa cài.

| Scenario | What Happens | User-Facing Message | Trạng thái |
|----------|--------------|---------------------|------------|
| Để trống mã chủ hàng, mã mặt hàng, số lượng hoặc ngày | Server Action trả lỗi từng ô, **không** gọi cơ sở dữ liệu; các ô khác giữ nguyên | "Nhập mã chủ hàng." / "Nhập mã mặt hàng." / "Nhập số lượng." / "Chọn ngày nghiệp vụ." | Đã xây |
| Mã chủ hàng hoặc mặt hàng không tồn tại | Tra mã → không thấy → lỗi ở ô tương ứng | "Không tìm thấy chủ hàng với mã này. Kiểm tra lại mã." (tương tự cho mặt hàng) | Đã xây |
| Số lượng không phải số, bằng 0 hoặc âm | Chặn ở Server Action; ràng buộc `quantity > 0` ở bảng là lưới an toàn | "Số lượng phải là số lớn hơn 0." | Đã xây |
| Mất mạng hoặc sinh mã lô thất bại | Trả lỗi `network`; **dữ liệu form giữ nguyên**, chưa có bản ghi | "Chưa lưu được do lỗi kết nối. Dữ liệu trên form vẫn được giữ nguyên, hãy thử lưu lại." | Đã xây |
| Chọn ngày nghiệp vụ đã chốt sổ khi nhập lô | Trigger `guard_closed_day` chặn (`P0009`) nhưng Server Action không bắt riêng mã → rơi vào thông báo "lỗi kết nối" chung, **sai nguyên nhân** | "Chưa lưu được do lỗi kết nối…" | Đã xây (khoảng trống diễn đạt, ghi nhận ở SCR007 mục 10) |
| Mã lô sinh trùng với lô đã seed (sequence chưa đuổi kịp) | Ràng buộc UNIQUE `lot_code` chặn; đã vá bằng `setval` ở `0021` | Không có (đã vá) | Đã xây |
| Hai người cùng bấm "Mở bán" một lô | Điều kiện trạng thái nằm trong mệnh đề UPDATE; người đến sau không khớp dòng nào | "Lô này không ở trạng thái có thể mở bán." | Đã xây |
| Chức danh khác `receiving` gọi nhập lô hoặc mở bán | Server Action từ chối trước khi chạm dữ liệu; RLS INSERT/UPDATE chỉ cho `receiving` | "Chức danh của bạn không được phép nhập lô hàng." | Đã xây |
| Mở bán lô của ngày đã chốt | Trigger chặn `P0009`; mã giao diện cụ thể **chưa xác định** (SCR008 mục 10) | Chưa xác định | Đã xây (chưa kiểm) |
| Chốt giao dịch trên lô **chưa mở bán** (`受付`/`下見済`) | RPC chốt giao dịch **không kiểm trạng thái lô** → vẫn cho chốt nếu đủ số lượng và tư cách | Không có | Đã xây — **khoảng trống so với FIG-011** (quan sát từ `0007_rpc_functions.sql:41-94`); spec không cấp mã mới |
| Hai bàn cùng chốt một lô, tổng vượt số lượng | Dòng lô khoá `FOR UPDATE`; người sau nhận `P0004` kèm danh sách giao dịch đã mua | "Lô {mã} chỉ còn {x} {đơn vị}, không đủ cho yêu cầu {y}. Đã bán cho: {mã giao dịch}." | Đã xây |
| Sửa `quantity` của lô trực tiếp qua giao diện lập trình khi đã bán | Chính sách RLS `lots_update` cho `receiving` UPDATE **mọi cột**, không trigger chặn → có thể làm `remaining` âm | Không có | Đã xây — **khoảng trống** (xem [technical-spec](./technical-spec.md#business-rules), BR-LOT-02) |
| Ghi đánh giá cho lô đã `公開`/`確定`/`引渡し完了` | `P0060` | "Lô đang ở trạng thái {trạng thái}, không ghi thêm kết quả đánh giá được." | Chỉ đặc tả (đề xuất) |
| Ngày đánh giá nằm ngoài ngày nghiệp vụ của lô | Chặn ở form | "Thời điểm đánh giá phải nằm trong ngày nghiệp vụ {ngày}." | Chỉ đặc tả |
| Hạ số lượng lô thấp hơn phần đã bán | `P0061` kèm danh sách giao dịch đã mua | "Lô đã bán {x} {đơn vị} cho {mã}; không hạ số lượng xuống {y} {đơn vị} được." | Chỉ đặc tả (đề xuất) — nếu khách chốt "số lượng bất biến" thì bỏ |
| Điều chỉnh/ghi đánh giá lô thuộc ngày đã chốt | `P0002` (kiểm trong RPC) hoặc `P0009` (trigger); form ẩn, dẫn sang phiếu điều chỉnh | "Ngày nghiệp vụ {ngày} đã chốt sổ lúc {giờ}. Muốn sửa phải lập phiếu điều chỉnh." | Chỉ đặc tả |
| Chức danh đối chiếu mở chi tiết lô | Hiện RLS `lots` **không cho `reconcile` đọc** → màn trả rỗng dù route cho qua nếu không mở quyền | Không có | Chỉ đặc tả (cần migration mở quyền đọc) |
| Mất mạng khi nhập lô (mục tiêu: vẫn ghi được) | LAB-3: submit thất bại kèm lỗi rõ, không có hàng đợi. Đặc tả: lưu tạm, tự gửi khi có mạng, khoá chống trùng do client sinh | LAB-3: như dòng "mất mạng". Đặc tả: chỉ báo "đang chờ đồng bộ" | Chỉ đặc tả (MOCK-2, [SCR035](../../screens/SCR035_SyncQueue/spec.md)) |
