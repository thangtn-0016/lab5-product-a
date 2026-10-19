# SCR007_LotEntry — Screen Spec

> Alias LAB-1: **SC-07** · ロット入力 · Nhập lô hàng

**Screen**: SCR007_LotEntry: Nhập lô hàng
**Feature**: F003_LotsAndInspection — [screens.md](../../features/F003_LotsAndInspection/screens.md)
**Type**: atomic
**Route**: `/lots/new`
**Generated**: 2026-10-06
**Trạng thái**: Đã xây ở LAB-3 · ⌨️ bàn phím bắt buộc
**Tính năng LAB-1**: `F03-01` (tiếp nhận lô hàng) *(`F03-02`/`F03-06` gắn với màn này ở (tài liệu gốc trong repo, không nộp) `product-a-screen-list.md` nhưng LAB-3 chưa xây: đánh giá cảm quan → `SCR009`, offline → `SCR035`).*
**User story**: `US-01`
**Ảnh**: [sc07-lot-new-vi.png](../../design/screenshots/sc07-lot-new-vi.png) · [sc07-lot-new-ja.png](../../design/screenshots/sc07-lot-new-ja.png) · [sc07-focus-vi.png](../../design/screenshots/sc07-focus-vi.png)

## Purpose

Ghi nhận một lô hàng vừa lên bờ vào hệ thống: chủ hàng nào, mặt hàng gì, số lượng bao nhiêu. Người dùng là nhân viên `receiving` ở khu tiếp nhận, thao tác liên tục từ **02:00 JST**, nhiều lô một ca, thường **đeo găng tay** và đứng ở terminal dùng chung theo ca. Vì vậy form phải nhập xong một lô, lưu, rồi vào ngay lô kế tiếp mà không rời bàn phím, và không được giữ lại dữ liệu lô trước sau khi lưu (người ca sau có thể ngồi vào đúng máy này).

`quantity` ghi ở đây là **bất biến** sau khi tạo — không có màn sửa số lượng lô (tài liệu gốc trong repo, không nộp: `db-schema.md` §2: cột `lots.quantity` không đổi; ADR-003). *(Xem câu hỏi về điều chỉnh có kiểm soát ở `SCR009`.)*

## Screen Layout

Khung `AppLayout`; nội dung là `PageHeader` rồi một thẻ form `max-w-xl` xếp dọc: dải banner (thành công `Toast` + lỗi chung), ô "Ngày nghiệp vụ" nhỏ ở đầu form (nằm ngoài luồng Tab), 3 ô bắt buộc (chủ hàng, mặt hàng, số lượng), 2 ô tuỳ chọn (số chứng từ, ghi chú), nút "Lưu và nhập tiếp" và dòng gợi ý phím tắt cuối form (`app/(app)/lots/new/lot-entry-form.tsx:52-184`). Thứ tự bắt buộc ở đầu, tuỳ chọn ở cuối — để ngắt giữa luồng vẫn giữ được phần quan trọng (nguyên tắc ⌨️ của LAB-1).

### Layout Sketch

```
┌──────────────────────────────────────────────┐
│ R1: AppHeader + R2: AppNav (khung chung)     │
├──────────────────────────────────────────────┤
│ R3: PageHeader "Nhập lô hàng"                │
│ R4: Banner - - - - - - - - - - - - - - - -   │ Toast xanh (đã lưu {lot_code}) / lỗi chung đỏ
│ ┌──────────────────────────────────────────┐ │
│ │ R5: Ngày nghiệp vụ [date] (tabIndex -1)  │ │
│ │ R6: Mã chủ hàng *  (autofocus, datalist) │ │
│ │     Mã mặt hàng *  (datalist) Đơn vị: X  │ │
│ │     Số lượng *                           │ │
│ │     Số chứng từ tiếp nhận  (tuỳ chọn)    │ │
│ │     Ghi chú (tuỳ chọn)                   │ │
│ │ R7: [Lưu và nhập tiếp]   gợi ý phím tắt  │ │
│ └──────────────────────────────────────────┘ │
└──────────────────────────────────────────────┘
```

### Layout Regions

