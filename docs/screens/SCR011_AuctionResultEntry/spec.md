# SCR011_AuctionResultEntry — Screen Spec

> Alias LAB-1: **SC-11** · せり結果の記録 · Ghi kết quả phiên đấu giá

**Screen**: SCR011_AuctionResultEntry: Ghi kết quả phiên đấu giá
**Feature**: F004_TradeRecording — [screens.md](../../features/F004_TradeRecording/screens.md)
**Type**: atomic
**Route**: `/trades/auction`
**Generated**: 2026-10-06
**Trạng thái**: Đã xây ở LAB-3 (đã vá chống mất dữ liệu khi mất mạng — commit `c62a586`, 2026-10-04)
**Tính năng LAB-1**: `F04-05` (ghi kết quả せり)
**User story**: `US-06`
**Ảnh**: [sc11-trade-auction-vi.png](../../design/screenshots/sc11-trade-auction-vi.png)

## Purpose

Ghi lại kết quả một phiên đấu giá (せり, ~10% khối lượng giao dịch) **sau khi phiên đã diễn ra ở hiện trường** — đấu giá bằng 手やり/符丁 diễn ra ngoài hệ thống, người `trading` chỉ nhập tay lại kết quả cuối cùng. Đây là điểm **cố ý không tự động hoá** (`CAP-HUMAN-01`, FR-SERI-01 — "không OCR, không nhận dạng 手やり/符丁, không gợi ý tự động"): thiếu tính năng ở màn này là **yêu cầu của RFP**, không phải nợ kỹ thuật.

## Screen Layout

Khung `AppLayout`; nội dung xếp dọc `gap-6`: `PageHeader`, banner thành công, banner lỗi, rồi form 6 ô xếp một cột `max-w-lg` + nút "Ghi kết quả" (`app/(app)/trades/auction/auction-form.tsx:23-117`). Không có vùng tham khảo lô/người mua (khác `SCR010`), không có ô tải ảnh.

### Layout Sketch

```
┌────────────────────────────────────────────────┐
│ R1: AppHeader + R2: AppNav (khung chung)       │
├────────────────────────────────────────────────┤
│ R3: PageHeader "Ghi kết quả phiên đấu giá"     │
│ R4: Banner thành công (xanh) - - - - - - - - - │
│ R5: Banner lỗi (đỏ) - - - - - - - - - - - - -  │
│ R6: Form (max-w-lg)                            │
│     Mã lô* · Mã người thắng* · Số lượng*       │
│     Đơn giá* · Thời điểm phiên* · Người xác nhận*│
│     [Ghi kết quả]                              │
└────────────────────────────────────────────────┘
```

### Layout Regions

| Region ID | Name | Position | Scrollable | Key Components | Responsive Behavior |
|-----------|------|----------|------------|----------------|---------------------|
| R1/R2 | Header + Nav | static | no | khung chung | — |
| R3 | Tiêu đề | static | no | `PageHeader` | always |
| R4 | Banner thành công | static (có điều kiện) | no | `div[role=status]` | state riêng |
| R5 | Banner lỗi | static (có điều kiện) | no | `div[role=alert][aria-live=assertive]` (không tự nhận focus) | state riêng |
| R6 | Form 6 ô | static | no | 6 `Field`, nút submit | `max-w-lg` |

## Child Routes

Không áp dụng — màn này là một route đơn, không phải khung chứa route con.

## User Flow

> **Scope:** within-screen interactions only.

### Happy Path

1. Trang tải, 6 ô trắng, không field nào tự focus.
2. Nhập lần lượt: mã lô, mã người thắng, số lượng, đơn giá, thời điểm phiên đấu giá, người xác nhận (thứ tự tự do, không có gợi ý `datalist`).
3. Bấm "Ghi kết quả" (hoặc Enter trong một ô text — submit mặc định của `<form>`).
4. Client kiểm còn trống field nào không (`validateFormat`); sau đó Server Action tự kiểm role → đọc + kiểm định dạng 6 field (rỗng, số dương, ngày giờ hợp lệ) → không hợp lệ → lỗi `generic` (không có field-level).
5. Tra `lotCode`/`buyerCode` → `id`; không tìm thấy → `P0001`.
6. Gọi RPC `record_auction_result` (cùng khung `_confirm_trade` với `SCR010`): khoá dòng lô → kiểm ngày chốt (`P0002`) → kiểm tư cách người thắng (`P0003`) → kiểm số lượng còn lại (`P0004`) → insert `trades` (`trade_type='auction'`) + set `auction_session_at`/`auction_confirmed_by_name` + cập nhật `lots.status` nếu hết hàng.
7. **Nhánh thành công:** banner xanh, 6 field xoá về trắng (`setValues(blankValues())`), không tự focus.

