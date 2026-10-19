# Edge Cases — F004_TradeRecording

Nguồn: [SCR010](../../screens/SCR010_NegotiatedTradeConfirm/spec.md) · [SCR011](../../screens/SCR011_AuctionResultEntry/spec.md) · [SCR012](../../screens/SCR012_TradeList/spec.md) · [SCR013](../../screens/SCR013_TradeDetail/spec.md) mục 4, 5, 9; `app/(app)/trades/actions.ts`; migration `0007`, `0030`, `0034`, `0035`. `P0062` do SCR013 đề xuất, chưa cài. Rẽ nhánh luôn theo `error.code`, không theo `error.message`.

| Scenario | What Happens | User-Facing Message | Trạng thái |
|----------|--------------|---------------------|------------|
| Thiếu một trong 4 ô, hoặc số lượng/đơn giá không phải số dương (SCR010) | Chặn định dạng ở client, **không** kiểm tư cách/số lượng ở client | "Vui lòng nhập đủ mã lô, mã người mua, số lượng và đơn giá." / "Số lượng và đơn giá phải là số dương." | Đã xây |
| Mã lô hoặc mã người mua không tra được | `P0001` trước khi gọi RPC; không bản ghi nào | "Không tìm thấy mã lô hoặc mã người mua đã nhập. Kiểm tra lại hai mã này." | Đã xây |
| Người mua hết hiệu lực | `P0003` trong RPC, cùng transaction; **không tạo giao dịch** | "Người mua {tên} đã hết hạn tham gia từ {ngày}, căn cứ {căn cứ}. Liên hệ {phòng} để xử lý." (bản tiếng Nhật tra lại theo căn cứ) | Đã xây |
| Lô không đủ số lượng | `P0004` kèm `remaining`, `requested`, `sold_to[]` | "Lô {mã} chỉ còn {x} {đơn vị}, không đủ cho yêu cầu {y} {đơn vị}. Đã bán cho: {mã giao dịch}." | Đã xây |
| Hai bàn chốt cùng một lô vài giây | `SELECT … FOR UPDATE` khoá dòng lô; người sau thấy số đã trừ → có thể `P0004` | như dòng trên | Đã xây |
| Vùng tham khảo báo "còn đủ" nhưng đã cũ khi bấm chốt | Kết quả cuối do RPC quyết; vùng tham khảo **không** được chặn nút chốt | "Thông tin tham khảo — hệ thống vẫn kiểm tra lại đầy đủ khi bấm chốt, kết quả cuối cùng có thể khác nếu dữ liệu vừa thay đổi." | Đã xây |
| Ngày nghiệp vụ của lô đã chốt sổ | `P0002` | "Ngày nghiệp vụ {ngày} đã chốt sổ lúc {giờ}, không thể ghi thêm giao dịch." | Đã xây |
| Chức danh khác `trading` | `P0005` ở Server Action, rồi ở RPC | "Chức danh của bạn không được phép thực hiện thao tác này." | Đã xây |
| Mất mạng lúc bấm chốt | Gửi thất bại; **4 ô giữ nguyên**, không bản ghi nào được tạo | "Mất kết nối mạng — không thể gửi giao dịch. Kiểm tra lại kết nối rồi thử lại. Chưa có bản ghi nào được tạo." | Đã xây |
| Chốt xong, ca sau ngồi vào máy | Form xoá sạch cả vùng tham khảo; không autofill người mua cũ | Không có | Đã xây |
| Ghi kết quả đấu giá thiếu thời điểm phiên hoặc người xác nhận | Chặn ở Server Action (mã `generic`); lưới cuối là trigger hoãn → `P0008` lúc commit nếu có đường ghi vòng qua | "Có lỗi xảy ra khi gửi giao dịch, vui lòng thử lại." | Đã xây |
| Chèn thẳng SQL một dòng `auction` thiếu hai trường | Bị từ chối `P0008` lúc commit, không để lại dòng rác (ADR-004, đã kiểm hai chiều) | "Giao dịch đấu giá thiếu thời điểm phiên hoặc người xác nhận" (lỗi hạ tầng, người dùng thường không gặp) | Đã xây |
| Mất mạng khi ghi kết quả đấu giá | Banner lỗi mạng; **6 ô giữ nguyên** (đã sửa để không dùng `useActionState`) | "Mất kết nối mạng — không thể gửi kết quả. Kiểm tra lại kết nối rồi thử lại." | Đã xây |
| Lọc theo mã người mua gõ sai | Không phải lỗi; danh sách rỗng | "Chưa có giao dịch nào khớp bộ lọc." | Đã xây |
| `receiving` mở `/trades` | Lớp chặn route đưa sang `/403`; dữ liệu `trades` vẫn đọc được ngầm cho màn giao hàng (migration `0013`) | Trang "không đủ quyền" | Đã xây (hai tầng cố ý lệch) |
| Danh sách rất dài (700–1.200 giao dịch/ngày) | `listTrades` (`lib/domain/trades.ts:61-86`) không gọi `.range()`/`.limit()`. Giới hạn hàng mặc định của PostgREST (`supabase/config.toml`: `max_rows = 1000`, cấu hình local; dự án thật chưa kiểm) có thể **cắt danh sách im lặng** ở ngày nhiều giao dịch — người dùng không biết thiếu dòng | Không có | Đã xây — **rủi ro** (chưa tái hiện; chưa có ngưỡng phân trang) |
| Huỷ giao dịch đã có lần giao (kể cả lần còn `chờ giao`/`đang giao`) | `P0062` khi `exists(select 1 from deliveries where trade_id = …)`; **không** dựa vào `v_trade_delivery_status.delivery_count` (view chỉ đếm `completed`/`exception`, `0006_derived_views.sql:35`, bộ lọc `:42`) — nếu dùng view, lần giao đang chờ lọt guard và giao dịch bị huỷ khi hàng đã lên xe. Dẫn sang xử lý ngoại lệ giao hàng | "Giao dịch này đã có {n} lần giao ({x} {đơn vị}). Phải xử lý ngoại lệ giao hàng trước khi huỷ." | Chỉ đặc tả (đề xuất) |
| Huỷ sau khi ngày đã chốt sổ | `P0002`, dẫn sang phiếu điều chỉnh; nút huỷ ẩn hẳn | "Ngày nghiệp vụ {ngày} đã chốt sổ… Huỷ giao dịch sau chốt sổ phải lập phiếu điều chỉnh." | Chỉ đặc tả |
| Gõ lại mã giao dịch không khớp khi huỷ | Chặn ở hộp thoại | "Mã giao dịch gõ lại không khớp." | Chỉ đặc tả |
| Hai người cùng huỷ một giao dịch | Người sau thấy trạng thái đã huỷ, trang tự nạp lại | "Giao dịch này đã được huỷ lúc {giờ}." | Chỉ đặc tả |
| Tab lịch sử sửa của `trading`/`reconcile` | Rỗng nếu RLS `audit_log` vẫn chỉ cho quản trị đọc; nhật ký cũng **thiếu cột lý do** | "Chưa có thay đổi nào được ghi vết." (gây hiểu sai) | Chỉ đặc tả (cần migration cột + nới RLS) |
