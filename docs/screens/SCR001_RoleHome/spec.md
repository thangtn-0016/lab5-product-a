# SCR001_RoleHome — Screen Spec

> Alias LAB-1: **SC-01** · 職掌別ホーム · Trang chủ theo chức danh

**Screen**: SCR001_RoleHome: Trang chủ theo chức danh
**Feature**: F010_Notifications — [screens.md](../../features/F010_Notifications/screens.md) *(màn nền: nơi thông báo trong ứng dụng hiện lên; mỗi thẻ số liệu hỗ trợ ngữ cảnh cho F### khác)*
**Type**: atomic *(4 biến thể theo role — xem `## Component Variants`)*
**Route**: `/`
**Generated**: 2026-10-06
**Trạng thái**: Đã xây ở LAB-3
**Tính năng LAB-1**: `F10-01` `F02-07` `F07-05` (tham chiếu dòng SC-01 trong (tài liệu gốc trong repo, không nộp) `product-a-screen-list.md`)
**User story**: không có US riêng — màn nền; mỗi số liệu hỗ trợ ngữ cảnh cho một US khác (xem `### Dependencies & Risks`)
**Ảnh**: [admin](../../design/screenshots/sc01-home-admin-vi.png) · [receiving](../../design/screenshots/sc01-home-receiving-vi.png) · [trading](../../design/screenshots/sc01-home-trading-vi.png) · [reconcile](../../design/screenshots/sc01-home-reconcile-vi.png)

## Purpose

Trang hiện ngay sau đăng nhập, trả lời "hôm nay có gì cần làm" theo đúng chức danh — mỗi người vào ca thấy ngay việc cần ưu tiên, không phải đi tìm ở màn khác. Thiết kế gốc LAB-1 (`F10-01`) mô tả **7 biến thể** theo 7 chức danh; bản đã xây ở LAB-3 (scope-lock `S2`, hệ quả của `D6` — chỉ có 4 role, xem (tài liệu gốc trong repo, không nộp) `lab3-scope-lock.md`) chỉ có **4 biến thể**. Bộ spec LAB-4 đã chốt 9 role (PREMISE C-09): 5 role còn lại (`party_admin`, `rule_admin`, `ops_admin`, `market_manager`, `internal_audit`, *(đề xuất)*) chưa có biến thể trang chủ — xem Unresolved Questions. Đây là rút gọn có chủ ý của bản đã xây, không phải thiếu sót.

## Screen Layout

Màn dùng khung chung `AppLayout` (header trên cùng có tên + chức danh + nút đổi ngôn ngữ + đăng xuất; thanh nav dọc `w-64` bên trái lọc theo `canAccess()`; vùng `<main>` `p-6` bên phải) (`app/(app)/layout.tsx:19-27`). Vùng nội dung là `PageHeader` (tiêu đề biến thể + lời chào) rồi lưới `StatCard`: 3 cột từ breakpoint `sm` cho `receiving`/`trading`/`reconcile`; `sys_admin` có thêm một hàng 4 thẻ theo chức danh (`app/(app)/home/admin-home.tsx:22`, `receiving-home.tsx:18`). Dưới `sm` các thẻ xếp 1 cột.

### Layout Sketch

```
┌──────────────────────────────────────────────────────────┐
│ R1: AppHeader — {Họ tên · Chức danh}  [VI][JA] [Đăng xuất]│
├──────────────┬───────────────────────────────────────────┤
│ R2: AppNav   │ R3: PageHeader  "{Tiêu đề} / Xin chào, X" │
│ (lọc theo    │ R4: lưới StatCard (3 thẻ; sys_admin:      │
│  role, w-64) │      4 thẻ chức danh + 1 thẻ đổi quyền)   │
└──────────────┴───────────────────────────────────────────┘
```

### Layout Regions

| Region ID | Name | Position | Scrollable | Key Components | Responsive Behavior |
|-----------|------|----------|------------|----------------|---------------------|
| R1 | App header | static (đầu trang) | no | `AppHeader`, nút `setLocale`, nút `signOut` | `flex-wrap`, luôn hiện |
| R2 | Nav dọc | static (trái) | no | `AppNav` + `MENU_ITEMS` lọc bằng `canAccess()` | cố định `w-64`; ẩn hẳn nếu không có mục nào |
| R3 | Tiêu đề trang | static | no | `PageHeader` | luôn hiện |
| R4 | Lưới thẻ số liệu | static | no | `StatCard` ×3 (`sys_admin`: ×4 + ×1) | `grid-cols-1` → `sm:grid-cols-3`/`sm:grid-cols-4` |

## Child Routes

Không áp dụng — màn này là một route đơn, không phải khung chứa route con.

## User Flow

> **Scope:** within-screen interactions only. Điều hướng chéo màn xem [screen-flow.md](../../generated/screen-flow.md).

### Happy Path

1. Đăng nhập thành công → `redirect("/")` → `HomePage` gọi `getCurrentUser()` (`app/(app)/page.tsx:18`).
2. Server Component gọi đúng 1 trong 4 hàm `get*HomeStats()` theo `user.role` (`app/(app)/page.tsx:24-70`).
3. Render tiêu đề biến thể + lời chào + các `StatCard` tương ứng (R3, R4).
4. Không có hành động nào khác trên chính màn này — điều hướng tiếp diễn ra qua menu R2 (ngoài phạm vi SCR001).

### Branches

| Decision point | Condition | Outcome on this screen | Source |
|----------------|-----------|------------------------|--------|
| Bước 1 | không có session / `app_users.status ≠ active` | `redirect("/login")` | `app/(app)/page.tsx:19`, `lib/auth/current-user.ts:31` |
| Bước 2 | `user.role` = `receiving`/`trading`/`reconcile`/`sys_admin` | render biến thể tương ứng | `app/(app)/page.tsx:25,36,47,58` |

## Data Inventory

> Toàn bộ là chỉ đọc. Số liệu đọc thật từ Supabase, không có hằng số trong `lib/domain/home-stats.ts` (`home-stats.ts:5-8`).

| Field | Display Label | Source | Format | Empty Behavior | Cross-ref |
|-------|---------------|--------|--------|----------------|-----------|
| (#1, cả 4) lời chào + tên | "Xin chào, {họ tên}" / "こんにちは、{氏名}" | `app_users.full_name` qua `getCurrentUser()` | text; luôn có vì đã qua `getCurrentUser()` | không áp dụng | `entities.md` app_users |
| `newLotsToday` *(receiving, #2)* | Số lô nhập hôm nay / 本日受付したロット数 | `count(lots) WHERE business_date = ngày mới nhất trong business_days` (`home-stats.ts:49-56`) | số nguyên ≥ 0 | 0 khi không có `business_days` nào | `lots` |
| `lotsPending` *(receiving, #3)* | Lô đang chờ (đã tiếp nhận / đã mở bán) / 対応待ちロット | `count(lots) WHERE status IN ('受付','公開')` — không lọc ngày (`home-stats.ts:58-61`) | số nguyên ≥ 0 | 0 | `lots` |
| `tradesAwaitingDelivery` *(receiving, #4)* | Giao dịch đã chốt còn phải giao / 確定済みで未引渡しの取引 | đếm dòng `v_trade_delivery_status` có `remaining > 0` — toàn lịch sử (`home-stats.ts:62,72`) | số nguyên ≥ 0 | 0 | `v_trade_delivery_status` |
| `lotsAvailable` *(trading, #5)* | Lô đã mở bán còn hàng / 公開中で在庫ありのロット | `lots.status='公開'` rồi lọc qua `v_lot_availability.remaining > 0` (`home-stats.ts:79-95`) | số nguyên ≥ 0 | 0 | `v_lot_availability` |
| `tradesConfirmedToday` *(trading, #6)* | Số giao dịch chốt hôm nay / 本日確定した取引数 | `trades WHERE business_date = ngày mới nhất AND status='confirmed'` (`home-stats.ts:97-106`) | số nguyên ≥ 0 | 0 | `trades` |
| `totalAmountToday` *(trading, #7)* | Tổng giá trị giao dịch hôm nay / 本日の取引総額 | `sum(trades.amount)` cùng điều kiện #6 (`home-stats.ts:107-110`) | `.toLocaleString()` (`trading-home.tsx:21`) | 0 | `trades.amount` (GENERATED) |
| `openBusinessDays` *(reconcile, #8)* | Ngày nghiệp vụ chưa chốt sổ / 未締めの業務日 | `count(business_days) WHERE status='open'` — mọi ngày đang mở (`home-stats.ts:117-123`) | số nguyên ≥ 0 | 0 | `business_days` |
| `pendingReconciliation` *(reconcile, #9)* | Số giao dịch chờ đối chiếu (còn phải giao) / 照合待ちの取引 | `trades` đã `confirmed` thuộc các ngày `open`, lọc `v_trade_delivery_status.remaining > 0` (`home-stats.ts:125-144`) | số nguyên ≥ 0 | 0 | `v_trade_delivery_status` |
| `exceptionCount` *(reconcile, #10)* | Số lần giao ngoại lệ / 例外扱いの引渡し件数 | `count(deliveries) WHERE status='exception'` — toàn lịch sử (`home-stats.ts:146-150`) | số nguyên ≥ 0 | 0 | `deliveries` |
| `accountsByRole` ×4 thẻ *(sys_admin, #11)* | Số tài khoản theo chức danh / 職掌別アカウント数 | `app_users` nhóm theo `role`, khởi tạo đủ 4 role = 0 trước khi cộng (`home-stats.ts:159-172`) | số nguyên ≥ 0 mỗi thẻ | 0 cho role chưa có tài khoản | `app_users.role` |
| `roleChangesLast7Days` *(sys_admin, #12)* | Số lần đổi quyền trong 7 ngày gần nhất / 直近7日間の権限変更回数 | `count(role_change_log) WHERE changed_at >= Date.now() − 7 ngày` — **giờ hệ thống (wall-clock), KHÔNG theo ngày nghiệp vụ** (`home-stats.ts:174-178`) | số nguyên ≥ 0 | 0 | `role_change_log` |

Ghi chú: "hôm nay" ở #2/#6/#7 = `business_date` lớn nhất trong `business_days` (bất kể status), **không phải** `new Date()` phía JS (`home-stats.ts:33-47`).

## UI States

| State | Trigger | Visual Behavior | User Action Available | Source |
|-------|---------|----------------|-----------------------|--------|
| loading | `Promise.all(...)` các truy vấn Supabase chưa xong | **Không có skeleton/spinner** — Server Component chỉ render sau khi toàn bộ truy vấn hoàn tất; không có `loading.tsx` trong `app/(app)/` | none | `app/(app)/page.tsx:17`; `find app -name loading.tsx` rỗng |
| empty | chưa có dữ liệu nghiệp vụ (môi trường mới seed) | mọi số về 0 một cách tự nhiên — không có thông báo "trống" riêng | none | `home-stats.ts:70-72` |
| error (mạng/DB) | bất kỳ truy vấn nào trong `lib/domain/home-stats.ts` lỗi → hàm `throw error` ngay; không có `try/catch` ở `page.tsx`, không có `error.tsx` trong `app/(app)/` | rơi xuống trang lỗi mặc định của Next.js — không có thiết kế riêng cho màn này | tải lại trang | `home-stats.ts:45,65-67` |
| saving / success | không áp dụng — màn chỉ đọc, không có thao tác ghi | — | — | — |
| read-only | không áp dụng — màn này vốn luôn chỉ đọc | — | — | — |

## Validation & Error Feedback

### A) Client-side

Không áp dụng — SCR001 không có form nhập liệu, chỉ đọc. Khả năng lỗi duy nhất là lỗi đọc dữ liệu từ Supabase, xem `## UI States` (error).

### B) Server-side

Không áp dụng — không có hành động kiểu submit. Không có mã lỗi `P00xx` nào phát sinh trên màn này.

## Interaction Patterns

- **Không có yêu cầu bàn phím riêng** (không thuộc 3 màn ⌨️ bắt buộc `SCR007`, `SCR010`, `SCR014`) — không có tương tác nào ngoài điều hướng bằng nav R2.
- **Lời chào/tiêu đề đổi theo ngôn ngữ ngay khi bấm VI/JA ở header** — `setLocale` đặt cookie `locale` rồi `revalidatePath("/", "layout")` — source: `lib/i18n/locale-cookie.ts:26-35`, `app/(app)/app-header.tsx:26-42`.

## Accessibility

| Aspect | Status | Notes |
|--------|--------|-------|
| ARIA roles/labels | absent | `StatCard` chỉ là `<div><p>`; không `role`/`aria-label` (`stat-card.tsx:7-13`) |
| Keyboard navigation | supported | chỉ có link/nút của header và nav; tab theo thứ tự DOM |
| Focus management | unmanaged | không có modal/autofocus |
| Screen reader compatibility | unknown | `<html lang>` theo locale (`app/layout.tsx:28-31`); chưa kiểm bằng trình đọc màn hình |

## Conditional Rendering

| Condition | Type | Renders | Hidden | Notes |
|-----------|------|---------|--------|-------|
| `user.role === "receiving"` | auth | `ReceivingHome` | 3 biến thể còn lại | `app/(app)/page.tsx:25` |
| `user.role === "trading"` | auth | `TradingHome` | — | `app/(app)/page.tsx:36` |
| `user.role === "reconcile"` | auth | `ReconcileHome` | — | `app/(app)/page.tsx:47` |
| `user.role === "sys_admin"` | auth | `AdminHome` (4 thẻ chức danh + 1 thẻ đổi quyền) | — | `app/(app)/page.tsx:58` |
| `getCurrentUser()` trả null | auth | `redirect("/login")` | toàn bộ nội dung | `app/(app)/page.tsx:19` |

## Component Variants

| Component | Discriminating field | Variants on this screen | Screen-specific props/slots | Cross-ref |
|-----------|---------------------|------------------------|----------------------------|-----------|
| `StatCard` + 4 view biến thể | `user.role` (`switch`) | `ReceivingHome`, `TradingHome`, `ReconcileHome`, `AdminHome` | `userName`, `greeting`, `home` (nhánh i18n riêng), `stats` | `lib/domain/home-stats.ts:10-31` (4 interface `*HomeStats`) |

## Security Surface

| Guard | Type | Consequence if bypassed |
|-------|------|------------------------|
| `ROUTE_POLICY` entry `{ prefix: "/", roles: "all" }` — **không chặn theo role ở tầng route** | auth | mọi role đã đăng nhập mở được; `lib/auth/route-policy.ts:26` |
| `proxy.ts` → `updateSession`: chưa đăng nhập → `/login`; `app_users.status ≠ active` → `signOut` + `/login?reason=suspended` | auth | không có session thì không tới được màn này — `lib/supabase/proxy.ts:58-80` |
| `switch (user.role)` trong `page.tsx` quyết định nội dung | role | biến thể sai role = lộ số liệu sai role; RLS các bảng/view nền (tài liệu gốc trong repo, không nộp: `db-schema.md` §6) vẫn là tầng chặn thật khi truy vấn — `lots` SELECT chỉ `receiving`+`trading` (`supabase/migrations/0009_rls_policies.sql:44-45`) |
| RLS `app_users_select_all_sys_admin` | permission | chỉ `sys_admin` đếm được `app_users` toàn bảng cho thẻ #11 — `0009_rls_policies.sql:22-23` |

Toàn bộ là `select`/`count` trực tiếp qua PostgREST, **không qua RPC** — chỉ đọc nên RLS đủ chặn theo role.

## Source References

1. Page/View: `app/(app)/page.tsx:17-71`
2. Layout khung: `app/(app)/layout.tsx:13-28`; header `app/(app)/app-header.tsx:16-55`; nav `app/(app)/app-nav.tsx:16-37`
3. Biến thể: `app/(app)/home/receiving-home.tsx:14-25`, `trading-home.tsx:14-21`, `reconcile-home.tsx:14-21`, `admin-home.tsx:16-35`; thẻ `stat-card.tsx:7-13`
4. Truy vấn: `lib/domain/home-stats.ts:49-182`
5. Phân quyền route: `lib/auth/route-policy.ts:17-37`; proxy `proxy.ts:9-17`, `lib/supabase/proxy.ts:19-91`
6. i18n: `lib/i18n/vi/home.ts`, `lib/i18n/ja/home.ts`

## Source Walkthrough

1. **File:** `lib/domain/home-stats.ts:1-47` — bắt đầu từ định nghĩa 4 interface thống kê và `getLatestBusinessDate`: hiểu "hôm nay" là gì.
2. **File:** `lib/domain/home-stats.ts:49-182` — 4 hàm truy vấn, mỗi hàm một biến thể.
3. **File:** `app/(app)/page.tsx:17-71` — điểm vào: lấy user, chọn biến thể.
4. **File:** `app/(app)/home/stat-card.tsx:7-13` — thẻ số liệu dùng chung.

### Call Hierarchy

```text
HomePage -> getCurrentUser -> switch(role) -> get{Role}HomeStats -> {Role}Home -> StatCard
```

### Read Calls *(mục bổ sung)*

| Thao tác | Cách gọi | Tham số | Trả về |
|---|---|---|---|
| Ngày nghiệp vụ mới nhất | `select business_date from business_days order by business_date desc limit 1` | không | `business_date` hoặc `null` |
| Số liệu `receiving` | `count` từ `lots`, `select` từ `v_trade_delivery_status` | ngày mới nhất, `status IN ('受付','公開')` | 3 số |
| Số liệu `trading` | `lots` (status=公開) → `v_lot_availability`; `select` từ `trades` | ngày mới nhất | 3 số |
| Số liệu `reconcile` | `business_days` (open), `trades`, `v_trade_delivery_status`, `deliveries` | — | 3 số |
| Số liệu `sys_admin` | `select role from app_users` (đếm tay phía JS), `count` từ `role_change_log` | mốc 7 ngày | object `accountsByRole` + 1 số |

### Dependencies & Risks *(mục bổ sung)*

- Cần `business_days` có ít nhất 1 dòng để "hôm nay" có nghĩa cho `receiving`/`trading` — nếu không, `newLotsToday`/`tradesConfirmedToday`/`totalAmountToday` luôn = 0 dù có dữ liệu lịch sử.
- `roleChangesLast7Days` dùng mốc giờ hệ thống (`Date.now()`), khác cách các số khác dùng "ngày nghiệp vụ" — dễ hiểu sai khi so sánh "7 ngày" giữa hai ngữ cảnh.
- 4 biến thể là rút gọn có chủ ý từ 7 biến thể gốc LAB-1 (`S2`, (tài liệu gốc trong repo, không nộp) `lab3-scope-lock.md`) — không phải thiếu sót.
- Không có `error.tsx`/`loading.tsx` trong nhóm route `app/(app)/` — một lỗi Supabase bất kỳ hiện trang lỗi mặc định Next.js. Áp dụng cho cả `SCR003`, `SCR008`, `SCR012`.
- **Đối chiếu code (2026-10-06):** spec cũ ghi `lotsPending` đếm `status IN ('受付','下見済')`; code thật đếm `('受付','公開')` (`home-stats.ts:61`) và nhãn "đã tiếp nhận / đã mở bán" khớp code. Spec này theo code.
- Hỗ trợ ngữ cảnh cho: `US-01` (nhập lô — thẻ lô), `US-03` (chốt giao dịch — thẻ giao dịch), `US-07` (giao hàng — thẻ giao dịch còn phải giao), `US-10` (đối chiếu — thẻ chờ đối chiếu). Link chéo: flow [FL-01](../../flows/) (lô → giao dịch → giao hàng → đối chiếu).

### Unresolved Questions *(mục bổ sung)*

- Có cần thêm `loading.tsx`/`error.tsx` cho nhóm `app/(app)/` để SCR001 (và các màn khác) có trạng thái tải/lỗi tử tế hơn mặc định Next.js?
- `roleChangesLast7Days` nên đổi sang tính theo ngày nghiệp vụ cho nhất quán, hay giữ nguyên giờ hệ thống vì đây vốn là số liệu quản trị (không phải nghiệp vụ chợ)?
- Trang chủ của 5 role đề xuất (`party_admin`, `rule_admin`, `ops_admin`, `market_manager`, `internal_audit`) chưa thiết kế: hiện `switch (user.role)` không có nhánh cho các role này. Cần chốt thẻ số liệu riêng, hay dùng chung một trang chủ rút gọn?
