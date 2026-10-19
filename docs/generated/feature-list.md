# Feature List

**Project**: Product A — 水産物卸売市場 業務支援システム (hệ thống hỗ trợ nghiệp vụ chợ đầu mối thủy sản)
**Generated**: 2026-10-06
**Analysis Scope**: Bản đầy đủ theo LAB-1 — 12 feature (F001–F012), 63 tính năng con LAB-1 (F01-01 … F11-04) + 4 tính năng đề xuất mới của F012. Prototype LAB-3 (Next.js 16 + Supabase) đã xây 11/37 màn; phần còn lại là **đặc tả**.

**Code Format**: `F###_NameSlug` (khớp tên thư mục trong [features/](../features/)). Tính năng con giữ **mã LAB-1** (`F01-01` … `F11-04`).
**Screen Code Format**: `SCR###_NameSlug` (khớp [screens/](../screens/)); alias LAB-1 `SC-xx`.
**User Story Code Format**: giữ mã LAB-1 `US-01` … `US-20` (không cấp mã `US###` mới).
**Background Logic / Permission Code**: registry chưa cấp mã `BL###`/`PERM###` cho dự án này — các mục tương ứng ở dưới liệt kê bằng **tên** (trigger, RPC, chức danh), không bịa mã. Ma trận quyền ở [permissions.md](../system/permissions.md).
**Ký hiệu trạng thái**: **Đã xây** (chạy được đầu-cuối trên dữ liệu thật ở LAB-3; lỗi đã biết ghi riêng ở B-01…B-05 và chú thích cạnh dòng tương ứng — không loại khỏi số "đã xây") · **Một phần** (có xương, thiếu thịt — nêu rõ thiếu gì) · **Chỉ đặc tả** (thiết kế LAB-4, chưa có code). **(đề xuất)** = suy luận ngoài nguồn khách.