### Branches

| Decision point | Condition | Outcome on this screen | Source |
|----------------|-----------|------------------------|--------|
| Bước 4 | còn field trống | banner `V-format`, không gọi server | `use-auction-form.ts:82-93` |
| Bước 4–6 | bị từ chối (P0001–P0005 / generic) | banner đỏ, 6 field giữ nguyên để sửa và gửi lại | `use-auction-form.ts:109-111` |
| Bước 4–6 | mất mạng | banner lỗi mạng, 6 field giữ nguyên, gửi lại được ngay | `use-auction-form.ts:112-114` |

## Data Inventory

| Field | Display Label | Source | Format | Empty Behavior | Cross-ref |
|-------|---------------|--------|--------|----------------|-----------|
| `tradeCode` (banner) | "Đã ghi kết quả đấu giá {trade_code}. Lô còn lại {remaining} {unit}." | RPC trả `{trade_code, remaining_after}` | text | ẩn khi chưa ghi | `trades.trade_code` |
| `unit` (banner) | đơn vị tính cho banner thành công | `select` từ `v_lot_availability` + `commodities` qua `lookupUnitForBanner` (gọi `getLotPreview` nội bộ, nuốt lỗi — trả `undefined` nếu thất bại) — chỉ tra **sau** khi ghi thành công, không hiện trước lúc nhập | text | `""` | `commodities.unit` |

Không có vùng tham khảo lô/người mua riêng (khác `SCR010`) — đây là khác biệt thiết kế có chủ đích (CAP-HUMAN), LAB-5 không "thêm cho đủ bộ" giống `SCR010`.

## UI States

| State | Trigger | Visual Behavior | User Action Available | Source |
|-------|---------|----------------|-----------------------|--------|
| empty | mở trang lần đầu / sau khi ghi thành công | 6 ô trắng | nhập | `use-auction-form.ts:25-34,105` |
| saving | `isPending` (`useTransition`) | nút "Ghi kết quả" đổi nhãn "Đang ghi…", disabled | none | `auction-form.tsx:110-116` |
| success | RPC trả `{trade_code, remaining_after}` | banner xanh `role="status"`: "Đã ghi kết quả đấu giá {trade_code}. Lô còn lại {remaining} {unit}."; 6 field trắng lại, **không tự focus field nào** | nhập kết quả kế | `use-auction-form.ts:99-107` |
| error (nghiệp vụ) | RPC trả P0001–P0005 / `generic` | banner đỏ `role="alert" aria-live="assertive"`, dữ liệu 6 field **giữ nguyên** | sửa & gửi lại | `auction-form.tsx:38-46` |
| error (mạng) | `recordAuctionResult` throw (mất mạng giữa chừng) | banner đỏ `role="alert" aria-live="assertive"` ngay trong form (`t.auction.networkError`), **không điều hướng, không mất dữ liệu** — 6 field giữ nguyên giá trị đã gõ, gọi lại được ngay | gửi lại | `use-auction-form.ts:112-114` |
| read-only | ngày nghiệp vụ đã chốt | không có giao diện riêng — rơi vào nhánh `P0002` như mọi lỗi nghiệp vụ | — | `0007_rpc_functions.sql:65-69` |

## Validation & Error Feedback

### A) Client-side