| Region ID | Name | Position | Scrollable | Key Components | Responsive Behavior |
|-----------|------|----------|------------|----------------|---------------------|
| R1/R2 | Header + Nav | static | no | `AppHeader`, `AppNav` | khung chung `(app)/layout.tsx` |
| R3 | Tiêu đề | static | no | `PageHeader` | always |
| R4 | Dải banner | static (có điều kiện) | no | `Toast` (success, tự tắt ≥ 8 giây hoặc bấm đóng), `<p role="alert" aria-live="polite">` | `max-w-xl` |
| R5 | Ngày nghiệp vụ | static | no | `<input type="date" tabIndex={-1}>` | luôn hiện, ngoài luồng Tab |
| R6 | Các ô nhập | static | no | `CodeInput` ×2 (`<input list>` + `<datalist>`), ô số lượng, ô chứng từ, `textarea` | cột đơn |
| R7 | Nút lưu + gợi ý | static | no | `button[type=submit]`, dòng `t.keyboardHint` | always |

## Child Routes

Không áp dụng — màn này là một route đơn, không phải khung chứa route con.

## User Flow

> **Scope:** within-screen interactions only. Điều hướng chéo màn: [screen-flow.md](../../generated/screen-flow.md).

### Happy Path

1. Trang tải, `businessDate` tự điền ngày nghiệp vụ đang mở, focus vào ô mã chủ hàng.
2. Gõ mã chủ hàng (gợi ý `datalist` hiện khi gõ) → Tab.
3. Gõ mã mặt hàng → Tab (đơn vị tính hiện ra nếu mã khớp).
4. Gõ số lượng → **Enter lưu ngay** (bỏ qua hai ô tuỳ chọn còn lại), hoặc Tab để đi tiếp.
5. (Tuỳ chọn) Gõ số chứng từ tiếp nhận → Tab. (Tuỳ chọn) Gõ ghi chú.
6. Bấm "Lưu và nhập tiếp" hoặc `Ctrl+Enter`.
7. **Nhánh thành công:** server sinh `lot_code` qua `next_lot_code`, insert vào `lots` với `status` mặc định của DB (`受付`) → banner xanh hiện mã lô vừa tạo, form xoá sạch, focus về ô 1, quay lại bước 2.

### Branches

| Decision point | Condition | Outcome on this screen | Source |
|----------------|-----------|------------------------|--------|
| Bước 6 | lỗi field (rỗng, không tìm thấy mã, số lượng không hợp lệ) | thông báo đỏ dưới field liên quan + banner chung "Kiểm tra lại các trường…", dữ liệu giữ nguyên, người dùng sửa rồi gửi lại | `actions.ts:66-80,96-101`, `lot-entry-form.tsx:61-69` |
| Bước 6 | lỗi mạng / `next_lot_code` / insert thất bại | banner đỏ đầu form, dữ liệu giữ nguyên, thử lưu lại (không cần gõ lại) | `actions.ts:91-94,108-111,124-127` |
| Bước 6 | role gọi ≠ `receiving` | banner `forbidden` | `actions.ts:54-57` |
| Bất kỳ lúc nào | `Escape` | xoá toàn bộ form về trắng, focus về ô mã chủ hàng | `use-lot-entry-form.ts:101-106` |

## Data Inventory

> Field chỉ đọc / tính ra (field nhập liệu ở `## Validation & Error Feedback`).

| Field | Display Label | Source | Format | Empty Behavior | Cross-ref |
|-------|---------------|--------|--------|----------------|-----------|
| đơn vị tính | "Đơn vị: {unit}" | tính phía client từ `commodities.unit` khi `commodityCode` khớp một mã trong danh sách (`lot-entry-form.tsx:47-49,129-134`) — không gửi lên server | text | ẩn khi mã không khớp | `commodities.unit` |
| `lot_code` | "Đã lưu lô {lot_code}. Tiếp tục nhập lô mới." | sinh phía server qua RPC `next_lot_code(business_date)` (`actions.ts:105`), chỉ hiện trong banner thành công | `LOT-YYYYMMDD-NNNN` | không hiện trên form lúc nhập | `lots.lot_code` |
| gợi ý chủ hàng | option `{code} · {name}` | `listParticipants(supabase, "wholesaler")` (`page.tsx:28,33-36`) | `datalist` | rỗng nếu chưa seed | `participants` |
| gợi ý mặt hàng | option `{code} · {tên theo locale}` | `listCommodities` (`page.tsx:29,38-42`) | `datalist` | rỗng nếu chưa seed | `commodities` |
| gợi ý phím tắt | "Esc để xoá form · Ctrl+Enter để lưu từ bất kỳ ô nào" | `t.keyboardHint` | text | — | — |

