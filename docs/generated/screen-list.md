# Screen List

**Project**: Product A — 水産物卸売市場 業務支援システム (hệ thống hỗ trợ nghiệp vụ chợ đầu mối thủy sản)
**Generated**: 2026-10-06
**Analysis Scope**: 37 màn hình = 35 màn của bản đầy đủ LAB-1 (11 màn đã xây ở LAB-3 + 24 màn thiết kế) + 2 màn thiết kế mới của LAB-4 (SCR036, SCR037). Nguồn: các screen spec trong [screens/](../screens/), danh sách màn của LAB-1 (tài liệu gốc trong repo, không nộp: `product-a-screen-list`), code `app/`.

**Code Format**: `SCR###_NameSlug` (ví dụ `SCR010_NegotiatedTradeConfirm`) | `SCR###/REG###` cho vùng của màn composite. Alias LAB-1 (`SC-xx`) ghi kèm ở mỗi mục.

**Note**: Ánh xạ tính năng ở [feature-list.md](./feature-list.md) (cột Feature ở bảng dưới chỉ để tra nhanh); ánh xạ user story ở `user-stories.md`. Trang `/403` và route `/deliveries` **không** nằm trong 37 màn — xem [screen-flow.md](./screen-flow.md) § Ngoài 37 màn.

**Trạng thái**: ● đã xây ở LAB-3 (đặc tả đọc từ code thật + ảnh deploy) · ○ thiết kế, chưa có code. Route của màn ○ là **đề xuất**. **Role** `party_admin` `rule_admin` `ops_admin` `market_manager` `internal_audit` là **đề xuất** (chưa có trong enum `app_role`, prototype chỉ có 4: `sys_admin` `receiving` `trading` `reconcile`).

**Region Guidance**: chỉ khai báo REG khi vùng có ≥1 tín hiệu độc lập (endpoint riêng, ghi riêng, trạng thái tải riêng). Màn không khai REG là atomic (có thể gồm nhiều khối nhưng một nguồn dữ liệu + một đường ghi).

## Screen Index