| Field | Type | Required | Constraints | Async Check | Error Message |
|-------|------|----------|-------------|-------------|---------------|
| `lotCode` | text | yes | không rỗng; phải tồn tại trong `lots.lot_code` | Server Action tra mã | `V-format` / `P0001` |
| `buyerCode` | text | yes | không rỗng; phải tồn tại trong `participants.code` | Server Action tra mã | `V-format` / `P0001` |
| `quantity` | number (`step=0.001`) | yes | `> 0` | không | `V-format` / `generic` |
| `unitPrice` | number (`step=0.01`) | yes | `> 0` | không | `V-format` / `generic` |
| `sessionAt` | `datetime-local` | yes | phải parse được thành ngày giờ hợp lệ | không | `V-format` / `generic` |
| `confirmedByName` | text | yes | không rỗng | không | `V-format` |

**Không có field tải ảnh, không có field gợi ý/tự động điền** — đúng ràng buộc CAP-HUMAN. Toàn bộ 6 field là input thô, không có `datalist`/combobox nào (khác `SCR007`/`SCR010` vốn gợi ý mã qua `datalist`).

`sessionAt`/`confirmedByName` tách biệt với `confirmed_at`/`confirmed_by` (cột hệ thống, tự động = thời điểm/người gọi API) — hai cặp cố ý khác nhau vì terminal dùng chung theo ca: người chịu trách nhiệm kết quả phiên đấu giá tại hiện trường có thể không phải người đang gõ máy (tài liệu gốc trong repo, không nộp: `architecture.md` §3.6; `supabase/migrations/0030_auction_manual_fields.sql:1-10`).

#### Field Labels & Defaults *(mục bổ sung)*

| # | Field | Nhãn VI | Nhãn JA | Mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|
| 1 | `lotCode` | Mã lô | ロット番号 | rỗng | nhập tay → tra `lots.id` |
| 2 | `buyerCode` | Mã người thắng | 落札者コード | rỗng | nhập tay → tra `participants.id` |
| 3 | `quantity` | Số lượng | 数量 | rỗng | nhập tay → `trades.quantity` |
| 4 | `unitPrice` | Đơn giá | 単価 | rỗng | nhập tay → `trades.unit_price` |
| 5 | `sessionAt` | Thời điểm phiên đấu giá | せり実施日時 | rỗng | nhập tay → `trades.auction_session_at` |
| 6 | `confirmedByName` | Người xác nhận | 確認者 | rỗng | nhập tay → `trades.auction_confirmed_by_name` |

#### Mã lỗi hiển thị

| Mã | Điều kiện kích hoạt | Thông báo VI | Thông báo JA | Hiện ở đâu | Chặn gửi? |
|---|---|---|---|---|:---:|
| `V-format` (client, không field-level) | thiếu field bắt buộc (`validateFormat` chỉ kiểm còn trống) | Vui lòng nhập đủ tất cả các trường. | すべての項目を入力してください。 | banner `role="alert"` đầu form | ● |
| `generic` (server, mã `generic`, không field-level) | `quantity`/`unitPrice ≤ 0`, hoặc `sessionAt` không parse được thành `Date` hợp lệ (Server Action kiểm lại) | Có lỗi xảy ra khi gửi giao dịch, vui lòng thử lại. | 取引の送信中にエラーが発生しました。もう一度お試しください。 | banner | ● |
| `P0001` | mã lô hoặc mã người thắng không tra được `id` | Không tìm thấy mã lô hoặc mã người mua đã nhập. Kiểm tra lại hai mã này. | 入力されたロット番号または買参人コードが見つかりません。両方のコードを確認してください。 | banner | ● |
| `P0002` | ngày nghiệp vụ của lô đã chốt sổ | Ngày nghiệp vụ {business_date} đã chốt sổ lúc {closed_at}, không thể ghi thêm giao dịch. | 業務日{business_date}はすでに{closed_at}に締め処理されており、取引を追加できません。 | banner | ● |
| `P0003` | tư cách người thắng hết hiệu lực (kiểm trong RPC, cùng khung với `SCR010`) | Người mua {buyer_name} đã hết hạn tham gia từ {expired_on}, căn cứ {basis_label}. Liên hệ {contact_office} để xử lý. | 買参人{buyer_name}の参加資格は{expired_on}に失効しています（根拠：{basis_label}）。{contact_office}にお問い合わせください。 | banner | ● |
| `P0004` | `remaining < quantity` (kiểm trong RPC) | Lô {lot_code} chỉ còn {remaining} {unit}, không đủ cho yêu cầu {requested} {unit}. Đã bán cho: {sold_to}. | ロット{lot_code}の残数量は{remaining} {unit}で、要求数量{requested} {unit}に不足しています。販売済み取引：{sold_to}。 | banner | ● |
| `P0005` | role ≠ `trading` | Chức danh của bạn không được phép thực hiện thao tác này. | この操作を行う権限がありません。 | banner | ● |
| (network) | lỗi `fetch`/Server Action throw | Mất kết nối mạng — không thể gửi kết quả. Kiểm tra lại kết nối rồi thử lại. | ネットワーク接続が切断されました — 結果を送信できません。接続を確認して再度お試しください。 | banner `role="alert"` trong form, giống mọi lỗi nghiệp vụ khác — `useAuctionForm` gọi `recordAuctionResult` trong `try/catch` + `useTransition` (cùng khuôn mẫu `SCR010`/`SCR014`) | ● |

