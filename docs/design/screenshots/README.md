# Ảnh chụp màn hình LAB-3 (production) — nền cho wireframe LAB-4

Toàn bộ ảnh dưới đây chụp từ bản deploy thật **https://lab3-product-a.vercel.app**,
viewport 1440×900, `prefers-color-scheme` ép `dark` để xác nhận bản sửa giao diện
(nền trắng, font Geist) đã lên production — không phải chụp từ `next dev`.
Dùng làm ảnh nền tham chiếu khi viết screen spec cho LAB-4, KHÔNG phải ảnh final
để ký duyệt UI.

Ngày chụp: 2026-10-04.

## Chỉ mục

| File | Màn hình (SC) | Route | Vai trò đăng nhập | Ngôn ngữ | Mô tả trạng thái |
|---|---|---|---|---|---|
| `sc02-login-vi.png` | SC-02 | `/login` | (chưa đăng nhập) | vi | Form đăng nhập trống, mặc định |
| `sc02-login-ja.png` | SC-02 | `/login` | (chưa đăng nhập) | ja | Form đăng nhập trống, cookie `locale=ja` |
| `sc01-home-admin-vi.png` | SC-01 | `/` | admin@demo.local | vi | Trang chủ — tổng quan hệ thống (số tài khoản theo chức danh) |
| `sc01-home-receiving-vi.png` | SC-01 | `/` | receiving@demo.local | vi | Trang chủ — vai trò Tiếp nhận & Giao hàng |
| `sc01-home-trading-vi.png` | SC-01 | `/` | trading@demo.local | vi | Trang chủ — vai trò Giao dịch |
| `sc01-home-reconcile-vi.png` | SC-01 | `/` | reconcile@demo.local | vi | Trang chủ — vai trò Đối chiếu |
| `sc03-accounts-vi.png` | SC-03 | `/accounts` | admin@demo.local | vi | Danh sách tài khoản nội bộ, mặc định không lọc |
| `sc04-account-detail-vi.png` | SC-04 | `/accounts/f0261dd1-c24d-4adc-bec9-078c711d9925` | admin@demo.local | vi | Chi tiết tài khoản & quyền — tài khoản `reconcile@demo.local` (dòng đầu danh sách) |
| `sc07-lot-new-vi.png` | SC-07 | `/lots/new` | receiving@demo.local | vi | Form nhập lô hàng, trống, mặc định |
| `sc07-lot-new-ja.png` | SC-07 | `/lots/new` | receiving@demo.local | ja | Form nhập lô hàng, cookie `locale=ja` |
| `sc07-focus-vi.png` | SC-07 | `/lots/new` | receiving@demo.local | vi | Sau 3 lần Tab từ ô mã chủ hàng (autofocus) — viền focus xanh dương rõ trên ô "Số chứng từ tiếp nhận", minh hoạ keyboard-first |
| `sc08-lots-vi.png` | SC-08 | `/lots` | receiving@demo.local | vi | Danh sách lô hàng, mặc định không lọc |
| `sc10-trade-negotiated-vi.png` | SC-10 | `/trades/negotiated` | trading@demo.local | vi | Form chốt giao dịch thoả thuận, trống, mặc định |
| `sc10-trade-negotiated-ja.png` | SC-10 | `/trades/negotiated` | trading@demo.local | ja | Form chốt giao dịch thoả thuận, trống, cookie `locale=ja` |
| `sc10-preview-vi.png` | SC-10 | `/trades/negotiated` | trading@demo.local | vi | Đã gõ mã lô `LOT-20260924-0008` + mã người mua `PTA-02`, blur cả hai ô — vùng tham khảo hiện đủ thông tin lô (mặt hàng, trạng thái, còn lại) và tư cách người mua (còn hiệu lực), thành tiền tạm tính; CHƯA bấm chốt |
| `sc10-reject-p0003-vi.png` | SC-10 | `/trades/negotiated` | trading@demo.local | vi | Lô hợp lệ `LOT-20260924-0008`, người mua **PTI-01 đã hết hạn tư cách**, số lượng nhỏ (5), đơn giá 100 → bấm chốt (Enter ở ô đơn giá) → banner lỗi P0003 "Người mua đã hết hạn tham gia từ 2026-09-14, căn cứ Giấy phép..." |
| `sc10-reject-p0004-vi.png` | SC-10 | `/trades/negotiated` | trading@demo.local | vi | Cùng lô, người mua hợp lệ `PTA-02`, số lượng vượt tồn (999999) → bấm chốt → banner lỗi P0004 "Lô ... chỉ còn 60 kg, không đủ cho yêu cầu 999999 kg..." |
| `sc11-trade-auction-vi.png` | SC-11 | `/trades/auction` | trading@demo.local | vi | Form ghi kết quả đấu giá, trống, mặc định |
| `sc12-trades-vi.png` | SC-12 | `/trades` | trading@demo.local | vi | Danh sách giao dịch, mặc định không lọc |
| `sc14-delivery-new-vi.png` | SC-14 | `/deliveries/new` | receiving@demo.local | vi | Form ghi giao hàng, trống, mặc định |
| `sc18-reconciliation-vi.png` | SC-18 | `/reconciliation` | reconcile@demo.local | vi | Màn đối chiếu/chốt sổ, mặc định |
| `sc18-reconciliation-ja.png` | SC-18 | `/reconciliation` | reconcile@demo.local | ja | Màn đối chiếu/chốt sổ, cookie `locale=ja` |
| `sc403-forbidden-vi.png` | SC-403 | `/lots` → đẩy `/403` | admin@demo.local | vi | admin (role `sys_admin`) mở `/lots` (route chỉ cho `receiving`/`trading`) → bị route-policy đẩy về `/403`, trang "Không đủ quyền truy cập" |

## Ghi chú kỹ thuật

- Mã lô `LOT-20260924-0008` dùng cho các kịch bản SC-10 (preview, reject P0003,
  reject P0004) được lấy **động** từ `/lots?status=公開` lúc chụp (lô đang ở
  trạng thái "Đã mở bán", còn lại 60 kg tại thời điểm chụp) — không phải số
  tự đoán theo ngày hiện tại, vì dữ liệu seed trên môi trường demo được nạp
  vào ngày khác với ngày chụp ảnh.
- Người mua `PTI-01` (trung gian) và `PTA-02` (người mua được chấp thuận) là
  dữ liệu seed cố định: `PTI-01` bị đặt hết hạn tư cách từ `2026-09-14` để
  luôn tái hiện được ca từ chối P0003; `PTA-02` còn hiệu lực.
- Cả hai kịch bản từ chối (P0003, P0004) đều bị RPC `confirm_negotiated_trade`
  chặn trong transaction phía server trước khi trừ tồn — không để lại tác
  dụng phụ nào lên dữ liệu seed, có thể chụp lại nhiều lần.
- Tài khoản `sys_admin` không có tài khoản nào khác xếp trước trong danh sách
  `/accounts` tại thời điểm chụp nên `sc04-account-detail-vi.png` rơi vào
  tài khoản `reconcile@demo.local` (dòng đầu bảng, sắp xếp mặc định của
  `listAccounts`) — không phải lựa chọn chủ ý.

## Việc đã thử nhưng không dựng được

Không có — toàn bộ 23 ảnh trong bảng trên đều chụp thành công, bao gồm cả
5 trạng thái lỗi/đặc biệt của Nhóm C.