| Code | Alias | Name | Type | ⌨️ | Feature | Route | Built | Spec |
|------|-------|------|------|:--:|---------|-------|:-----:|------|
| SCR001_RoleHome | SC-01 | Trang chủ theo chức danh | atomic (4 biến thể) | | F010 | `/` | ● | [spec](../screens/SCR001_RoleHome/spec.md) |
| SCR002_Login | SC-02 | Đăng nhập | atomic | | F001 | `/login` | ● | [spec](../screens/SCR002_Login/spec.md) |
| SCR003_AccountList | SC-03 | Danh sách tài khoản nội bộ | atomic | | F001 | `/accounts` | ● | [spec](../screens/SCR003_AccountList/spec.md) |
| SCR004_AccountDetail | SC-04 | Chi tiết tài khoản & quyền | composite | | F001 | `/accounts/[id]` | ● | [spec](../screens/SCR004_AccountDetail/spec.md) |
| SCR005_ParticipantList | SC-05 | Danh sách người tham gia | atomic | | F002 | `/participants` | ○ | [spec](../screens/SCR005_ParticipantList/spec.md) |
| SCR006_ParticipantProfile | SC-06 | Hồ sơ người tham gia (4 biến thể) | atomic (4 biến thể) | | F002 | `/participants/[id]`, `/participants/new` | ○ | [spec](../screens/SCR006_ParticipantProfile/spec.md) |
| SCR007_LotEntry | SC-07 | Nhập lô hàng | atomic | ⌨️ 📴 | F003 | `/lots/new` | ● | [spec](../screens/SCR007_LotEntry/spec.md) |
| SCR008_LotList | SC-08 | Danh sách lô hàng | composite | | F003 | `/lots` | ● | [spec](../screens/SCR008_LotList/spec.md) |
| SCR009_LotDetail | SC-09 | Chi tiết lô hàng | composite | | F003 | `/lots/[id]` | ○ | [spec](../screens/SCR009_LotDetail/spec.md) |
| SCR010_NegotiatedTradeConfirm | SC-10 | Chốt giao dịch thỏa thuận | composite | ⌨️ | F004 | `/trades/negotiated` | ● | [spec](../screens/SCR010_NegotiatedTradeConfirm/spec.md) |
| SCR011_AuctionResultEntry | SC-11 | Ghi kết quả phiên đấu giá | atomic | | F004 | `/trades/auction` | ● | [spec](../screens/SCR011_AuctionResultEntry/spec.md) |
| SCR012_TradeList | SC-12 | Danh sách giao dịch | atomic | | F004 | `/trades` | ● | [spec](../screens/SCR012_TradeList/spec.md) |
| SCR013_TradeDetail | SC-13 | Chi tiết giao dịch | atomic | | F004 | `/trades/[id]` | ○ | [spec](../screens/SCR013_TradeDetail/spec.md) |
| SCR014_DeliveryEntry | SC-14 | Ghi lần giao hàng | atomic | ⌨️ 📴 | F005 | `/deliveries/new` | ● | [spec](../screens/SCR014_DeliveryEntry/spec.md) |
| SCR015_DeliveryList | SC-15 | Danh sách giao hàng | atomic | | F005 | `/deliveries` | ○ | [spec](../screens/SCR015_DeliveryList/spec.md) |
| SCR016_DeliveryDetail | SC-16 | Chi tiết giao hàng theo giao dịch | atomic | | F005 | `/deliveries/trade/[tradeId]` | ○ | [spec](../screens/SCR016_DeliveryDetail/spec.md) |
| SCR017_DeliveryException | SC-17 | Ghi ngoại lệ giao hàng 🔒 | atomic | | F005 | `/deliveries/exceptions/new`, `/deliveries/exceptions`, `/deliveries/exceptions/[id]` | ○ | [spec](../screens/SCR017_DeliveryException/spec.md) |
| SCR018_DailyReconciliation | SC-18 | Bảng đối chiếu ngày & chốt sổ | atomic | | F006 | `/reconciliation` | ● | [spec](../screens/SCR018_DailyReconciliation/spec.md) |
| SCR019_DisputeList | SC-19 | Danh sách tranh chấp | atomic | | F006 | `/disputes` | ○ | [spec](../screens/SCR019_DisputeList/spec.md) |
| SCR020_DisputeDetail | SC-20 | Chi tiết tranh chấp | atomic | | F006 | `/disputes/[id]` | ○ | [spec](../screens/SCR020_DisputeDetail/spec.md) |
| SCR021_MarketCalendar | SC-21 | Lịch nghỉ chợ & ngày nghiệp vụ | atomic | | F006 | `/markets/[id]/calendar` | ○ | [spec](../screens/SCR021_MarketCalendar/spec.md) |
| SCR022_CorrectionRequestCreate | SC-22 | Tạo phiếu xin điều chỉnh 🔒 | atomic | | F007 | `/corrections/new` | ○ | [spec](../screens/SCR022_CorrectionRequestCreate/spec.md) |
| SCR023_PendingRequestList | SC-23 | Danh sách phiếu chờ duyệt | atomic | | F007 | `/corrections` | ○ | [spec](../screens/SCR023_PendingRequestList/spec.md) |
| SCR024_RequestDetailApproval | SC-24 | Chi tiết phiếu & duyệt 🔒 | composite | | F007 | `/corrections/[id]` | ○ | [spec](../screens/SCR024_RequestDetailApproval/spec.md) |
| SCR025_IncentiveResult | SC-25 | Kết quả tính tiền khuyến khích | atomic | | F008 | `/incentives`, `/incentives/[period_id]/[participant_id]` | ○ | [spec](../screens/SCR025_IncentiveResult/spec.md) |
| SCR026_RateVersionList | SC-26 | Danh sách phiên bản tỷ lệ | atomic | | F008 | `/incentive-rates` | ○ | [spec](../screens/SCR026_RateVersionList/spec.md) |
| SCR027_RateVersionApproval | SC-27 | Chi tiết & duyệt phiên bản tỷ lệ 🔒 | atomic | | F008 | `/incentive-rates/new`, `/incentive-rates/[version_id]` | ○ | [spec](../screens/SCR027_RateVersionApproval/spec.md) |
| SCR028_ReportCatalog | SC-28 | Danh mục báo cáo | atomic | | F009 | `/reports` | ○ | [spec](../screens/SCR028_ReportCatalog/spec.md) |
| SCR029_ReportViewExport | SC-29 | Xem & xuất báo cáo | atomic | | F009 | `/reports/[report_code]` | ○ | [spec](../screens/SCR029_ReportViewExport/spec.md) |
| SCR030_AccountingBatch | SC-30 | Trạng thái batch & lô xuất kế toán | composite | | F009 | `/operations/batches`, `/operations/acc-exports`, `/operations/acc-exports/[batch_code]` | ○ | [spec](../screens/SCR030_AccountingBatch/spec.md) |
| SCR031_NotificationInbox | SC-31 | Hộp thông báo | atomic | | F010 | `/notifications` | ○ | [spec](../screens/SCR031_NotificationInbox/spec.md) |
| SCR032_NotificationSettings | SC-32 | Cấu hình thông báo | atomic | | F010 | `/notifications/settings` | ○ | [spec](../screens/SCR032_NotificationSettings/spec.md) |
| SCR033_OpsAlertQueue | SC-33 | Hàng cảnh báo vận hành | atomic | | F010 | `/ops/alerts` | ○ | [spec](../screens/SCR033_OpsAlertQueue/spec.md) |
| SCR034_AuditLogSearch | SC-34 | Tra cứu nhật ký kiểm toán | atomic | | F011 | `/audit` | ○ | [spec](../screens/SCR034_AuditLogSearch/spec.md) |
| SCR035_SyncQueue | SC-35 | Hàng chờ đồng bộ | atomic | 📴 | F010 | `/sync-queue` | ○ | [spec](../screens/SCR035_SyncQueue/spec.md) |
| SCR036_QualificationReview | SC-N1 | Xét duyệt / tạm dừng tư cách tham gia 🔒 | composite | | F002 | `/qualifications`, `/qualifications/[id]` | ○ | [spec](../screens/SCR036_QualificationReview/spec.md) |
| SCR037_MarketManagement | SC-N2 | Quản lý chợ & quy tắc vận hành 🔒 | composite | | F012 | `/markets`, `/markets/[id]/rules`, `/markets/[id]/rules/[rule_key]` | ○ | [spec](../screens/SCR037_MarketManagement/spec.md) |

Ký hiệu LAB-1: ⌨️ bàn phím bắt buộc · 📴 chạy khi mất mạng · 🔒 có bước duyệt maker-checker. Ghi chú: `SCR035` thuộc nhóm "F10 · Thông báo" ở registry (F010) dù LAB-1 gán tính năng `F03-06` `F05-07` (ghi khi mất mạng).

---

## SCR001_RoleHome
**Type**: atomic (4 biến thể theo role)
### Description
Trang sau đăng nhập: 3–5 thẻ số liệu "việc cần làm hôm nay" theo đúng chức danh (`receiving`/`trading`/`reconcile`/`sys_admin`). LAB-1 thiết kế 7 biến thể, LAB-3 chốt 4.
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| `StatCard` | card | một số liệu + nhãn (`app/(app)/home/stat-card.tsx`) |
| `ReceivingHome`/`TradingHome`/`ReconcileHome`/`AdminHome` | variant view | biến thể theo `user.role` (`app/(app)/page.tsx:24-70`) |
### Data Displayed
- Entities: `lots`, `trades`, `deliveries`, `business_days`, `app_users`, `role_change_log`, view `v_trade_delivery_status`, `v_lot_availability`
### Routes/URLs
- `/` (mở cho mọi role: `ROUTE_POLICY` entry `"/"` = `all`)
### Related Screens
- SCR002_Login (nguồn: đăng nhập thành công; đích khi chưa đăng nhập) · nav tới SCR003/007/008/010/011/012/014/018 theo role