Cùng khung kiểm với `SCR010` (dùng chung `_confirm_trade` nội bộ, chỉ khác `trade_type='auction'`) — `P0003`/`P0004` **chạy trong RPC `record_auction_result`**, không kiểm ở client. Rẽ nhánh theo `error.code`, không so khớp `error.message` (`buildTradeRejectionMessage`, dùng chung với `SCR010`, `lib/domain/trade-rejection-message.ts:24-60`).

> **Đối chiếu code (2026-10-06):** spec cũ ghi `V-format` hiện thông báo `generic`; code hiện tại phân hai tầng — client trả "Vui lòng nhập đủ tất cả các trường." (`t.auction.validationRequired`, `lib/i18n/vi/trades.ts:161`), còn `generic` chỉ phát sinh từ Server Action khi số không dương / ngày giờ sai. Spec này theo code.

### B) Server-side

#### Ghi kết quả đấu giá
- **Endpoint:** Server Action `recordAuctionResult` (`app/(app)/trades/actions.ts:113-180`, nhận `FormData` do hook dựng từ state) → `rpc('record_auction_result', { p_lot_id, p_buyer_id, p_quantity, p_unit_price, p_session_at, p_confirmed_by_name })`
- **Request:** `lotCode, buyerCode, quantity, unitPrice, sessionAt, confirmedByName`
- **Success:** `jsonb {trade_code, remaining_after}` + `unit` (tra riêng) → `revalidatePath("/trades")` + `("/lots")`; banner xanh, 6 field xoá
- **Errors:** `P0001` | `P0002` | `P0003` | `P0004` | `P0005` | `generic` | (network)
- **Trigger:** nút "Ghi kết quả" hoặc Enter (submit mặc định); `handleFormSubmit` `preventDefault` rồi gọi action
- **Source:** `actions.ts:113-180`; RPC `0030_auction_manual_fields.sql:43-60` (gọi `_confirm_trade` ở `0007_rpc_functions.sql:41-96`)

Ghi bắt buộc qua RPC vì `trades` không có policy ghi qua RLS (giống `SCR010`; ADR-002).

## Interaction Patterns

- **Không có yêu cầu bàn phím riêng** — `SCR011` **không** nằm trong ba màn keyboard-first bắt buộc (`SCR007`, `SCR010`, `SCR014`).
- **Thứ tự Tab theo thứ tự DOM** (không có `tabIndex` tuỳ chỉnh): Mã lô → Mã người thắng → Số lượng → Đơn giá → Thời điểm phiên đấu giá → Người xác nhận → nút "Ghi kết quả" — source: `auction-form.tsx:48-117`.
- **`Enter` trong một input text submit form theo hành vi HTML gốc**; không có `Ctrl+Enter`/`Escape` riêng, không có `onKeyDown` tuỳ chỉnh — source: `auction-form.tsx:48`.
- **Focus ban đầu:** không có `autoFocus` trên field nào (khác `SCR007`/`SCR010`/`SCR014`). **Focus sau khi gửi:** không có xử lý focus chủ động ở cả hai nhánh thành công/thất bại.
- **Dialog chặn giữa luồng:** không có.
- **Mất mạng không làm mất dữ liệu đã gõ** — input controlled + gọi Server Action trong `onSubmit` (`preventDefault`) + `useTransition` + `try/catch`, không dùng `useActionState` — source: `use-auction-form.ts:47-71`.