## UI States

| State | Trigger | Visual Behavior | User Action Available | Source |
|-------|---------|----------------|-----------------------|--------|
| empty | mở trang lần đầu / sau khi chốt thành công | form trắng, `businessDate` tự điền ngày nghiệp vụ `open` mới nhất, focus ở ô mã chủ hàng | nhập | `use-lot-entry-form.ts:25-34,62-73` |
| saving | đang gửi (`isPending`) | nút "Lưu và nhập tiếp" disabled, mờ (`opacity-50`); nhãn không đổi | none | `lot-entry-form.tsx:174-180` |
| success | `createLot` trả `status: "success"` | banner xanh "Đã lưu lô {lot_code}. Tiếp tục nhập lô mới." (`Toast kind="success"`, có nút đóng; giữ tối thiểu 8 giây — `components/ui/toast.tsx:16`); form xoá sạch; focus quay về ô mã chủ hàng | nhập lô kế | `use-lot-entry-form.ts:62-73` |
| error (nghiệp vụ) | `validation` (field lỗi) | thông báo đỏ dưới từng field lỗi, dữ liệu các field khác giữ nguyên | sửa | `lot-entry-form.tsx:90-94,101,119,141` |
| error (mạng) | `network` | banner đỏ đầu form, **dữ liệu đã gõ được giữ nguyên nguyên vẹn** — xem `### Dependencies & Risks` | thử lưu lại | `use-lot-entry-form.ts:84-93` |
| read-only | ngày nghiệp vụ đã chốt | **không xử lý ở UI** — nếu `businessDate` chọn trúng một ngày đã `closed`, insert bị trigger `guard_closed_day`/FK chặn và rơi về nhánh `network` (thông báo chung, không phân biệt lý do) | — | `actions.ts:124-127`, `supabase/migrations/0010_closed_day_guard.sql:45-47` |

## Validation & Error Feedback

### A) Client-side

> Toàn bộ validation nghiệp vụ chạy **phía Server Action** (`createLot`); phía client chỉ lọc ký tự ô số lượng (`replace(/[^0-9.,]/g, "")`, `lot-entry-form.tsx:149`).

| Field | Type | Required | Constraints | Async Check | Error Message |
|-------|------|----------|-------------|-------------|---------------|
| `businessDate` | date | yes | phải tồn tại trong `business_days` với `status='open'` | không | `businessDateRequired` |
| `consignorCode` | text (datalist gợi ý) | yes | phải tồn tại trong `participants.code`; datalist chỉ gợi ý nhóm `wholesaler`, nhưng **server tra không lọc `party_type`** (`master-data.ts:62-73`) | Server Action tra mã | `consignorRequired`/`consignorNotFound` |
| `commodityCode` | text (datalist gợi ý) | yes | phải tồn tại trong `commodities.code` | Server Action tra mã | `commodityRequired`/`commodityNotFound` |
| `quantity` | number (`inputMode="decimal"`) | yes | `> 0`, chấp nhận cả dấu `,` và `.` làm phân cách thập phân (server `replace(",", ".")`) | không | `quantityRequired`/`quantityInvalid` |
| `referenceNo` | text | no | không ràng buộc định dạng | không | — |
| `notes` | textarea | no | không ràng buộc định dạng | không | — |

#### Field Labels & Defaults *(mục bổ sung)*

| # | Field | Nhãn VI | Nhãn JA | Mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|
| 1 | `businessDate` | Ngày nghiệp vụ | 業務日 | ngày nghiệp vụ `open` mới nhất (`getDefaultBusinessDate`, `lib/domain/lots.ts:124-134`) | `business_days.business_date` |
| 2 | `consignorCode` | Mã chủ hàng | 荷主コード | rỗng | `participants.code` (lọc `party_type='wholesaler'` ở gợi ý) |
| 3 | `commodityCode` | Mã mặt hàng | 品目コード | rỗng | `commodities.code` |
| 4 | `quantity` | Số lượng | 数量 | rỗng | nhập tay → `lots.quantity` (bất biến sau khi tạo) |
| 5 | `referenceNo` | Số chứng từ tiếp nhận | 受付伝票番号 | rỗng | nhập tay → `lots.receiving_reference_no` (migration `0020_lots_receiving_fields.sql`) |
| 6 | `notes` | Ghi chú | メモ | rỗng | nhập tay → `lots.notes` |