## SCR002_Login
**Type**: atomic
### Description
Cổng vào duy nhất; email + mật khẩu qua Supabase Auth; hiện banner khi tài khoản bị vô hiệu hoá (`?reason=suspended`).
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| `LoginForm` | form | `useActionState(signIn)` (`app/login/login-form.tsx`) |
| banner `reason=suspended` | alert | cảnh báo tài khoản bị khoá (`app/login/page.tsx:26-30`) |
### Data Displayed
- Entities: `auth.users` (qua Supabase Auth), `app_users.status` (gián tiếp, do `proxy.ts`)
### Routes/URLs
- `/login`, `/login?reason=suspended` (route công khai duy nhất — `lib/supabase/proxy.ts:5`)
### Related Screens
- SCR001_RoleHome (thành công → `redirect("/")`) · mọi màn khác (chưa đăng nhập → redirect về đây)

## SCR003_AccountList
**Type**: atomic
### Description
Danh sách tài khoản nội bộ, lọc theo chức danh/trạng thái; một trong hai màn duy nhất `sys_admin` mở được.
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| `FilterBar` | GET form | 2 select lọc (`components/ui/filter-bar.tsx`) |
| `DataTable` | table | 5 cột, tên là link sang SCR004 |
### Data Displayed
- Entities: `app_users` (họ tên, email, chức danh, trạng thái, ngày tạo)
### Routes/URLs
- `/accounts` (query `?role=&status=`)
### Related Screens
- SCR004_AccountDetail (chi tiết — bấm tên) · SCR001_RoleHome (nav)

## SCR004_AccountDetail
**Type**: composite
### Description
Chi tiết một tài khoản: đổi chức danh qua RPC `change_user_role` và xem lịch sử đổi quyền. Màn duy nhất `sys_admin` ghi được.
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| `RoleChangeForm` | form | chọn chức danh mới + ghi chú (`app/(app)/accounts/[id]/role-change-form.tsx`) |
| `RoleHistoryTable` | table | quyền cũ → quyền mới, ai, lúc nào |
### Data Displayed
- Entities: `app_users`, `role_change_log`
### Routes/URLs
- `/accounts/[id]` (khớp prefix `/accounts`)
### Related Screens
- SCR003_AccountList (parent)
### Regions
| Code | Label | Owner | Independence Signals |
|------|-------|-------|---------------------|
| REG001 | RoleChangeForm | F001_AdminAndPermissions | ghi riêng `rpc change_user_role`; kiểm role 3 tầng |
| REG002 | RoleHistoryTable | F001_AdminAndPermissions | đọc riêng `role_change_log` + truy vấn thứ hai `app_users` |

## SCR005_ParticipantList
**Type**: atomic
### Description
Tra nhanh "người này còn được giao dịch không, theo căn cứ nào": lọc theo loại đối tác, căn cứ, trạng thái hiệu lực, sắp hết hạn. Badge `条件付き` ≠ `再審査`.
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| bộ lọc 7 ô | GET form | `q`, `party_type`, `basis_type`, `status`, `expiring`, `warning_days`, `market_id` |
| bảng 10 cột + badge 5 màu | table | `v_participant_qualification` |
### Data Displayed
- Entities: `participants`, `participant_qualifications`, `qualification_basis_rules`, `markets`, view đề xuất `v_participant_qualification`
### Routes/URLs
- `/participants` (đề xuất)
### Related Screens
- SCR006_ParticipantProfile (mở dòng / thêm mới) · SCR036_QualificationReview (đường tắt khi `条件付き`/`再審査`)

## SCR006_ParticipantProfile
**Type**: atomic (4 biến thể: `市場登録` `許可` `承認` `卸売場登録`)
### Description
Lập/sửa hồ sơ người tham gia kèm căn cứ; dựng đúng một trong bốn biến thể theo `party_type`; **không** đổi trạng thái tại đây.
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| khối chung #1–#10 | form | mã, tên, loại, liên hệ, chợ, số hiệu, hiệu lực |
| khối riêng A/B/C/D | variant form | cột khác nhau ở bảng khác nhau (`qualification_license`, `qualification_approval`, …) |
### Data Displayed
- Entities: `participants`, `participant_qualifications`, `qualification_license`/`qualification_approval` (đề xuất), `markets`, `market_rules`, `qualification_transitions`
### Routes/URLs
- `/participants/[id]`, `/participants/new?party_type=…` (đề xuất)
### Related Screens
- SCR005_ParticipantList (parent) · SCR036_QualificationReview (đề nghị chuyển trạng thái) · SCR037_MarketManagement (quy chế biến thể D)

## SCR007_LotEntry
**Type**: atomic · ⌨️ 📴
### Description
Nhân viên `receiving` ghi nhận lô vừa lên bờ (chủ hàng, mặt hàng, số lượng) liên tục từ 02:00 JST; keyboard-first; mất mạng không mất dữ liệu.
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| `LotEntryForm` + `useLotEntryForm` | controlled form | Tab/Enter/Ctrl+Enter/Escape (`app/(app)/lots/new/`) |
| `CodeInput` | `<input list>` | gợi ý mã chủ hàng/mặt hàng (datalist) |
### Data Displayed
- Entities: `participants` (wholesaler), `commodities`, `business_days`, `lots`
### Routes/URLs
- `/lots/new` (chỉ `receiving`; đứng trước `/lots` trong `ROUTE_POLICY`)
### Related Screens
- SCR008_LotList (parent; nút "Nhập lô mới") · SCR035_SyncQueue (đề xuất: ghi nháp khi offline)