## Accessibility

| Aspect | Status | Notes |
|--------|--------|-------|
| ARIA roles/labels | present | `Field` gắn `aria-describedby`/`aria-invalid`; lỗi `role="alert" aria-live="assertive"`; thành công `role="status" aria-live="polite"` |
| Keyboard navigation | supported | thứ tự DOM; không phím tắt riêng |
| Focus management | unmanaged | banner lỗi **không** tự nhận focus (khác `SCR010`); không focus sau thành công/thất bại |
| Screen reader compatibility | unknown | chưa kiểm |

## Conditional Rendering

| Condition | Type | Renders | Hidden | Notes |
|-----------|------|---------|--------|-------|
| role ≠ `trading` | auth | — | cả route | `proxy.ts` → `/403`; `recordAuctionResult` tự kiểm tiếp (`actions.ts:118`) |
| `successBanner` | ui-state | banner xanh | — | `auction-form.tsx:24-36` |
| `errorMessage` | ui-state | banner đỏ | — | `auction-form.tsx:38-46` |
| `isPending` | ui-state | nhãn "Đang ghi…", nút disabled | nhãn "Ghi kết quả" | `auction-form.tsx:112-115` |

## Component Variants

Không áp dụng — màn không có component đổi hình dạng theo biến thể; khác biệt theo chức danh/trạng thái đã mô tả ở Conditional Rendering.

## Security Surface

| Guard | Type | Consequence if bypassed |
|-------|------|------------------------|
| `ROUTE_POLICY { prefix: "/trades/auction", roles: ["trading"] }` — `proxy.ts` | role | role khác → `/403` (`lib/auth/route-policy.ts:21`) |
| Server Action `recordAuctionResult` tự kiểm `user.role !== "trading"` → `P0005` **trước khi gọi RPC** | role | `actions.ts:117-124` |
| RPC `record_auction_result` → `_confirm_trade` tự kiểm `current_app_role()` lần nữa | role | `0007_rpc_functions.sql:53-56` |
| `trades` không có policy ghi qua RLS; ràng buộc DB: auction phải có đủ 2 trường (constraint trigger hoãn tới COMMIT) | permission | `0009_rls_policies.sql:53-55`; `0034_auction_fields_deferred_check.sql` |

## Source References

1. Page/View: `app/(app)/trades/auction/page.tsx:8-20`
2. Form: `app/(app)/trades/auction/auction-form.tsx:19-120`
3. State + submit: `app/(app)/trades/auction/use-auction-form.ts:72-131`
4. Server Action: `app/(app)/trades/actions.ts:90-180`
5. Ánh xạ lỗi: `lib/domain/rpc-errors.ts:46-78`, `lib/domain/trade-rejection-message.ts:24-60`
6. RPC/DB: `supabase/migrations/0030_auction_manual_fields.sql`, `0033_relax_auction_fields_check.sql`, `0034_auction_fields_deferred_check.sql`, `0035_auction_fields_trigger_refetch.sql`; `0007_rpc_functions.sql:41-96`
7. i18n: `lib/i18n/{vi,ja}/trades.ts` (nhánh `auction`)

## Source Walkthrough

1. **File:** `supabase/migrations/0030_auction_manual_fields.sql:43-60` — bắt đầu từ RPC: gọi lại `_confirm_trade` rồi gắn 2 trường nhập tay.
2. **File:** `app/(app)/trades/actions.ts:113-180` — Server Action: kiểm định dạng ở boundary.
3. **File:** `app/(app)/trades/auction/use-auction-form.ts:72-131` — state controlled + lý do không dùng `useActionState`.
4. **File:** `app/(app)/trades/auction/auction-form.tsx:19-120` — JSX 6 field.

### Call Hierarchy

```text
AuctionResultPage -> AuctionForm -> useAuctionForm -> recordAuctionResult -> findLotByCode + findParticipantByCode -> rpc record_auction_result -> _confirm_trade
```