#### Mã lỗi hiển thị

| Mã | Điều kiện kích hoạt | Thông báo VI | Thông báo JA | Hiện ở đâu | Chặn gửi? |
|---|---|---|---|---|:---:|
| `consignorRequired` | `consignorCode` rỗng | Nhập mã chủ hàng. | 荷主コードを入力してください。 | dưới ô mã chủ hàng | ● |
| `consignorNotFound` | mã không khớp `participants.code` nào | Không tìm thấy chủ hàng với mã này. Kiểm tra lại mã. | このコードの荷主が見つかりません。コードを確認してください。 | dưới ô mã chủ hàng | ● |
| `commodityRequired` | `commodityCode` rỗng | Nhập mã mặt hàng. | 品目コードを入力してください。 | dưới ô mã mặt hàng | ● |
| `commodityNotFound` | mã không khớp `commodities.code` nào | Không tìm thấy mặt hàng với mã này. Kiểm tra lại mã. | このコードの品目が見つかりません。コードを確認してください。 | dưới ô mã mặt hàng | ● |
| `quantityRequired` | `quantity` rỗng | Nhập số lượng. | 数量を入力してください。 | dưới ô số lượng | ● |
| `quantityInvalid` | `quantity` không phải số hoặc `≤ 0` | Số lượng phải là số lớn hơn 0. | 数量は0より大きい数値で入力してください。 | dưới ô số lượng | ● |
| `businessDateRequired` | `businessDate` rỗng | Chọn ngày nghiệp vụ. | 業務日を選択してください。 | dưới ô ngày nghiệp vụ | ● |
| `forbidden` | role gọi Server Action ≠ `receiving` | Chức danh của bạn không được phép nhập lô hàng. | あなたの職掌ではロット入力を行えません。 | banner đầu form | ● |
| `network` | lỗi mạng / `next_lot_code` / insert thất bại | Chưa lưu được do lỗi kết nối. Dữ liệu trên form vẫn được giữ nguyên, hãy thử lưu lại. | 通信エラーで保存できませんでした。入力内容は保持されています。もう一度保存してください。 | banner đầu form | ● |

Toàn bộ validation ở đây chạy **phía Server Action** (`createLot`), không phải RPC lỗi SQLSTATE — màn này không gọi RPC nghiệp vụ nào ngoài `next_lot_code`. **Không có mã `P0001`–`P0009` nào liên quan trực tiếp đến thao tác ghi của SCR007** (INSERT trực tiếp vào `lots`, có policy RLS cho `receiving`). Khi lỗi field xảy ra, `generalError = "validation"` nên banner chung "Kiểm tra lại các trường được đánh dấu bên dưới." cũng hiện (`lot-entry-form.tsx:61-69`).

### B) Server-side

#### Tạo lô hàng
- **Endpoint:** Server Action `createLot` (`app/(app)/lots/actions.ts:50-131`) → `rpc('next_lot_code', { p_date })` rồi `insert` vào `lots`
- **Request:** `consignorCode, commodityCode, quantity, businessDate, referenceNo, notes` (mã chủ hàng/mặt hàng được `trim().toUpperCase()`)
- **Success:** `{ status: "success", lotCode }` → `revalidatePath("/lots")`, banner xanh, form xoá, focus ô 1
- **Errors:** `validation` (kèm `fieldErrors`) | `forbidden` | `network` (mọi lỗi lookup/RPC/insert, **kể cả ngày đã chốt — trigger `guard_closed_day`, mã `P0009` ở tầng DB, nhưng Server Action không bắt mã Postgres để map riêng**)
- **Trigger:** nút "Lưu và nhập tiếp", Enter (submit mặc định của form) hoặc `Ctrl+Enter`
- **Source:** `app/(app)/lots/actions.ts:50-131`; hook `use-lot-entry-form.ts:84-98`

## Interaction Patterns