## SCR008_LotList
**Type**: composite
### Description
Toàn bộ lô, 5 trạng thái, số đã bán/còn lại (từ view); `receiving` có nút "Mở bán", `trading` chỉ xem.
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| `LotFilter` + `DataTable` | filter + table | lọc trạng thái/ngày; 8 cột |
| `StatusAction` | row action | "Mở bán" qua `publishLot` (chỉ `receiving`) |
### Data Displayed
- Entities: `lots`, `v_lot_availability`, `participants`, `commodities`
### Routes/URLs
- `/lots` (`?status=&businessDate=`)
### Related Screens
- SCR007_LotEntry (nút "Nhập lô mới") · SCR009_LotDetail (đề xuất: bấm mã lô) · SCR010_NegotiatedTradeConfirm (tra lô trước khi chốt)
### Regions
| Code | Label | Owner | Independence Signals |
|------|-------|-------|---------------------|
| REG001 | Bộ lọc + bảng lô | F003_LotsAndInspection | đọc `lots` + `v_lot_availability` |
| REG002 | Nút "Mở bán" theo dòng | F003_LotsAndInspection | ghi riêng (UPDATE `lots.status` qua `publishLot`) |

## SCR009_LotDetail
**Type**: composite
### Description
Xem thuộc tính lô, ghi kết quả đánh giá cảm quan (目利き, chỉ ghi lại), điều chỉnh thuộc tính có kiểm soát (có lý do).
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| khối thông tin lô | read-only | 10 field + trạng thái ngày |
| form đánh giá + hộp thoại điều chỉnh | form + dialog | `record_lot_evaluation`, `adjust_lot_attributes` (RPC đề xuất) |
### Data Displayed
- Entities: `lots`, `v_lot_availability`, `lot_evaluations`, `lot_adjustments` (đề xuất), `business_days`
### Routes/URLs
- `/lots/[id]` (đề xuất; cần thêm `reconcile` vào policy `/lots`)
### Related Screens
- SCR008_LotList (parent) · SCR022_CorrectionRequestCreate (khi ngày đã chốt)
### Regions
| Code | Label | Owner | Independence Signals |
|------|-------|-------|---------------------|
| REG001 | Thông tin lô | F003_LotsAndInspection | đọc `lots` + view |
| REG002 | Đánh giá cảm quan + lịch sử | F003_LotsAndInspection | ghi riêng `rpc record_lot_evaluation` |
| REG003 | Hộp thoại điều chỉnh | F003_LotsAndInspection | ghi riêng `rpc adjust_lot_attributes`; validation riêng |

## SCR010_NegotiatedTradeConfirm
**Type**: composite · ⌨️
### Description
`trading` chốt giao dịch thỏa thuận (相対, ~90% khối lượng): cổng chống bán vượt lô — tư cách người mua + số lượng còn lại kiểm trong RPC một transaction.
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| `NegotiatedForm` + `useNegotiatedForm` | controlled form | 4 field, phím tắt Enter/Ctrl+Enter/Escape |
| `LotBuyerPreview` | info panel | tham khảo lô/người mua (không phải cổng) |
### Data Displayed
- Entities: `lots`, `v_lot_availability`, `commodities`, `participants`, `participant_qualifications`, `trades` (kết quả)
### Routes/URLs
- `/trades/negotiated` (chỉ `trading`)
### Related Screens
- SCR012_TradeList (xem lại giao dịch vừa chốt) · SCR008_LotList (tra lô) · SCR005_ParticipantList (đề xuất: tra tư cách trước khi chốt)
### Regions
| Code | Label | Owner | Independence Signals |
|------|-------|-------|---------------------|
| REG001 | Form chốt | F004_TradeRecording | ghi riêng `rpc confirm_negotiated_trade` |
| REG002 | Vùng tham khảo | F004_TradeRecording | endpoint riêng `lookupLotPreview`/`lookupBuyerPreview`; trạng thái tải riêng |

## SCR011_AuctionResultEntry
**Type**: atomic
### Description
Nhập tay kết quả phiên đấu giá đã diễn ra (6 field); cố ý không OCR/gợi ý (CAP-HUMAN).
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| `AuctionForm` + `useAuctionForm` | controlled form | 6 field, chống mất dữ liệu khi mất mạng |
| banner thành công/lỗi | alert | `role="status"` / `role="alert"` |
### Data Displayed
- Entities: `trades` (`trade_type='auction'`, `auction_session_at`, `auction_confirmed_by_name`), `lots`, `participants`
### Routes/URLs
- `/trades/auction` (chỉ `trading`)
### Related Screens
- SCR012_TradeList (xem lại) · SCR010_NegotiatedTradeConfirm (cùng RPC khung `_confirm_trade`)

## SCR012_TradeList
**Type**: atomic
### Description
Danh sách giao dịch (thỏa thuận + đấu giá), lọc 4 tiêu chí; `trading` kiểm lại, `reconcile` làm nguồn đối chiếu.
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| `TradeFilter` | GET form | ngày, mã người tham gia, trạng thái, loại |
| `DataTable` | table | 9 cột, `amount` là cột GENERATED |
### Data Displayed
- Entities: `trades` (embed `lots`, `participants`)
### Routes/URLs
- `/trades` (`?businessDate=&buyerCode=&status=&tradeType=`; `trading`+`reconcile`)
### Related Screens
- SCR013_TradeDetail (đề xuất: bấm mã giao dịch) · SCR010/SCR011 (nơi sinh giao dịch) · SCR018_DailyReconciliation (đối chiếu)