**Feature Types**:
- `ui` — có màn hình (SCR###)
- `background` — chỉ có logic nền
- `mixed` — có cả màn hình và logic nền

## Feature Hierarchy

Sắp theo ưu tiên P0 → P1. Ưu tiên của feature = mức cao nhất trong các tính năng con (LAB-1: P0/P1; `NFR` là tính năng sinh từ yêu cầu phi chức năng, không gán mức P).

| Code | Name | Type | Language | Workspace | Priority |
|------|------|------|----------|-----------|----------|
| F001_AdminAndPermissions | Quản trị & phân quyền (F01) | ui | TypeScript · SQL | `app/` · `lib/` · `supabase/migrations/` | P0 |
| F002_ParticipantsAndQualification | Người tham gia & tư cách (F02) | mixed | TypeScript · SQL | `app/` · `lib/` · `supabase/migrations/` | P0 |
| F003_LotsAndInspection | Lô hàng & đánh giá (F03) | ui | TypeScript · SQL | `app/` · `lib/` · `supabase/migrations/` | P0 |
| F004_TradeRecording | Ghi nhận giao dịch (F04) | ui | TypeScript · SQL | `app/` · `lib/` · `supabase/migrations/` | P0 |
| F005_DeliveryTracking | Theo dõi giao hàng (F05) | ui | TypeScript · SQL | `app/` · `lib/` · `supabase/migrations/` | P0 |
| F006_ReconciliationAndClosing | Đối chiếu & chốt kỳ (F06) | ui | TypeScript · SQL | `app/` · `lib/` · `supabase/migrations/` | P0 |
| F007_PostCloseCorrection | Điều chỉnh sau chốt (F07) | ui | TypeScript · SQL | `app/` · `lib/` · `supabase/migrations/` | P0 |
| F008_IncentiveAndRateVersion | Tiền khuyến khích & version quy chế (F08) | mixed | TypeScript · SQL | `app/` · `lib/` · `supabase/migrations/` | P0 |
| F009_ReportsAndExport | Báo cáo & xuất dữ liệu (F09) | mixed | TypeScript · SQL | `app/` · `lib/` · `supabase/migrations/` | P0 |
| F011_AuditLog | Nhật ký kiểm toán (F11) | mixed | TypeScript · SQL | `app/` · `lib/` · `supabase/migrations/` | P0 |
| F012_MarketManagement | Quản lý chợ & quy tắc vận hành (mới, D-02) | ui | TypeScript · SQL | `app/` · `lib/` · `supabase/migrations/` | P0 |
| F010_Notifications | Thông báo (F10) | mixed | TypeScript · SQL | `app/` · `lib/` · `supabase/migrations/` | P1 |

Cột Workspace: toàn bộ nằm trong một repo (Next.js App Router + Postgres/Supabase). Thư mục nguồn của phần **đã xây** liệt kê ở từng mục bên dưới; phần **chỉ đặc tả** chưa có tệp nào.

### Trạng thái triển khai theo feature

| Code | Tính năng con LAB-1 | Đã xây | Một phần | Chỉ đặc tả | Màn đã xây / tổng |
|------|:-------------------:|:------:|:--------:|:----------:|:-----------------:|
| F001 | 5 | 1 | 3 | 1 | 3 / 3 |
| F002 | 7 | 1 | 0 | 6 | 0 / 3 |
| F003 | 6 | 1 | 2 | 3 | 2 / 3 |
| F004 | 7 | 4 | 2 | 1 | 3 / 4 |
| F005 | 7 | 2 | 3 | 2 | 1 / 4 |
| F006 | 5 | 2 | 1 | 2 | 1 / 4 |
| F007 | 5 | 0 | 0 | 5 | 0 / 3 |
| F008 | 6 | 0 | 0 | 6 | 0 / 3 |
| F009 | 6 | 0 | 0 | 6 | 0 / 3 |
| F010 | 5 | 0 | 0 | 5 | 1 / 5 |
| F011 | 4 | 0 | 1 | 3 | 0 / 1 |
| F012 | 4 *(đề xuất)* | 0 | 0 | 4 | 0 / 1 |
| **Tổng** | **63 + 4** | **11** | **12** | **40 + 4** | **11 / 37** |

## Feature Details

### F001_AdminAndPermissions: Quản trị & phân quyền

**Type**: ui
**Description**: Đăng nhập tài khoản nội bộ, gán chức danh, đổi quyền có lưu vết, chặn route theo chức danh. `sys_admin` cố ý không mở được màn nghiệp vụ. Tài liệu: [business-context](../features/F001_AdminAndPermissions/business-context.md) · [technical-spec](../features/F001_AdminAndPermissions/technical-spec.md) · [screens](../features/F001_AdminAndPermissions/screens.md) · [edge-cases](../features/F001_AdminAndPermissions/edge-cases.md).

**Workspace**: `proxy.ts`, `lib/supabase/proxy.ts`, `lib/auth/`, `app/login/`, `app/(app)/accounts/`, `supabase/migrations/0002`, `0009`, `0012`
**Languages**: TypeScript · SQL
**Components**: 5 tính năng con LAB-1

| Mã LAB-1 | Tính năng | Ưu tiên | Màn | Trạng thái |
|---|---|---|---|---|
| F01-01 | Đăng nhập và gán chức danh | P0 | SCR002 | Đã xây — 4 chức danh (LAB-1 nêu 7; sai lệch có ý thức S2); thiết kế dùng 9 role key + cờ `can_approve` trên tài khoản (đã chốt PREMISE C-09) |
| F01-02 | Xác thực hai lớp cho quản trị và người duyệt | NFR | SCR002 | Chỉ đặc tả (MOCK-1) |
| F01-03 | Quản lý tài khoản nội bộ | P0 | SCR003, SCR004 | Một phần — xem, lọc, đổi chức danh; **thiếu** tạo, sửa, khoá, mở lại |
| F01-04 | Cấp, thu, tạm dừng, mở lại quyền — có lưu vết | P0 | SCR004 | Một phần — đổi chức danh + `role_change_log`; thiếu tạm dừng/mở lại |
| F01-05 | Bảo vệ phiên làm việc | NFR | — (hành vi nền) | Một phần — chặn tài khoản `suspended`; thiếu timeout, khoá tạm, log bất thường |

**Related Screens**:
- SCR002_Login: [spec](../screens/SCR002_Login/spec.md) · đã xây · `/login`
- SCR003_AccountList: [spec](../screens/SCR003_AccountList/spec.md) · đã xây · `/accounts`
- SCR004_AccountDetail: [spec](../screens/SCR004_AccountDetail/spec.md) · đã xây · `/accounts/[id]`
- Trang `/403` (ngoài 37 màn): đích của lớp chặn route

**Related User Stories**: không có — LAB-1 không viết US cho nhóm F01.

**Related APIs/Routes**:
- (GET) `/login`, `/403`, `/accounts`, `/accounts/[id]` — đã xây
- Server Action `signIn`, `signOut`, `changeRole` — đã xây
- RPC `change_user_role(p_user_id, p_new_role, p_note)`; hàm đọc chức danh `current_app_role()` — đã xây

**Related Data Models**: `app_users`, `role_change_log`, `audit_log` (ghi thêm một dòng mỗi lần đổi quyền)

**Related Background Logic**: lớp chặn route `proxy.ts` (kiểm phiên + `app_users.status` + `ROUTE_POLICY`); RLS ba bảng trên.

**Related Permissions**: `sys_admin` (xem/ghi), 3 chức danh còn lại chỉ đăng nhập. Chức danh đề xuất `party_admin`, `rule_admin`, `ops_admin`, `market_manager`, `internal_audit` *(đề xuất — chưa có trong DB)*; số chức danh và cờ `can_approve` trên tài khoản đã chốt (PREMISE C-09); `internal_audit` là role riêng, không dùng cờ `can_audit`.

---

### F002_ParticipantsAndQualification: Người tham gia & tư cách tham gia

**Type**: mixed
**Description**: Hồ sơ người tham gia theo bốn căn cứ (đăng ký chợ · giấy phép · chấp thuận · đăng ký tại điểm giao dịch), trạng thái hiệu lực, kiểm tư cách lúc chốt. **LAB-3 chỉ có phép kiểm** (`_assert_buyer_qualified`) và dữ liệu seed; hai máy trạng thái `許可`/`承認` tách riêng là thiết kế (D-01, ADR-006). Tài liệu: [business-context](../features/F002_ParticipantsAndQualification/business-context.md) · [technical-spec](../features/F002_ParticipantsAndQualification/technical-spec.md) · [screens](../features/F002_ParticipantsAndQualification/screens.md) · [edge-cases](../features/F002_ParticipantsAndQualification/edge-cases.md).

**Workspace**: `supabase/migrations/0003`, `0007`, `0031`; hiển thị lỗi ở `lib/domain/trade-preview.ts`
**Languages**: TypeScript · SQL
**Components**: 7 tính năng con LAB-1

| Mã LAB-1 | Tính năng | Ưu tiên | Màn | Trạng thái |
|---|---|---|---|---|
| F02-01 | Hồ sơ đơn vị bán buôn (căn cứ đăng ký chợ) | P0 | SCR006 (biến thể A) | Chỉ đặc tả (dữ liệu seed) |
| F02-02 | Hồ sơ đơn vị trung gian (giấy phép) | P0 | SCR006 (biến thể B) | Chỉ đặc tả (dữ liệu seed) |
| F02-03 | Hồ sơ người tham gia mua bán được chấp thuận (chấp thuận) | P0 | SCR006 (biến thể C) | Chỉ đặc tả (dữ liệu seed) |
| F02-04 | Hồ sơ người mua tại chợ (đăng ký tại điểm giao dịch) | P0 | SCR006 (biến thể D) | Chỉ đặc tả (dữ liệu seed) |
| F02-05 | Quản lý trạng thái hiệu lực (FIG-010) | P0 | SCR005, SCR006, SCR036 | Chỉ đặc tả — enum 3 giá trị chung hiện có, chưa có đường đổi trạng thái |
| F02-06 | Kiểm tư cách tại thời điểm chốt | P0 | — (hành vi nền; hiện ở SCR010/011/014) | Đã xây — kiểm đồng nhất, chưa rẽ nhánh theo căn cứ |
| F02-07 | Cảnh báo hồ sơ gần hết hạn | P1 | SCR005 | Chỉ đặc tả |

**Related Screens**:
- SCR005_ParticipantList: [spec](../screens/SCR005_ParticipantList/spec.md) · chỉ đặc tả · `/participants` *(đề xuất)*
- SCR006_ParticipantProfile: [spec](../screens/SCR006_ParticipantProfile/spec.md) · chỉ đặc tả · 4 biến thể
- SCR036_QualificationReview: [spec](../screens/SCR036_QualificationReview/spec.md) · chỉ đặc tả · màn mới ngoài LAB-1 (D-01)

**Related User Stories**: US-04 (F02-06)

**Related APIs/Routes**:
- RPC nội bộ `_assert_buyer_qualified(p_participant_id, p_as_of)` — đã xây (không gọi được từ ngoài)
- RPC `upsert_participant_profile`, `request_qualification_transition`, `decide_qualification_transition`, `cancel_qualification_transition`, `list_allowed_transitions` *(đề xuất)*
- Route `/participants`, `/participants/[id]`, `/participants/new`, `/qualifications`, `/qualifications/[id]` *(đề xuất)*

**Related Data Models**: `participants`, `participant_qualifications`, `qualification_basis_rules`; đề xuất `qualification_license`, `qualification_approval`, `qualification_transitions`

**Related Background Logic**: phép kiểm tư cách trong `_confirm_trade` và `record_delivery`; (đề xuất) tự chuyển `失効` khi quá ngày hết hạn.

**Related Permissions**: `party_admin` (ghi, lập phiếu) *(đề xuất)*; cờ `app_users.can_approve` cho người duyệt *(đề xuất)*; `reconcile`, `trading` chỉ xem *(đề xuất)*.

---

### F003_LotsAndInspection: Lô hàng & kết quả đánh giá

**Type**: ui
**Description**: Tiếp nhận lô, vòng đời 5 trạng thái (FIG-011), đánh giá cảm quan do người nhập, số lượng khả dụng không âm. Mở đầu [FL-01](../flows/FL-01-lot-trade-delivery-reconciliation.md). Tài liệu: [business-context](../features/F003_LotsAndInspection/business-context.md) · [technical-spec](../features/F003_LotsAndInspection/technical-spec.md) · [screens](../features/F003_LotsAndInspection/screens.md) · [edge-cases](../features/F003_LotsAndInspection/edge-cases.md).

**Workspace**: `app/(app)/lots/`, `lib/domain/lots.ts`, `supabase/migrations/0004`, `0006`, `0020`, `0021`
**Languages**: TypeScript · SQL
**Components**: 6 tính năng con LAB-1

| Mã LAB-1 | Tính năng | Ưu tiên | Màn | Trạng thái |
|---|---|---|---|---|
| F03-01 | Tiếp nhận lô hàng | P0 | SCR007 ⌨️ | Một phần — sinh mã lô, chủ hàng, mặt hàng, số lượng chạy thật; "đính kèm chứng từ" mới là ô số tham chiếu, chưa tải tệp |
| F03-02 | Ghi kết quả đánh giá cảm quan | P0 | SCR009 | Chỉ đặc tả — `lot_status` có `下見済` nhưng không có đường nhập |
| F03-03 | Quản lý trạng thái lô (FIG-011) | P0 | SCR008, SCR009 | Một phần — `受付/下見済 → 公開` (nút Mở bán), `確定`, `引渡し完了` tự chuyển (sai cấp, B-02: thiết kế chuyển khi mọi giao dịch `confirmed` của lô đã hoàn tất, PREMISE G3); thiếu `下見済` |
| F03-04 | Bảo đảm số lượng khả dụng không âm | P0 | — (hành vi nền; hiện ở SCR010) | Đã xây — view dẫn xuất + `P0004` (số khả dụng tính từ `trades`, nên B-02 chỉ chạm trạng thái lô, không chạm công thức); khoảng trống: cột `quantity` sửa trực tiếp được |
| F03-05 | Điều chỉnh thuộc tính lô có kiểm soát | P1 | SCR009 | Chỉ đặc tả |
| F03-06 | Ghi nháp lô khi mất mạng | NFR | SCR035 (+SCR007) | Chỉ đặc tả (MOCK-2) |

**Related Screens**:
- SCR007_LotEntry: [spec](../screens/SCR007_LotEntry/spec.md) · đã xây · `/lots/new`
- SCR008_LotList: [spec](../screens/SCR008_LotList/spec.md) · đã xây · `/lots`
- SCR009_LotDetail: [spec](../screens/SCR009_LotDetail/spec.md) · chỉ đặc tả · `/lots/[id]` *(đề xuất)*

**Related User Stories**: US-01, US-02, US-05, US-09 (dùng chung F005)

**Related APIs/Routes**:
- (GET) `/lots`, `/lots/new` — đã xây; Server Action `createLot`, `publishLot`
- RPC `next_lot_code(p_date)` — đã xây
- RPC `record_lot_evaluation`, `adjust_lot_attributes`, `sync_queued_lot` *(đề xuất)*

**Related Data Models**: `lots`, `business_days`, `commodities`, `participants`; view `v_lot_availability`; đề xuất `lot_evaluations`

**Related Background Logic**: trigger `guard_closed_day_lots`; sequence `lot_code_seq`.

**Related Permissions**: `receiving` (ghi), `trading` (xem); `reconcile` (đề xuất, chưa có quyền đọc `lots`).

---

### F004_TradeRecording: Ghi nhận giao dịch

**Type**: ui
**Description**: Giao dịch thỏa thuận 相対 (~90% giá trị) và kết quả đấu giá せり nhập tay (~10%); cổng chốt kiểm đồng thời tư cách người mua và số lượng lô trong một transaction. Tài liệu: [business-context](../features/F004_TradeRecording/business-context.md) · [technical-spec](../features/F004_TradeRecording/technical-spec.md) · [screens](../features/F004_TradeRecording/screens.md) · [edge-cases](../features/F004_TradeRecording/edge-cases.md).

**Workspace**: `app/(app)/trades/`, `lib/domain/trades.ts`, `trade-preview.ts`, `rpc-errors.ts`, `supabase/migrations/0005`, `0007`, `0030`–`0035`, `0041`
**Languages**: TypeScript · SQL
**Components**: 7 tính năng con LAB-1

| Mã LAB-1 | Tính năng | Ưu tiên | Màn | Trạng thái |
|---|---|---|---|---|
| F04-01 | Tạo giao dịch thỏa thuận trực tiếp | P0 | SCR010 ⌨️ | Đã xây — tạo và chốt là một bước |
| F04-02 | Cổng chốt giao dịch | P0 | SCR010 | Đã xây |
| F04-03 | Hủy giao dịch chưa chốt sổ | P1 | SCR013 | Chỉ đặc tả — enum có `cancelled`, không có RPC huỷ |
| F04-04 | Quản lý trạng thái giao dịch (FIG-012) | P0 | SCR012, SCR013 | Một phần — 2 trạng thái (`confirmed`, `cancelled`), LAB-1 mô tả 5 |
| F04-05 | Ghi kết quả phiên đấu giá | P0 | SCR011 | Đã xây — nhập tay, không đọc ảnh |
| F04-06 | Tra cứu bản ghi đấu giá và lịch sử sửa | P1 | SCR012, SCR013 | Một phần — SCR012 lọc được giao dịch せり; chưa hiện chi tiết phiên, chưa có lịch sử sửa (đặc tả ở SCR013) |
| F04-07 | Tra cứu và lọc giao dịch | P1 | SCR012 | Đã xây |

**Related Screens**:
- SCR010_NegotiatedTradeConfirm: [spec](../screens/SCR010_NegotiatedTradeConfirm/spec.md) · đã xây · `/trades/negotiated`
- SCR011_AuctionResultEntry: [spec](../screens/SCR011_AuctionResultEntry/spec.md) · đã xây · `/trades/auction`
- SCR012_TradeList: [spec](../screens/SCR012_TradeList/spec.md) · đã xây · `/trades`
- SCR013_TradeDetail: [spec](../screens/SCR013_TradeDetail/spec.md) · chỉ đặc tả · `/trades/[id]` *(đề xuất)*

**Related User Stories**: US-03, US-04 (F02), US-05 (F03), US-06

**Related APIs/Routes**:
- (GET) `/trades`, `/trades/negotiated`, `/trades/auction` — đã xây; Server Action `confirmTrade`, `recordAuctionResult`
- RPC `confirm_negotiated_trade(p_lot_id, p_buyer_id, p_quantity, p_unit_price)`, `record_auction_result` (6 tham số), `next_trade_code(p_date)` — đã xây
- RPC `cancel_trade` *(đề xuất)*

**Related Data Models**: `trades`, `lots`, `participant_qualifications`, `business_days`; view `v_lot_availability`

**Related Background Logic**: `_confirm_trade` (nội bộ); trigger `guard_closed_day_trades`; constraint trigger `trg_auction_fields_complete` (`P0008`).

**Related Permissions**: `trading` (ghi), `reconcile` (xem danh sách); `receiving` không mở được route `/trades` dù RLS đọc được (D-05).

---

### F005_DeliveryTracking: Theo dõi giao hàng

**Type**: ui
**Description**: Ghi từng lần giao của một giao dịch, lũy kế + số còn lại, ngoại lệ, chốt hoàn tất. Ba lỗi số liệu đã phát hiện ở tầng giao hàng (B-01, B-02 đã xác minh; B-04 mới đọc code) và B-05 (view đối chiếu, đã xác nhận trên DB thật, xem F06-01). Tài liệu: [business-context](../features/F005_DeliveryTracking/business-context.md) · [technical-spec](../features/F005_DeliveryTracking/technical-spec.md) · [screens](../features/F005_DeliveryTracking/screens.md) · [edge-cases](../features/F005_DeliveryTracking/edge-cases.md).

**Workspace**: `app/(app)/deliveries/`, `lib/domain/deliveries.ts`, `supabase/migrations/0005`, `0006`, `0007`, `0040`
**Languages**: TypeScript · SQL
**Components**: 7 tính năng con LAB-1

| Mã LAB-1 | Tính năng | Ưu tiên | Màn | Trạng thái |
|---|---|---|---|---|
| F05-01 | Ghi từng lần giao cho một giao dịch | P1 | SCR014 ⌨️ | Đã xây |
| F05-02 | Quản lý trạng thái giao hàng | P0 | SCR014, SCR016 | Một phần — chọn 1 trong 4 trạng thái lúc ghi; không có đường cập nhật sau |
| F05-03 | Tính lũy kế và số còn lại | P1 | SCR014, SCR016 | Đã xây — cảnh báo chia nhiều lần: ngưỡng 3 là giả định; lỗi đã biết **B-01** (`returned`/`partial_cancel` bị cộng vào "đã giao") và B-04 làm số lũy kế/trạng thái sai trong ca ngoại lệ |
| F05-04 | Chốt hoàn tất giao hàng | P0 | SCR016 | Một phần — `record_delivery` đã kiểm tư cách người nhận và chặn giao vượt (`P0006`); chưa có thao tác chốt hoàn tất riêng (`P0063`, SCR016); cấp hoàn tất đã chốt là **giao dịch** (`trades.delivery_completed_at`, PREMISE G3); BR-DEL-03 chưa định nghĩa "khớp" (khách chưa trả lời) |
| F05-05 | Ghi ngoại lệ giao hàng | P1 | SCR014, SCR017 | Một phần — ghi thẳng kèm lý do, **chưa** có người xác nhận nhập tay và bước duyệt; `over` không ghi được |
| F05-06 | Liên kết giao hàng với giao dịch và đối chiếu | P1 | SCR015, SCR016 | Chỉ đặc tả |
| F05-07 | Ghi lần giao khi mất mạng | NFR | SCR035 (+SCR014) | Chỉ đặc tả (MOCK-2) |

**Related Screens**:
- SCR014_DeliveryEntry: [spec](../screens/SCR014_DeliveryEntry/spec.md) · đã xây · `/deliveries/new`
- SCR015_DeliveryList: [spec](../screens/SCR015_DeliveryList/spec.md) · chỉ đặc tả · `/deliveries` *(đề xuất; route hiện chỉ chuyển hướng sang `/deliveries/new`)*
- SCR016_DeliveryDetail: [spec](../screens/SCR016_DeliveryDetail/spec.md) · chỉ đặc tả
- SCR017_DeliveryException: [spec](../screens/SCR017_DeliveryException/spec.md) · chỉ đặc tả

**Related User Stories**: US-07, US-08, US-09 (dùng chung F003), US-18 (dùng chung F006)

**Related APIs/Routes**:
- (GET) `/deliveries` (chuyển hướng), `/deliveries/new` — đã xây; Server Action `recordDelivery`
- RPC `record_delivery(p_trade_id, p_quantity, p_status, p_exception_type, p_reason)`, `log_blocked_write_attempt(p_table, p_business_date, p_error_code, p_context)` — đã xây
- RPC `complete_trade_delivery`, `decide_delivery_exception`, `sync_queued_delivery` *(đề xuất)*

**Related Data Models**: `deliveries`, `trades`, `lots`; view `v_trade_delivery_status`; đề xuất `delivery_exception_requests`

**Related Background Logic**: trigger `guard_closed_day_deliveries`; ghi nhật ký lần thử bị chặn (Server Action → RPC riêng).

**Related Permissions**: `receiving` (ghi); `reconcile` duyệt ngoại lệ *(đề xuất)*.

---

### F006_ReconciliationAndClosing: Đối chiếu & chốt kỳ

**Type**: ui
**Description**: Bảng đối chiếu ngày, chốt sổ (cảnh báo chứ không chặn — ADR-005), chặn sửa trực tiếp sau chốt, tranh chấp, lịch nghỉ chợ. Tài liệu: [business-context](../features/F006_ReconciliationAndClosing/business-context.md) · [technical-spec](../features/F006_ReconciliationAndClosing/technical-spec.md) · [screens](../features/F006_ReconciliationAndClosing/screens.md) · [edge-cases](../features/F006_ReconciliationAndClosing/edge-cases.md).

**Workspace**: `app/(app)/reconciliation/`, `lib/domain/reconciliation.ts`, `supabase/migrations/0006`, `0008`, `0010`, `0040`
**Languages**: TypeScript · SQL
**Components**: 5 tính năng con LAB-1

| Mã LAB-1 | Tính năng | Ưu tiên | Màn | Trạng thái |
|---|---|---|---|---|
| F06-01 | Lập bảng đối chiếu ngày | P0 | SCR018 | Đã xây — lỗi **B-05** đã xác nhận: `total_amount` bị nhân theo số lần giao (ngày 2026-09-23: 53250.00 so với 38250.00 thật); chưa sửa, đã có bản vá nháp `0042_fix_daily_reconciliation_amount.sql`, chưa áp |
| F06-02 | Chốt sổ ngày nghiệp vụ | P0 | SCR018 | Đã xây |
| F06-03 | Chặn sửa trực tiếp sau chốt | P0 | — (hành vi nền; hiện ở SCR014, SCR018) | Một phần — chặn đủ 3 bảng; log lần thử chỉ phủ đường ghi lần giao |
| F06-04 | Quản lý tranh chấp | P1 | SCR019, SCR020 | Chỉ đặc tả |
| F06-05 | Quản lý lịch nghỉ chợ và ngày nghiệp vụ | P1 | SCR021 | Chỉ đặc tả — lịch nhập tay đã chốt (PREMISE C-08, `calendar_mode=manual`); khách chưa xác nhận giả định (câu 16) |

**Related Screens**:
- SCR018_DailyReconciliation: [spec](../screens/SCR018_DailyReconciliation/spec.md) · đã xây · `/reconciliation`
- SCR019_DisputeList: [spec](../screens/SCR019_DisputeList/spec.md) · chỉ đặc tả · `/disputes` *(đề xuất)*
- SCR020_DisputeDetail: [spec](../screens/SCR020_DisputeDetail/spec.md) · chỉ đặc tả
- SCR021_MarketCalendar: [spec](../screens/SCR021_MarketCalendar/spec.md) · chỉ đặc tả · `/markets/[id]/calendar` *(đề xuất; nằm dưới quản lý chợ [F012](../features/F012_MarketManagement/technical-spec.md))*

**Related User Stories**: US-10, US-11, US-12, US-18 (phần tranh chấp)

**Related APIs/Routes**:
- (GET) `/reconciliation` — đã xây; Server Action `closeDay`, `getCloseConditionsAction`
- RPC `close_business_day(p_date)` — đã xây
- RPC chuyển trạng thái tranh chấp, quản lý lịch nghỉ *(đề xuất, tên chưa chốt)*

**Related Data Models**: `business_days`, `audit_log`; view `v_daily_reconciliation`; đề xuất `disputes`, `dispute_events`, `market_holidays`

**Related Background Logic**: trigger `guard_closed_day` (3 bảng); RPC `log_blocked_write_attempt`.

**Related Permissions**: `reconcile` (chốt sổ, tranh chấp); `rule_admin` (lịch nghỉ) *(đề xuất)*.

---

### F007_PostCloseCorrection: Điều chỉnh sau chốt

**Type**: ui
**Description**: Phiếu xin điều chỉnh (lý do + chứng cứ), duyệt người-khác-người-đề-nghị, sinh bản ghi reverse/delta không ghi đè, truy vết về phiếu. Thuộc [FL-02](../flows/FL-02-post-close-correction.md). Đặc tả toàn bộ — chưa xây. Tài liệu: [business-context](../features/F007_PostCloseCorrection/business-context.md) · [technical-spec](../features/F007_PostCloseCorrection/technical-spec.md) · [screens](../features/F007_PostCloseCorrection/screens.md) · [edge-cases](../features/F007_PostCloseCorrection/edge-cases.md).

**Workspace**: chưa có (đặc tả)
**Languages**: TypeScript · SQL (dự kiến)
**Components**: 5 tính năng con LAB-1

| Mã LAB-1 | Tính năng | Ưu tiên | Màn | Trạng thái |
|---|---|---|---|---|
| F07-01 | Tạo phiếu xin điều chỉnh | P0 | SCR022 | Chỉ đặc tả (MOCK-6) |
| F07-02 | Duyệt hoặc từ chối phiếu | P0 | SCR024 | Chỉ đặc tả |
| F07-03 | Sinh bản ghi điều chỉnh khi duyệt | P0 | SCR024 | Chỉ đặc tả |
| F07-04 | Truy vết bản ghi điều chỉnh về phiếu xin | P0 | SCR024 | Chỉ đặc tả |
| F07-05 | Danh sách phiếu chờ duyệt | P1 | SCR023 | Chỉ đặc tả |

**Related Screens**: [SCR022_CorrectionRequestCreate](../screens/SCR022_CorrectionRequestCreate/spec.md) · [SCR023_PendingRequestList](../screens/SCR023_PendingRequestList/spec.md) · [SCR024_RequestDetailApproval](../screens/SCR024_RequestDetailApproval/spec.md) — tất cả chỉ đặc tả

**Related User Stories**: US-16, US-17

**Related APIs/Routes** *(đề xuất)*: `/corrections/new`, `/corrections`, `/corrections/[id]`; RPC `create_correction_request`, `cancel_correction_request`, `decide_correction_request`

**Related Data Models** *(đề xuất)*: `correction_requests`, `correction_entries` (cho số âm ngay từ migration đầu); view `v_correction_requests`, `v_correction_target`, `v_correction_diff`

**Related Background Logic**: không có logic nền riêng; ràng buộc maker ≠ checker nằm trong RPC duyệt.

**Related Permissions**: người lập và người duyệt phải là hai người khác nhau; số cấp duyệt đã chốt một cấp (PREMISE C-03b), khách chưa xác nhận (câu 3). Cần ≥ 2 tài khoản `reconcile` hoặc chức danh duyệt riêng *(đề xuất)*.

---

### F008_IncentiveAndRateVersion: Tiền khuyến khích & version quy chế

**Type**: mixed
**Description**: Tính 完納奨励金 (hệ số thuế 110/100, làm tròn xuống theo JPY, không gia hạn, quá hạn = 0), lưu phiên bản tỷ lệ đã áp dụng, version tỷ lệ có 発効日 tương lai + maker-checker + rollback, delta kỳ sau. Đặc tả toàn bộ (MOCK-4). Tài liệu: [business-context](../features/F008_IncentiveAndRateVersion/business-context.md) · [technical-spec](../features/F008_IncentiveAndRateVersion/technical-spec.md) · [screens](../features/F008_IncentiveAndRateVersion/screens.md) · [edge-cases](../features/F008_IncentiveAndRateVersion/edge-cases.md).

**Workspace**: chưa có (đặc tả)
**Languages**: TypeScript · SQL (dự kiến)
**Components**: 6 tính năng con LAB-1

| Mã LAB-1 | Tính năng | Ưu tiên | Màn | Trạng thái |
|---|---|---|---|---|
| F08-01 | Tính tiền khuyến khích | P0 | SCR025 | Chỉ đặc tả — chờ test vector Appendix C và định nghĩa "hạn thanh toán" |
| F08-02 | Lưu phiên bản tỷ lệ đã áp dụng | P0 | SCR025 | Chỉ đặc tả |
| F08-03 | Tạo phiên bản tỷ lệ mới | P0 | SCR026, SCR027 | Chỉ đặc tả |
| F08-04 | Duyệt phiên bản tỷ lệ | P0 | SCR027 | Chỉ đặc tả |
| F08-05 | Quay lại phiên bản trước | P0 | SCR026, SCR027 | Chỉ đặc tả |
| F08-06 | Sinh chênh lệch kỳ sau | P0 | SCR025 | Chỉ đặc tả |

**Related Screens**: [SCR025_IncentiveResult](../screens/SCR025_IncentiveResult/spec.md) · [SCR026_RateVersionList](../screens/SCR026_RateVersionList/spec.md) · [SCR027_RateVersionApproval](../screens/SCR027_RateVersionApproval/spec.md) — tất cả chỉ đặc tả

**Related User Stories**: US-13, US-19

**Related APIs/Routes** *(đề xuất)*: `/incentives`, `/incentives/[period_id]/[participant_id]`, `/incentive-rates`, `/incentive-rates/new`, `/incentive-rates/[version_id]`; RPC `calculate_incentive`, `resolve_incentive_rate`, `draft_incentive_rate`, `submit_incentive_rate`, `approve_incentive_rate`, `reject_incentive_rate`, `rollback_incentive_rate`

**Related Data Models** *(đề xuất)*: `incentive_rate_versions`, `incentive_results`, `settlement_periods`; view `v_incentive_rate_versions`, `v_incentive_results`

**Related Background Logic**: tính tiền theo kỳ; tối đa 4 lần đổi/năm (GOV-RULE-01); mọi truy vấn tra `料率` theo `as_of` (ngày nghiệp vụ), không `now()`.

**Related Permissions**: người lập ≠ người duyệt; `rule_admin` *(đề xuất)*.

---

### F009_ReportsAndExport: Báo cáo & xuất dữ liệu

**Type**: mixed
**Description**: 12 báo cáo cố định (9 ngày + 3 tháng, `RPT-01` … `RPT-12`), xuất CSV, batch báo cáo ngày xong trước 11:00 JST, xuất số liệu kế toán sau khi chốt (IF-ACC-01), cảnh báo khi batch lỗi. Không có report builder (ngoài phạm vi). Đặc tả toàn bộ (MOCK-5). Tài liệu: [business-context](../features/F009_ReportsAndExport/business-context.md) · [technical-spec](../features/F009_ReportsAndExport/technical-spec.md) · [screens](../features/F009_ReportsAndExport/screens.md) · [edge-cases](../features/F009_ReportsAndExport/edge-cases.md).

**Workspace**: chưa có (đặc tả)
**Languages**: TypeScript · SQL (dự kiến)
**Components**: 6 tính năng con LAB-1

| Mã LAB-1 | Tính năng | Ưu tiên | Màn | Trạng thái |
|---|---|---|---|---|
| F09-01 | 9 báo cáo hằng ngày | P0 | SCR028, SCR029 | Chỉ đặc tả |
| F09-02 | 3 báo cáo hằng tháng | P1 | SCR028, SCR029 | Chỉ đặc tả |
| F09-03 | Xuất CSV | P0 | SCR029 | Chỉ đặc tả |
| F09-04 | Chạy batch báo cáo ngày | NFR | SCR030 (hiển thị) | Chỉ đặc tả |
| F09-05 | Xuất số liệu sang hệ thống kế toán | P0 | SCR030 | Chỉ đặc tả — chờ câu 4 (cách gửi) |
| F09-06 | Cảnh báo khi batch hoặc xuất dữ liệu lỗi | NFR | SCR030, SCR033 | Chỉ đặc tả |

**Related Screens**: [SCR028_ReportCatalog](../screens/SCR028_ReportCatalog/spec.md) · [SCR029_ReportViewExport](../screens/SCR029_ReportViewExport/spec.md) · [SCR030_AccountingBatch](../screens/SCR030_AccountingBatch/spec.md) — tất cả chỉ đặc tả

**Related User Stories**: US-14, US-15

**Related APIs/Routes** *(đề xuất)*: `/reports`, `/reports/[report_code]`, `/operations/batches`, `/operations/acc-exports`, `/operations/acc-exports/[batch_code]`; RPC `run_report`, `export_report_csv`, `preview_acc_export`, `generate_acc_export`, `retry_batch`

**Related Data Models** *(đề xuất)*: `report_runs`, `batch_runs`, `acc_export_batches`, `acc_export_lines`; view `v_report_catalog`, `v_batch_status`, `v_acc_export_batches`

**Related Background Logic**: batch báo cáo ngày (trước 11:00 JST); xuất kế toán idempotent theo `batch_code`; mọi truy vấn báo cáo theo `business_date` + `as_of`, không `now()`.

**Related Permissions**: `reconcile` *(đề xuất)*, `ops_admin` *(đề xuất)*.

---

### F010_Notifications: Thông báo

**Type**: mixed
**Description**: Thông báo hai kênh (trong ứng dụng + email, không SMS/FAX), 4 loại sự kiện, gửi quan trọng trong 5 phút, thử lại tối đa 3 lần rồi vào hàng cảnh báo vận hành, cấu hình người nhận/ngưỡng/thời điểm. Registry xếp thêm trang chủ theo chức danh (SCR001, đã xây) và hàng chờ đồng bộ (SCR035) vào feature này. Tài liệu: [business-context](../features/F010_Notifications/business-context.md) · [technical-spec](../features/F010_Notifications/technical-spec.md) · [screens](../features/F010_Notifications/screens.md) · [edge-cases](../features/F010_Notifications/edge-cases.md).

**Workspace**: `app/(app)/page.tsx`, `app/(app)/home/`, `lib/domain/home-stats.ts` (chỉ trang chủ); phần còn lại chưa có
**Languages**: TypeScript · SQL
**Components**: 5 tính năng con LAB-1

| Mã LAB-1 | Tính năng | Ưu tiên | Màn | Trạng thái |
|---|---|---|---|---|
| F10-01 | Gửi thông báo hai kênh | P1 | SCR001, SCR031 | Chỉ đặc tả — SCR001 (đã xây) mới chỉ hiện "việc cần làm" tính từ DB, không có hộp thông báo, không gửi email (MOCK-7) |
| F10-02 | Bốn loại sự kiện được thông báo | P1 | SCR031 | Chỉ đặc tả |
| F10-03 | Bảo đảm thời gian gửi việc quan trọng | P1 | — (hành vi nền; hiện ở SCR033) | Chỉ đặc tả |
| F10-04 | Thử lại khi gửi lỗi | P1 | SCR033 | Chỉ đặc tả |
| F10-05 | Cấu hình thông báo | P1 | SCR032 | Chỉ đặc tả |

**Related Screens**:
- SCR001_RoleHome: [spec](../screens/SCR001_RoleHome/spec.md) · đã xây · `/` (4 biến thể theo chức danh)
- [SCR031_NotificationInbox](../screens/SCR031_NotificationInbox/spec.md) · [SCR032_NotificationSettings](../screens/SCR032_NotificationSettings/spec.md) · [SCR033_OpsAlertQueue](../screens/SCR033_OpsAlertQueue/spec.md) · [SCR035_SyncQueue](../screens/SCR035_SyncQueue/spec.md) — chỉ đặc tả (SCR035 phục vụ F03-06, F05-07)

**Related User Stories**: không có US riêng (LAB-1: "hỗ trợ vận hành"); SCR035 phục vụ US-09

**Related APIs/Routes**:
- (GET) `/` — đã xây
- *(đề xuất)* `/notifications`, `/notifications/settings`, `/ops/alerts`, `/sync-queue`; RPC `mark_notification_read`, `mark_all_notifications_read`, `update_notification_setting`, `acknowledge_ops_alert`, `close_ops_alert`, `retry_notification_delivery`, `sync_queued_lot`, `sync_queued_delivery`

**Related Data Models** *(đề xuất)*: `notifications`, `notification_settings`, `delivery_attempts`, `ops_alerts`; hàng đợi đồng bộ nằm trong IndexedDB của máy, không có bảng

**Related Background Logic**: gửi email + retry 3 lần; việc quan trọng trong 5 phút (BR-NOTIFY-01).

**Related Permissions**: người nhận cấu hình được; `ops_admin` *(đề xuất)*; `receiving` cho hàng chờ đồng bộ.

---

### F011_AuditLog: Nhật ký kiểm toán

**Type**: mixed
**Description**: Ghi vết mọi thao tác quan trọng (ai · lúc nào · số cũ · số mới · lý do), tra cứu bốn chiều trong 2 phút, hiện phiên bản quy chế đã áp dụng, lưu trữ 7 năm. **LAB-3 ghi thật** dòng cho đổi quyền, chốt sổ và lần thử ghi bị chặn, nhưng chưa có cột `reason`, chưa phủ toàn bộ bảng và không có màn tra cứu (MOCK-8). Tài liệu: [business-context](../features/F011_AuditLog/business-context.md) · [technical-spec](../features/F011_AuditLog/technical-spec.md) · [screens](../features/F011_AuditLog/screens.md) · [edge-cases](../features/F011_AuditLog/edge-cases.md).

**Workspace**: `supabase/migrations/0002`, `0008`, `0012`, `0040` (phần ghi)
**Languages**: SQL · TypeScript (dự kiến cho màn tra cứu)
**Components**: 4 tính năng con LAB-1

| Mã LAB-1 | Tính năng | Ưu tiên | Màn | Trạng thái |
|---|---|---|---|---|
| F11-01 | Ghi vết mọi thao tác quan trọng | P0 | — (hành vi nền) | Một phần — 3 loại sự kiện; thiếu cột lý do và trigger phủ toàn bộ |
| F11-02 | Tra cứu vết theo 4 chiều | P1 | SCR034 | Chỉ đặc tả |
| F11-03 | Hiện phiên bản quy chế đã áp dụng | P1 | SCR034 (+SCR025) | Chỉ đặc tả |
| F11-04 | Lưu trữ theo thời hạn | NFR | — (hành vi nền) | Chỉ đặc tả |

**Related Screens**: [SCR034_AuditLogSearch](../screens/SCR034_AuditLogSearch/spec.md) · chỉ đặc tả · `/audit` *(đề xuất)*

**Related User Stories**: US-20

**Related APIs/Routes**: *(đề xuất)* `/audit`; view `v_audit_search`. Đường ghi hiện có: các RPC `change_user_role`, `close_business_day`, `log_blocked_write_attempt`.

**Related Data Models**: `audit_log`, `role_change_log`

**Related Background Logic**: ghi `audit_log` bên trong RPC `SECURITY DEFINER`; (đề xuất) trigger phủ toàn bộ bảng nghiệp vụ.

**Related Permissions**: hiện chỉ `sys_admin` đọc `audit_log`; `internal_audit` *(đề xuất; role riêng, không dùng cờ `can_audit`)*.

---

### F012_MarketManagement: Quản lý chợ & quy tắc vận hành

**Type**: ui
**Description**: **Mới ở LAB-4** (không có trong LAB-1; vá sai lệch D-02). Thực thể chợ và **quy tắc vận hành riêng từng chợ dưới dạng dữ liệu có phiên bản** (draft → approved → superseded/rolled back), tra theo ngày hiệu lực `as_of` (không `now()`), tối đa 4 lần đổi/năm, người lập ≠ người duyệt (GOV-RULE-01). Quyết định kiến trúc: [ADR-007](../decisions/ADR-007-market-rules-as-versioned-data.md). Tài liệu: [business-context](../features/F012_MarketManagement/business-context.md) · [technical-spec](../features/F012_MarketManagement/technical-spec.md) · [screens](../features/F012_MarketManagement/screens.md) · [edge-cases](../features/F012_MarketManagement/edge-cases.md).

**Workspace**: chưa có (đặc tả)
**Languages**: TypeScript · SQL (dự kiến)
**Components**: 4 tính năng con **(mới ở LAB-4; mã `F12-01` … `F12-04` đặt tiếp theo quy tắc LAB-1, ưu tiên P0 vì tư cách phụ thuộc quy chế chợ — thiếu quy chế thì `resolve_market_rule` raise `P0017`)**

| Mã | Tính năng (đề xuất) | Ưu tiên | Màn | Trạng thái |
|---|---|---|---|---|
| F12-01 *(mới)* | Quản lý thực thể chợ | P0 | SCR037 | Chỉ đặc tả |
| F12-02 *(mới)* | Soạn, duyệt, thay thế quy tắc chợ có phiên bản | P0 | SCR037 | Chỉ đặc tả |
| F12-03 *(mới)* | Tra quy tắc theo ngày hiệu lực (`resolve_market_rule`) | P0 | — (hành vi nền; dùng ở SCR005, SCR006, SCR011, SCR036) | Chỉ đặc tả |
| F12-04 *(mới)* | Quay lại phiên bản quy tắc trước, giới hạn 4 lần/năm | P0 | SCR037 | Chỉ đặc tả |

**Related Screens**: [SCR037_MarketManagement](../screens/SCR037_MarketManagement/spec.md) · chỉ đặc tả · `/markets` *(đề xuất)*; lịch nghỉ chợ [SCR021_MarketCalendar](../screens/SCR021_MarketCalendar/spec.md) thuộc F006 nhưng nằm dưới route của màn này.

**Related User Stories**: không có US riêng; `US-19` (đổi tỷ lệ không sửa phần mềm) là US cùng khuôn, dùng làm mẫu điều kiện chấp nhận.

**Related APIs/Routes** *(đề xuất)*: `/markets`, `/markets/[id]/rules`, `/markets/[id]/rules/[rule_key]`; RPC `upsert_market`, `draft_market_rule`, `approve_market_rule`, `rollback_market_rule`, `resolve_market_rule`

**Related Data Models** *(đề xuất)*: `markets`, `market_rules`; view `v_market_rule_versions`; gắn `market_id` vào `business_days`, `lots`, `participant_qualifications`

**Related Background Logic**: `resolve_market_rule(p_market_id, p_rule_key, p_as_of)`; di trú: tạo một chợ mặc định → backfill → đổi khoá `business_days` (bước phá tương thích, làm sau cùng).

**Related Permissions** *(khớp Security Surface của SCR037)*: `rule_admin` (soạn, maker); `rule_admin` có `can_approve` và khác người soạn (duyệt/quay lui, checker); `sys_admin`, `reconcile`, `party_admin` chỉ xem; `trading`, `receiving` không mở.

---

## Summary

- **Total Features**: 12 (F001–F012)
- **Total Sub-features**: 63 (LAB-1) + 4 (F012, đề xuất) — **11 đã xây · 12 một phần · 40 chỉ đặc tả** (+ 4 chỉ đặc tả của F012)
- **Total Screens**: 37 SCR — 35 từ LAB-1 + 2 màn mới (SCR036, SCR037); **11 đã xây**, 26 chỉ đặc tả
- **Total User Stories**: 20 (US-01 … US-20); F001, F010, F012 không có US riêng
- **Total Routes**: 13 route đã xây (`page.tsx`: `/`, `/login`, `/403`, `/accounts`, `/accounts/[id]`, `/lots`, `/lots/new`, `/trades`, `/trades/negotiated`, `/trades/auction`, `/deliveries`, `/deliveries/new`, `/reconciliation`) — trong đó `/deliveries` chỉ chuyển hướng; các route còn lại là đề xuất
- **Total Data Models**: 11 bảng + 3 view + 11 enum đã xây ([entities.md](./entities.md)); bảng/view đề xuất liệt kê theo từng feature
- **Total Background Logic**: 2 hàm trigger đã xây (`guard_closed_day` trên 3 bảng, `check_auction_fields_complete`) + 6 RPC nghiệp vụ (`confirm_negotiated_trade`, `record_auction_result`, `record_delivery`, `close_business_day`, `change_user_role`, `log_blocked_write_attempt`); chưa cấp mã `BL###`
- **Total Permissions**: 4 chức danh đã có (`sys_admin`, `receiving`, `trading`, `reconcile`) + 5 đề xuất; chưa cấp mã `PERM###`
- **Languages Detected**: TypeScript (Next.js 16 App Router), SQL / PL/pgSQL (Supabase Postgres)

## Cross-Reference Validation

Kiểm bằng đối chiếu với bảng mã của bộ này và các thư mục thật trong bộ này.

- [x] 12 mã F### duy nhất, khớp tên thư mục `features/`
- [x] Mỗi feature có đủ 4 tệp (business-context, technical-spec, screens, edge-cases)
- [x] Mỗi SCR### trong 37 mã có đúng một feature sở hữu (F010 giữ SCR001 và SCR035; SCR021 thuộc F006 dù nằm dưới route của F012); mỗi SCR### có thư mục `screens/` tương ứng
- [x] Số tính năng con khớp LAB-1: 5 + 7 + 6 + 7 + 7 + 5 + 5 + 6 + 6 + 5 + 4 = 63; phân bổ P0/P1/NFR khớp bảng phân bổ của LAB-1
- [x] 20 US-01 … US-20 đều gắn một feature: US-01, 02, 05 → F003 · US-03, 06 → F004 · US-04 → F002 · US-07, 08 → F005 · US-09 → F003/F005 · US-10, 11, 12 → F006 · US-13, 19 → F008 · US-14, 15 → F009 · US-16, 17 → F007 · US-18 → F005/F006 · US-20 → F011
- [x] Mọi route "đã xây" khớp `find app -name page.tsx` (13 tệp)
- [x] Mọi RPC "đã xây" khớp `supabase/migrations/` (`0007`, `0008`, `0012`, `0030`, `0040`)
- [x] Mã `F12-01` … `F12-04` đã đặt cho 4 tính năng của F012, thống nhất ưu tiên P0 ở mọi bảng
- [ ] Mã `BL###`/`PERM###`: **không cấp** (registry không định nghĩa); liệt kê bằng tên
- [ ] Mọi route/RPC/bảng đánh dấu *(đề xuất)* là thiết kế, chưa có trong code — không được coi là đã tồn tại

**Điểm lệch nguồn đã gặp (dùng LAB-1 làm chuẩn):** bản nháp screen spec trước đó (tài liệu gốc trong repo, không nộp) gán mã tính năng khác LAB-1 cho một số màn (ví dụ SC-10 → `F05-01`, SC-11 → `F05-02`, SC-14 → `F05-03`, SC-07 → `F04-01`) và US khác (SC-10 → `US-05`, SC-18 → `US-09`); bảng trên dùng mã trong bảng tính năng và user story của LAB-1 (tài liệu gốc trong repo, không nộp).