- **Thứ tự Tab:** Mã chủ hàng (autofocus) → Mã mặt hàng → Số lượng → Số chứng từ tiếp nhận → Ghi chú → nút "Lưu và nhập tiếp". Ô "Ngày nghiệp vụ" có `tabIndex={-1}` — nằm ngoài luồng Tab chính dù hiện ở đầu form (xác nhận qua ảnh `sc07-focus-vi.png`: sau 3 lần Tab từ ô mã chủ hàng, focus rơi đúng vào "Số chứng từ tiếp nhận", bỏ qua ngày nghiệp vụ) — source: `lot-entry-form.tsx:22-25,84`.
- ⌨️ **`Ctrl+Enter` lưu từ bất kỳ ô nào trong form** (bắt ở cấp `<form onKeyDown>`, kể cả khi đang gõ trong `textarea` ghi chú) — source: `use-lot-entry-form.ts:107-110`.
- ⌨️ **Enter trong input thường submit form theo hành vi HTML gốc** (không có handler riêng; nút `type="submit"` nằm trong cùng `<form onSubmit>`) — source: `lot-entry-form.tsx:71-76`.
- ⌨️ **`Escape` xoá toàn bộ form về trắng và đưa focus về ô mã chủ hàng** — source: `use-lot-entry-form.ts:79-82,101-106`.
- **Focus ban đầu:** ô mã chủ hàng (`autoFocus`) — `lot-entry-form.tsx:110`. **Focus sau khi gửi:** thành công → về ô mã chủ hàng (`consignorRef.current?.focus()`, `use-lot-entry-form.ts:71`); thất bại (validation hoặc mạng) → không có xử lý focus riêng, focus ở nguyên vị trí người dùng vừa rời.
- **Dialog chặn giữa luồng:** không có.
- **Gõ ký tự không phải số vào ô số lượng bị loại ngay** (chỉ cho `0-9 . ,`) — source: `lot-entry-form.tsx:149`.
- **Gợi ý mã dùng `<input list>` + `<datalist>` gốc HTML**, không dùng combobox thư viện (mọi combobox thư viện đều cần chuột để mở danh sách) — source: `app/(app)/lots/new/code-input.tsx:21-29`.
- **Không giữ dữ liệu lô trước sau khi lưu** (máy dùng chung theo ca) — source: `use-lot-entry-form.ts:69`.

## Accessibility

| Aspect | Status | Notes |
|--------|--------|-------|
| ARIA roles/labels | present | `Field` gắn `aria-describedby`/`aria-invalid` (`components/ui/field.tsx:28-32`); lỗi field `role="alert"`; banner lỗi chung `aria-live="polite"`; `Toast` `role="status"` |
| Keyboard navigation | supported | Tab/Enter/Ctrl+Enter/Escape như trên; không có hành động chỉ hiện khi hover |
| Focus management | managed | autofocus ô 1; về ô 1 sau khi lưu thành công; **không** focus banner khi thất bại |
| Screen reader compatibility | unknown | chưa kiểm bằng trình đọc màn hình |

## Conditional Rendering

| Condition | Type | Renders | Hidden | Notes |
|-----------|------|---------|--------|-------|
| role ≠ `receiving` | auth | — | cả route | `proxy.ts` → `/403`; `createLot` tự kiểm tiếp (`actions.ts:55`) |
| `banner` có giá trị | ui-state | `Toast` thành công | — | `lot-entry-form.tsx:53-60` |
| `generalError` | ui-state | `<p role="alert">` | — | `lot-entry-form.tsx:61-69` |
| `matchedCommodity` | ui-state | dòng "Đơn vị: {unit}" | — | `lot-entry-form.tsx:129-134` |
| `fieldErrors.*` | ui-state | thông báo dưới ô | — | `lot-entry-form.tsx:90-94,101,119,141` |

## Component Variants

Không áp dụng — màn không có component đổi hình dạng theo biến thể; khác biệt theo chức danh/trạng thái đã mô tả ở Conditional Rendering.

## Security Surface