## SCR013_TradeDetail
**Type**: atomic
### Description
Xem đủ số liệu một giao dịch, huỷ khi ngày chưa chốt (kèm lý do), xem tab lịch sử sửa. Sau chốt sổ chỉ đi qua phiếu điều chỉnh.
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| khối số liệu + tiến độ giao hàng | read-only | link sang SCR016 |
| hộp thoại huỷ + tab lịch sử sửa | dialog + table | `cancel_trade` (RPC đề xuất) |
### Data Displayed
- Entities: `trades`, `v_trade_delivery_status`, `audit_log`/lịch sử sửa
### Routes/URLs
- `/trades/[id]` (đề xuất; khớp prefix `/trades` — không cần đổi policy)
### Related Screens
- SCR012_TradeList (parent) · SCR016_DeliveryDetail (xem các lần giao) · SCR022_CorrectionRequestCreate (sau chốt sổ)

## SCR014_DeliveryEntry
**Type**: atomic · ⌨️ 📴
### Description
`receiving` ghi một lần giao hàng cho một giao dịch (giao nhiều lần); thấy đã giao/còn lại trước khi nhập; ghi ngoại lệ cần lý do.
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| `DeliveryForm` + `useDeliveryForm` | controlled form | mã giao dịch, số lượng, trạng thái, loại ngoại lệ, lý do |
| `TradeProgress` | info panel | tiến độ giao hàng tham khảo |
### Data Displayed
- Entities: `v_trade_delivery_status`, `trades`, `lots`, `participants`, `commodities`, `deliveries`
### Routes/URLs
- `/deliveries/new` (chỉ `receiving`)
### Related Screens
- SCR015_DeliveryList (đề xuất) · SCR016_DeliveryDetail · SCR035_SyncQueue (offline)

## SCR015_DeliveryList
**Type**: atomic
### Description
Mỗi dòng một giao dịch: lũy kế đã giao, còn lại, số lần giao, ứ đọng; bộ lọc ứ đọng dùng chung định nghĩa với `RPT-04`.
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| bộ lọc (ngày mặc định hôm nay, "chỉ ứ đọng") | GET form | giữ trạng thái trên URL |
| bảng giao dịch | table | "Có ngoại lệ" lên đầu |
### Data Displayed
- Entities: `v_trade_delivery_status`, `trades`, `deliveries`
### Routes/URLs
- `/deliveries` (đề xuất; **hiện route này chuyển hướng sang `/deliveries/new`** — xem screen-flow)
### Related Screens
- SCR016_DeliveryDetail (bấm mã giao dịch) · SCR014_DeliveryEntry (nút "Ghi lần giao", chỉ `receiving`)

## SCR016_DeliveryDetail
**Type**: atomic
### Description
Từng lần giao của một giao dịch + lũy kế/còn lại; "Chốt hoàn tất giao hàng" kiểm quy tắc 精算 (`BR-DEL-03`) và tư cách người nhận.
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| khối lũy kế + bảng lần giao | read-only | sắp theo `seq` |
| hộp thoại chốt hoàn tất | dialog | checklist 3 dòng; `complete_trade_delivery` (RPC đề xuất) |
### Data Displayed
- Entities: `trades`, `deliveries`, `v_trade_delivery_status`, quy tắc 精算 (`market_rules`)
### Routes/URLs
- `/deliveries/trade/[tradeId]` (đề xuất)
### Related Screens
- SCR015_DeliveryList (parent) · SCR013_TradeDetail (đến từ) · SCR014_DeliveryEntry · SCR017_DeliveryException

## SCR017_DeliveryException
**Type**: atomic (lập phiếu maker + duyệt checker, 3 route)
### Description
Ghi 4 loại ngoại lệ (giao thiếu/thừa/trả hàng/huỷ một phần); `delta=0` tự áp dụng, `delta≠0` vào hàng duyệt của `reconcile` (maker≠checker).
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| phiếu lập (maker, `receiving`) | form | loại, lần giao liên quan, số thực tế, lý do |
| khối duyệt (checker, `reconcile`) | decision form | `decide_delivery_exception` (RPC đề xuất) |
### Data Displayed
- Entities: `delivery_exception_requests`, `deliveries`, `trades`
### Routes/URLs
- `/deliveries/exceptions/new`, `/deliveries/exceptions`, `/deliveries/exceptions/[id]` (đề xuất)
### Related Screens
- SCR016_DeliveryDetail (nơi bắt đầu) · SCR015_DeliveryList/SCR018 (số liệu đổi sau duyệt)

## SCR018_DailyReconciliation
**Type**: atomic
### Description
Bảng gom giao dịch + giao hàng + ngoại lệ theo ngày nghiệp vụ; `reconcile` chốt sổ ngày đang mở qua hộp thoại (điều kiện chưa đạt chỉ cảnh báo, ADR-005).
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| `ReconciliationFilter` + bảng | GET form + table | `v_daily_reconciliation` |
| `CloseDayDialog` | `<dialog>` | `getCloseConditionsAction` + RPC `close_business_day` |
### Data Displayed
- Entities: `v_daily_reconciliation`, `business_days`, `v_trade_delivery_status`
### Routes/URLs
- `/reconciliation` (`?businessDate=`; chỉ `reconcile`)
### Related Screens
- SCR012_TradeList · SCR019_DisputeList (đề xuất) · SCR022_CorrectionRequestCreate (sau chốt sổ) · SCR030_AccountingBatch (batch ~10:00)

## SCR019_DisputeList
**Type**: atomic
### Description
Danh sách tranh chấp đang mở/quá hạn; 3 tiêu chí lọc LAB-1 (đang mở, quá hạn, chưa có người xử lý); hai số tổng không đổi theo bộ lọc.
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| bộ lọc + 2 số tổng | GET form + counters | mặc định "đang mở", quá hạn lên đầu |
| bảng `disputes` | table | mỗi dòng một vụ |
### Data Displayed
- Entities: `disputes`, `app_users` (người xử lý)
### Routes/URLs
- `/disputes` (đề xuất)
### Related Screens
- SCR020_DisputeDetail (bấm dòng) · SCR017_DeliveryException/SCR018 (nơi sinh tranh chấp)