### Read Calls *(mục bổ sung)*

| Thao tác | Cách gọi | Tham số | Trả về |
|---|---|---|---|
| Ghi kết quả đấu giá | `rpc('record_auction_result', {...})` gọi từ `recordAuctionResult` sau khi tra `lotCode`/`buyerCode` → `id` qua `findLotByCode`/`findParticipantByCode` | `p_lot_id, p_buyer_id, p_quantity, p_unit_price, p_session_at, p_confirmed_by_name` | `jsonb {trade_code, remaining_after}` hoặc raise `P0001`–`P0005` |
| Tra đơn vị tính cho banner thành công | `select` từ `v_lot_availability`+`commodities` qua `lookupUnitForBanner` | `lotCode` | `string \| undefined` |

### Dependencies & Risks *(mục bổ sung)*

- Cần ít nhất một lô `公開` còn `remaining > 0` để demo ghi thành công; cùng bộ seed `PTI-01`/`PTA-02` dùng chung với `SCR010` để demo `P0003`/`P0004` nếu cần.
- **Không có vùng tham khảo** (preview lô/người mua) như `SCR010` — người nhập phải tự nhớ hoặc tra ở màn khác (`SCR008`, `SCR012`) nếu muốn biết trước số lượng còn lại; đây là khác biệt thiết kế có chủ đích (CAP-HUMAN) chứ không phải thiếu sót.
- Form dùng controlled state (`use-auction-form.ts`) + tự xoá field bằng `setValues(blank)` sau khi thành công — không còn remount bằng `key` (xem `### Unresolved Questions`).
- **Quan sát từ code (2026-10-06):** `useAuctionForm` gọi `buildTradeRejectionMessage(t, {...})` **không truyền context** (`use-auction-form.ts:109-111`) nên `{buyer_name}`, `{lot_code}`, `{unit}` trong thông báo `P0003`/`P0004` được thay bằng chuỗi rỗng (`formatTemplate` thay key thiếu bằng `""`, `rpc-errors.ts:73-78`) — khác `SCR010` truyền đủ `buyerName/lotCode/unit` (`use-negotiated-form.ts:93-102`). Chưa có quyết định về việc vá (xem Unresolved Questions). Cùng với `SCR010`, đề xuất LAB-5 cho Server Action trả `buyer_name`, `lot_code`, `unit` để thông báo không phụ thuộc state client.
- Ma trận quyền (9 role, PREMISE C-09): `trading` ●/●; `sys_admin`, `receiving`, `reconcile` ✗/✗; 5 role đề xuất (`party_admin`, `rule_admin`, `ops_admin`, `market_manager`, `internal_audit`) ✗/✗. `can_approve` không liên quan.
- Link chéo: [F004](../../features/F004_TradeRecording/screens.md); flow FL-01; ADR-002, ADR-004.

### Unresolved Questions *(mục bổ sung)*

- **Đã xử lý (2026-10-04):** màn này trước dùng `useActionState` + `<form action={formAction}>` thuần, dựa lập luận sai "không có yêu cầu bàn phím tuỳ biến" trong khi lý do thật cần controlled input là chống mất dữ liệu khi mất mạng (giống `SCR007`/`SCR010`/`SCR014`) — `useActionState` khiến fetch thất bại lúc offline bị Next.js coi là tín hiệu resubmit bằng native form submission, trình duyệt điều hướng thật và xoá sạch dữ liệu đã gõ. Đã chuyển `AuctionForm`/`recordAuctionResult` sang mẫu chống mất dữ liệu dùng chung với ba màn kia: input controlled, gọi thẳng Server Action trong `onSubmit` (đã `preventDefault`) + `useTransition` + `try/catch` (`use-auction-form.ts`). Đã xác nhận bằng cách tắt mạng thật (Puppeteer `setOfflineMode`): banner lỗi mạng hiện đúng, 6 field giữ nguyên giá trị, URL không đổi; bật mạng lại và gửi lại thành công.
- *(bổ sung)* Có vá việc thiếu context cho `P0003`/`P0004` ở màn này không (xem `### Dependencies & Risks`)?