| Guard | Type | Consequence if bypassed |
|-------|------|------------------------|
| `ROUTE_POLICY { prefix: "/lots/new", roles: ["receiving"] }` — `proxy.ts` | role | role khác → `/403`; entry này đứng **trước** `/lots` để không bị prefix chung nuốt (`lib/auth/route-policy.ts:18-19`) |
| **Tầng 2** — Server Action `createLot` tự kiểm `user.role !== "receiving"` → `forbidden`, độc lập proxy (lý do: Server Function là POST tới chính route chứa nó — ADR-004, (tài liệu gốc trong repo, không nộp) `architecture.md` §5) | role | `actions.ts:54-57` |
| RLS `lots`: SELECT `receiving`+`trading`; INSERT chỉ `receiving` | permission | `supabase/migrations/0009_rls_policies.sql:44-47` |
| Trigger `guard_closed_day` chặn INSERT/UPDATE/DELETE `lots` khi ngày đã chốt (`P0009`) | permission | `0010_closed_day_guard.sql:45-47` |

Không có RLS chặn SELECT trên `lots` cho `receiving`/`trading` (cả hai cùng xem được), nhưng INSERT chỉ có policy cho `receiving`.

## Source References

1. Page/View: `app/(app)/lots/new/page.tsx:19-56`
2. Form: `app/(app)/lots/new/lot-entry-form.tsx:26-186`
3. State + bàn phím: `app/(app)/lots/new/use-lot-entry-form.ts:53-129`
4. Ô gõ mã: `app/(app)/lots/new/code-input.tsx:30-60`
5. Server Action: `app/(app)/lots/actions.ts:50-131`
6. Truy vấn: `lib/domain/master-data.ts:62-86,110-131`; `lib/domain/lots.ts:124-134`
7. Phân quyền: `lib/auth/route-policy.ts:18`; RLS `supabase/migrations/0009_rls_policies.sql:44-47`; trigger `0010_closed_day_guard.sql`; RPC `0004_business_days_and_lots.sql:15-27`; cột `0020_lots_receiving_fields.sql`
8. i18n: `lib/i18n/{vi,ja}/lots.ts`

## Source Walkthrough

1. **File:** `app/(app)/lots/actions.ts:50-131` — bắt đầu từ Server Action: toàn bộ validation + ghi.
2. **File:** `app/(app)/lots/new/use-lot-entry-form.ts:53-129` — state controlled, bàn phím, lý do không dùng `useActionState`.
3. **File:** `app/(app)/lots/new/lot-entry-form.tsx:26-186` — JSX và thứ tự Tab.
4. **File:** `app/(app)/lots/new/code-input.tsx:30-60` — ô gõ mã thuần bàn phím.
5. **File:** `app/(app)/lots/new/page.tsx:19-56` — tải trước gợi ý ở Server Component.

### Call Hierarchy

```text
NewLotPage -> listParticipants / listCommodities / getDefaultBusinessDate -> LotEntryForm -> useLotEntryForm -> createLot -> findParticipantByCode + findCommodityByCode -> rpc next_lot_code -> insert lots
```

### Read Calls *(mục bổ sung)*

| Thao tác | Cách gọi | Tham số | Trả về |
|---|---|---|---|
| Tải danh sách chủ hàng | `select` từ `participants` qua `listParticipants(supabase, "wholesaler")` (Server Component, trước render) | `party_type='wholesaler'` | mảng `{code, name}` → `datalist` |
| Tải danh sách mặt hàng | `select` từ `commodities` qua `listCommodities(supabase)` | không lọc | mảng `{code, nameVi, nameJa, unit}` |
| Tải ngày nghiệp vụ mặc định | `select` từ `business_days` qua `getDefaultBusinessDate(supabase)` | `status='open'`, mới nhất | `business_date` ISO |
| Tra mã → id | `findParticipantByCode` / `findCommodityByCode` (`lib/domain/master-data.ts`) trong `createLot` | mã đã nhập | `Participant \| null`, `Commodity \| null` |
| Sinh mã lô | `rpc('next_lot_code', { p_date: businessDate })` | ngày nghiệp vụ | `text` `LOT-YYYYMMDD-NNNN` |
| Ghi lô hàng | `insert` **trực tiếp** vào `lots` (không qua RPC ghi) | `lot_code, business_date, consignor_id, commodity_id, quantity, receiving_reference_no, notes, created_by` | hàng vừa insert hoặc lỗi Postgres |