## SCR020_DisputeDetail
**Type**: atomic
### Description
Chi tiết vụ tranh chấp, cập nhật trạng thái qua máy 6 chuyển tiếp hợp lệ, lịch sử chỉ thêm; không sửa số liệu ngày đã chốt — dẫn sang phiếu điều chỉnh.
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| khối thông tin vụ việc + cập nhật | read-only + form | nhận xử lý, ghi chú, dời hạn, kết quả |
| lịch sử `dispute_events` | table | append-only |
### Data Displayed
- Entities: `disputes`, `dispute_events`, `correction_requests` (liên kết)
### Routes/URLs
- `/disputes/[id]` (đề xuất)
### Related Screens
- SCR019_DisputeList (parent) · SCR022_CorrectionRequestCreate (nhánh "Lập phiếu điều chỉnh"; spec cũ ghi `SC-23` — xem screen-flow § Điểm lệch) · SCR021_MarketCalendar (kiểm `due_date`)

## SCR021_MarketCalendar
**Type**: atomic
### Description
Lịch nghỉ chợ (休市日) và chuỗi 業務日 sinh từ lịch; chế độ nhập tay hoặc đồng bộ nguồn ngoài (chờ câu 16); lỗi đồng bộ phải sinh cảnh báo SCR033.
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| lưới 12 tháng + điều khiển | grid + controls | đặt/xoá 休市日, nguyên tử theo lô |
| chuỗi 業務日 dẫn xuất | read-only | `generated_through` |
### Data Displayed
- Entities: `market_holidays`, `business_days`, `markets`
### Routes/URLs
- `/markets/[id]/calendar` (đề xuất; dưới SCR037 vì lịch là quy tắc theo chợ — ADR-007)
### Related Screens
- SCR037_MarketManagement (parent) · SCR033_OpsAlertQueue (cảnh báo đồng bộ lỗi) · SCR020_DisputeDetail

## SCR022_CorrectionRequestCreate
**Type**: atomic
### Description
`reconcile` (maker) lập phiếu xin điều chỉnh số liệu đã chốt: chọn bản ghi đích, số trước (ảnh chụp), số sau (đề nghị), lý do + chứng cứ bắt buộc. Bản ghi gốc không đổi.
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| khối A/B/C | form | bản ghi đích / số trước / số sau |
| cảnh báo "không sửa số cũ" | alert | `US-16` |
### Data Displayed
- Entities: `correction_requests`, `trades`/`deliveries` (bản ghi đích), `business_days`
### Routes/URLs
- `/corrections/new?target=` (đề xuất)
### Related Screens
- SCR012_TradeList, SCR018_DailyReconciliation, SCR009_LotDetail, SCR013_TradeDetail (đường vào, từ `P0009`/`P0002`) · SCR023_PendingRequestList (sau khi gửi)

## SCR023_PendingRequestList
**Type**: atomic
### Description
Danh sách phiếu chờ duyệt kèm thời gian chờ; nút duyệt chỉ hiện cho người có cờ `can_approve` và không phải người lập.
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| bộ lọc `decision_state` | GET form | `pending`/`approved`/… |
| bảng phiếu | table | 3 nhánh theo người xem |
### Data Displayed
- Entities: `correction_requests`, `app_users`
### Routes/URLs
- `/corrections` (đề xuất)
### Related Screens
- SCR024_RequestDetailApproval (mở phiếu & duyệt) · SCR022_CorrectionRequestCreate

## SCR024_RequestDetailApproval
**Type**: composite 🔒
### Description
Người duyệt thấy so sánh trước/sau (đọc lại số hiện tại, không dùng snapshot cũ), duyệt/từ chối; duyệt sinh bản ghi điều chỉnh append-only, bản gốc nguyên vẹn.
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| khối A thông tin + B so sánh | read-only | |
| khối C quyết định + D bản ghi sinh ra | decision form + read-only | `decide_correction_request`; chặn maker=checker bằng `P0023` |
### Data Displayed
- Entities: `correction_requests`, `correction_entries`, `audit_log`
### Routes/URLs
- `/corrections/[id]` (đề xuất)
### Related Screens
- SCR023_PendingRequestList (parent) · SCR018_DailyReconciliation, SCR025_IncentiveResult (số kỳ mở cộng thêm dòng điều chỉnh)
### Regions
| Code | Label | Owner | Independence Signals |
|------|-------|-------|---------------------|
| REG001 | Thông tin + so sánh (A/B) | F007_PostCloseCorrection | đọc lại số hiện tại, endpoint riêng |
| REG002 | Quyết định (C) | F007_PostCloseCorrection | ghi riêng RPC duyệt; chỉ hiện khi `pending` và người xem đủ điều kiện |

## SCR025_IncentiveResult
**Type**: atomic
### Description
Kết quả tiền khuyến khích theo kỳ × người tham gia; `reconcile` chạy tính cho kỳ đang mở; mỗi dòng gắn phiên bản 料率 cụ thể (cột lưu, không join động).
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| bộ lọc kỳ + bảng | GET form + table | mỗi dòng = kỳ × người × loại dòng |
| bung chi tiết một dòng | detail | `/incentives/[period_id]/[participant_id]` |
### Data Displayed
- Entities: `incentive_results`, `incentive_rate_versions`, `participants`
### Routes/URLs
- `/incentives`, `/incentives/[period_id]/[participant_id]` (đề xuất)
### Related Screens
- SCR027_RateVersionApproval (bấm `v{n}` cột 適用料率版, chỉ đọc) · SCR024_RequestDetailApproval (điều chỉnh)

## SCR026_RateVersionList
**Type**: atomic
### Description
Danh sách phiên bản tỷ lệ khuyến khích, 6 trạng thái hiển thị dẫn xuất; "Lập bản mới" sao từ bản đang hiệu lực.
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| bộ lọc + bảng phiên bản | GET form + table | mỗi dòng một phiên bản |
| nút "Lập bản mới" | action | `rule_admin` |
### Data Displayed
- Entities: `incentive_rate_versions`
### Routes/URLs
- `/incentive-rates` (đề xuất)
### Related Screens
- SCR027_RateVersionApproval (lập mới/chi tiết/duyệt/quay lui) · SCR025_IncentiveResult

## SCR027_RateVersionApproval
**Type**: atomic 🔒
### Description
Lập bản nháp (maker), duyệt/từ chối (checker khác người lập), xem vết 起案者/承認者/発効日; thuế 110/100 và làm tròn xuống JPY không phải field.
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| khối lập (maker) | form | sửa được khi `draft` và là người lập |
| khối vết + đối chiếu | read-only | chỉ hiện đối chiếu khi `pending` |
### Data Displayed
- Entities: `incentive_rate_versions`, `audit_log`
### Routes/URLs
- `/incentive-rates/new`, `/incentive-rates/[version_id]` (đề xuất)
### Related Screens
- SCR026_RateVersionList (parent) · SCR025_IncentiveResult (đi chéo, `reconcile` chỉ xem)

## SCR028_ReportCatalog
**Type**: atomic
### Description
Danh mục 12 báo cáo (seed `report_catalog`), trạng thái lần chạy gần nhất; tham số nhập ở SCR029, không nhập ở đây.
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| bảng catalog | table | lọc theo `allowed_roles` |
| badge lỗi + link "Xem cảnh báo" | badge | sang SCR030 |
### Data Displayed
- Entities: `report_catalog`, `report_runs`
### Routes/URLs
- `/reports` (đề xuất; `reconcile`, `market_manager`)
### Related Screens
- SCR029_ReportViewExport ("Mở báo cáo") · SCR030_AccountingBatch

## SCR029_ReportViewExport
**Type**: atomic
### Description
Xem một báo cáo theo tham số dựng từ `param_schema`, xuất CSV (4 dòng siêu dữ liệu); luôn truyền `as_of` tường minh (chống `now()`).
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| khối tham số | dynamic form | field động theo `report_code` |
| khối kết quả + hành động | table + action | xuất CSV; >50.000 dòng → hàng đợi nền |
### Data Displayed
- Entities: `report_catalog`, `report_runs`, dữ liệu theo từng báo cáo
### Routes/URLs
- `/reports/[report_code]` (ví dụ `/reports/RPT-05?businessDate=2026-03-04`; đề xuất)
### Related Screens
- SCR028_ReportCatalog (parent) · SCR030_AccountingBatch (RPT-06 cùng số liệu)

## SCR030_AccountingBatch
**Type**: composite
### Description
Trạng thái batch báo cáo ngày và lô xuất kế toán; chạy lại (`attempt<3`), chi tiết dòng 6 trường tối thiểu (`IF-ACC-01`); sai sau khi gửi → phiếu điều chỉnh (lô `delta`).
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| tab A batch báo cáo ngày | table | `batch_runs` |
| tab B lô xuất kế toán + chi tiết dòng | table | `acc_export_batches`, `acc_export_lines` |
### Data Displayed
- Entities: `batch_runs`, `acc_export_batches`, `acc_export_lines`, `ops_alerts`
### Routes/URLs
- `/operations/batches`, `/operations/acc-exports`, `/operations/acc-exports/[batch_code]` (đề xuất)
### Related Screens
- SCR018_DailyReconciliation (kích hoạt batch ~10:00) · SCR033_OpsAlertQueue (cảnh báo) · SCR022_CorrectionRequestCreate · SCR029 (RPT-06)
### Regions
| Code | Label | Owner | Independence Signals |
|------|-------|-------|---------------------|
| REG001 | Tab batch báo cáo ngày | F009_ReportsAndExport | endpoint riêng `batch_runs`; retry riêng |
| REG002 | Tab lô xuất kế toán | F009_ReportsAndExport | endpoint riêng `acc_export_batches`; tạo lô/retry riêng |

## SCR031_NotificationInbox
**Type**: atomic
### Description
Hộp thông báo cá nhân (2 kênh: trong ứng dụng + email; không SMS/FAX); bấm một dòng điều hướng tới đối tượng và đánh dấu đã đọc.
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| danh sách + bộ lọc | list | mỗi dòng một `notifications` |
| badge số chưa đọc | badge | hiện ở SCR001/nav |
### Data Displayed
- Entities: `notifications`
### Routes/URLs
- `/notifications` (đề xuất; mọi role)
### Related Screens
- SCR001_RoleHome (nguồn badge) · SCR020_DisputeDetail và các đối tượng khác theo `subject_ref` · SCR032_NotificationSettings

## SCR032_NotificationSettings
**Type**: atomic
### Description
Đặt người nhận, ngưỡng, thời điểm gửi; đổi cấu hình không làm mất lịch sử đã gửi (`F10-05`).
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| form cấu hình | form | người nhận, ngưỡng, `digest_at` |
| banner xác nhận | alert | nêu rõ thông báo đang chờ không bị ảnh hưởng |
### Data Displayed
- Entities: `notification_settings`
### Routes/URLs
- `/notifications/settings` (đề xuất; phải khai báo **trước** `/notifications` trong `ROUTE_POLICY`)
### Related Screens
- SCR031_NotificationInbox (parent) · SCR033_OpsAlertQueue