Ghi trực tiếp vào bảng (không qua RPC) vì `lots` có policy RLS INSERT cho `receiving` — khác `trades`/`deliveries` (không có policy ghi, bắt buộc qua RPC; ADR-002). Đọc và ghi tách hai bước (`next_lot_code` rồi `insert`) — không phải một transaction đơn, vì mỗi lô là một bản ghi mới, không trừ tồn của ai.

### Dependencies & Risks *(mục bổ sung)*

- Cần ít nhất một `participants` nhóm `wholesaler` và một `commodities` đã seed để gợi ý `datalist` có nội dung; nếu rỗng, form vẫn nhận mã gõ tay nhưng tra cứu báo `consignorNotFound`/`commodityNotFound`.
- Cần một `business_days` đang `status='open'` — nếu không, `getDefaultBusinessDate` rơi về ngày lịch hệ thống (UTC) (`lots.ts:133`), có thể không khớp `business_days` nào và khiến insert thất bại với lỗi FK (rơi vào nhánh `network`, thông báo không rõ ràng).
- **Chống mất dữ liệu khi mất mạng:** form dùng input controlled + gọi Server Action trong `onSubmit` (`preventDefault`) + `useTransition` + `try/catch`, KHÔNG dùng `useActionState` + `<form action>`: khi offline, Next.js coi fetch thất bại là tín hiệu làm lại bằng submit gốc → trình duyệt điều hướng thật và mất sạch dữ liệu (MOCK-2 cấm) — `use-lot-entry-form.ts:36-52`. Cùng khuôn với `SCR010`, `SCR011`, `SCR014`.
- `CodeInput` dùng `<input list>` + `<datalist>` gốc HTML — gợi ý có thể không giống nhau trên mọi trình duyệt/di động; hành vi chuẩn chỉ đảm bảo trên desktop với bàn phím thật (đúng mục tiêu keyboard-first).
- **Chênh lệch spec/code (2026-10-06):** spec cũ ghi mã chủ hàng "phải thuộc nhóm `wholesaler`" như một ràng buộc server; code chỉ lọc `wholesaler` ở **gợi ý**, còn tra server không lọc `party_type` — gõ tay mã của một người mua vẫn được chấp nhận làm chủ hàng (`master-data.ts:62-73`, `actions.ts:87-90`).
- Ma trận quyền (9 role, PREMISE C-09): `sys_admin` ✗/✗; `receiving` ●/● (gồm cả 目利き và 配送部門, PREMISE C-10); `trading` ✗/✗; `reconcile` ✗/✗; 5 role đề xuất (`party_admin`, `rule_admin`, `ops_admin`, `market_manager`, `internal_audit`) ✗/✗. `can_approve` không liên quan.
- Link chéo: [F003](../../features/F003_LotsAndInspection/screens.md); flow FL-01 (lô → giao dịch → giao hàng → đối chiếu); ADR-002, ADR-003, ADR-004.

### Unresolved Questions *(mục bổ sung)*

> **Ghi chú kiểm chứng (giữ từ spec cũ).** Bản nháp từng nêu nghi vấn hint "Enter để lưu ngay" không khớp hành vi thật, vì `handleFormKeyDown` chỉ xử lý `Escape` và `Ctrl+Enter`. Đã kiểm lại cấu trúc DOM: `<form onSubmit={handleFormSubmit}>` có nút `type="submit"` và ô số lượng nằm trong form đó, nên **Enter thường submit qua cơ chế mặc định của HTML**, không cần handler riêng. Hint nói đúng. Nghi vấn đã đóng, giữ lại ghi chú để người sau không mở lại.

- **Lỗi khi ngày nghiệp vụ đã chốt:** không có nhánh lỗi riêng cho trường hợp `businessDate` người dùng chọn trỏ tới một ngày `closed` — rơi vào thông báo `network` chung chung ("Chưa lưu được do lỗi kết nối") dù bản chất là bị chặn ghi (`guard_closed_day`, mã `P0009` ở tầng trigger, nhưng Server Action này không bắt riêng mã lỗi Postgres để map sang thông báo rõ nghĩa). Cần quyết định có đáng thêm nhánh lỗi riêng ở Phase sau hay không.
- *(bổ sung)* Có nên ràng buộc server chỉ nhận `party_type='wholesaler'` cho chủ hàng, hay giữ như hiện tại (chỉ gợi ý)?