## SCR033_OpsAlertQueue
**Type**: atomic
### Description
Hàng cảnh báo vận hành (email lỗi quá 3 lần, batch/xuất kế toán lỗi); chỉ là cửa sổ cảnh báo, không nhân đôi chức năng retry của SCR030.
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| danh sách + lịch sử từng lần thử | list + sub-table | `ops_alerts` |
| bộ lọc | filter | nguồn, trạng thái |
### Data Displayed
- Entities: `ops_alerts`, lịch sử lần thử
### Routes/URLs
- `/ops/alerts` (đề xuất)
### Related Screens
- SCR030_AccountingBatch ("Mở màn batch") · SCR021_MarketCalendar (đồng bộ lịch lỗi) · SCR032_NotificationSettings

## SCR034_AuditLogSearch
**Type**: atomic
### Description
Tra vết theo 4 chiều (mã giao dịch, ngày nghiệp vụ, người tham gia, loại thao tác), mỗi dòng đủ 5 trường (`FR-AUDIT-01`); tra lại giao dịch cũ trong 2 phút.
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| bộ lọc 4 chiều (+ chiều bổ sung đề xuất) | form | |
| bảng kết quả + mở rộng JSON | table | `old_value`/`new_value` |
### Data Displayed
- Entities: `audit_log` (cần thêm cột `reason` và nới RLS)
### Routes/URLs
- `/audit` (đề xuất; `sys_admin`, `internal_audit`, `reconcile`)
### Related Screens
- SCR009_LotDetail / SCR013_TradeDetail / SCR016_DeliveryDetail (liên kết sang bản ghi gốc)

## SCR035_SyncQueue
**Type**: atomic · 📴
### Description
Hàng chờ đồng bộ của `receiving` khi mất mạng (SCR007/SCR014): trạng thái từng bản ghi, retry; ngày đã chốt → `needs_correction` mở phiếu điều chỉnh với `clientGeneratedId`.
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| thanh trạng thái đầu trang | status bar | `pendingCount` |
| hàng đợi + thao tác dòng | table | IndexedDB phía client |
### Data Displayed
- Entities: hàng đợi cục bộ (IndexedDB), `lots`/`deliveries` sau đồng bộ
### Routes/URLs
- `/sync-queue` (đề xuất; chỉ `receiving`)
### Related Screens
- SCR007_LotEntry, SCR014_DeliveryEntry (nguồn) · SCR022_CorrectionRequestCreate (ngày đã chốt)

## SCR036_QualificationReview
**Type**: composite 🔒
### Description
Maker lập phiếu chuyển trạng thái tư cách (`許可`: `条件付き`; `承認`: `再審査`), checker khác người lập duyệt; hai máy trạng thái vẽ riêng (D-01).
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| danh sách phiếu | list | lọc `pending` |
| phiếu: khối lập (maker) + khối duyệt (checker) | form + decision | `V-05` ẩn khối duyệt khi maker=checker |
### Data Displayed
- Entities: `qualification_transitions`, `qualification_license`, `qualification_approval`, `participants`
### Routes/URLs
- `/qualifications`, `/qualifications/[id]` (đề xuất)
### Related Screens
- SCR005_ParticipantList, SCR006_ParticipantProfile (đường vào) · SCR010/SCR011 (hệ quả `P0010`/`P0011` từ `effective_on`)
### Regions
| Code | Label | Owner | Independence Signals |
|------|-------|-------|---------------------|
| REG001 | Danh sách phiếu | F002_ParticipantsAndQualification | đọc riêng, lọc `pending` |
| REG002 | Phiếu: lập + duyệt | F002_ParticipantsAndQualification | ghi riêng RPC lập/duyệt; validation riêng |

## SCR037_MarketManagement
**Type**: composite 🔒
### Description
Quản lý chợ và quy tắc vận hành có phiên bản theo chợ (5 `rule_key`), maker-checker, trần 4 lần đổi/năm, quay lui = tạo bản mới (D-02, ADR-007).
### Components
| Component | Type | Purpose |
|-----------|------|---------|
| khối chợ + danh mục `rule_key` | list | `markets` |
| bảng phiên bản + form lập/duyệt + quay lui | table + form | `market_rules` |
### Data Displayed
- Entities: `markets`, `market_rules`
### Routes/URLs
- `/markets`, `/markets/[id]/rules`, `/markets/[id]/rules/[rule_key]` (đề xuất)
### Related Screens
- SCR021_MarketCalendar (`/markets/[id]/calendar`) · SCR005/SCR006 (đọc quy chế, ngưỡng cảnh báo) · SCR025/SCR026
### Regions
| Code | Label | Owner | Independence Signals |
|------|-------|-------|---------------------|
| REG001 | Chợ + danh mục quy tắc | F012_MarketManagement | đọc `markets` |
| REG002 | Phiên bản quy tắc (lập/duyệt/quay lui) | F012_MarketManagement | ghi riêng RPC phiên bản; validation riêng (`P0015`) |

---

## Summary

- **Total Screens**: 37 (11 đã xây ● : SCR001, 002, 003, 004, 007, 008, 010, 011, 012, 014, 018 · 26 thiết kế ○ — trong đó 24 màn LAB-1 + 2 màn mới LAB-4)
- **Composite**: SCR004, SCR008, SCR009, SCR010, SCR024, SCR030, SCR036, SCR037 · **Atomic**: 29 còn lại (SCR001, SCR006 có biến thể)
- **Keyboard (⌨️)**: SCR007, SCR010, SCR014 · **Offline (📴)**: SCR007, SCR014, SCR035 · **Maker-checker (🔒)**: SCR017, SCR022, SCR024, SCR027, SCR036, SCR037

---

## Cross-Reference Validation

- [x] All SCR### codes are unique (SCR001–SCR037, khớp bảng mã màn của LAB-4)
- [x] All SCR### codes are referenced in [screen-flow.md](./screen-flow.md)
- [x] All related screen references are valid
- [x] All route URLs are properly formatted (route ○ ghi rõ là đề xuất)
- [ ] All SCR### codes are referenced in [feature-list.md](./feature-list.md) — kiểm bởi agent sở hữu file đó
- [x] No orphaned screen references
