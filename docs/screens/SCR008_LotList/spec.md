# SCR008_LotList — Screen Spec

> Alias LAB-1: **SC-08** · ロット一覧 · Danh sách lô hàng

**Screen**: SCR008_LotList: Danh sách lô hàng
**Feature**: F003_LotsAndInspection — [screens.md](../../features/F003_LotsAndInspection/screens.md)
**Type**: composite *(REG001 bộ lọc + bảng lô · REG002 nút "Mở bán" theo dòng — xem [screen-list.md](../../generated/screen-list.md))*
**Route**: `/lots`
**Generated**: 2026-10-06
**Trạng thái**: Đã xây ở LAB-3
**Tính năng LAB-1**: `F03-03`
**User story**: không có US riêng trực tiếp — hỗ trợ ngữ cảnh cho `US-01` (nhập lô) và là nguồn tra cứu lô cho `US-03` (chốt giao dịch chọn lô từ đây)
**Ảnh**: [sc08-lots-vi.png](../../design/screenshots/sc08-lots-vi.png)

## Purpose

Xem toàn bộ lô đã tiếp nhận, theo dõi 5 trạng thái và số lượng còn lại. `receiving` dùng để chuyển trạng thái "Mở bán"; `trading` chỉ xem để biết lô nào còn hàng trước khi sang `SCR010` chốt giao dịch. Đây **không** phải 1 trong 3 màn ⌨️ bắt buộc (đó là `SCR007`/`SCR010`/`SCR014`), dù cùng ở khu hiện trường.

## Screen Layout

Khung `AppLayout`; nội dung xếp dọc: `PageHeader` (tiêu đề + nút "Nhập lô mới" chỉ cho `receiving`), thanh lọc GET (trạng thái + ngày nghiệp vụ), rồi `DataTable` 8 cột (+ cột "Thao tác" chỉ `receiving`) (`app/(app)/lots/page.tsx:108-139`). Bảng cuộn ngang khi hẹp.

### Layout Sketch

```
┌───────────────────────────────────────────────────────────────┐
│ R1: AppHeader + R2: AppNav (khung chung)                      │
├───────────────────────────────────────────────────────────────┤
│ R3: PageHeader "Danh sách lô hàng"        [Nhập lô mới]*      │
│ R4: LotFilter [Trạng thái▾] [Ngày nghiệp vụ 📅] [Áp dụng][Xoá]│
│ R5: DataTable Mã lô|Ngày|Chủ hàng|Mặt hàng|SL đầu|Đã bán|Còn  │
│                lại|Trạng thái|[Thao tác: Mở bán]*             │
└───────────────────────────────────────────────────────────────┘
 * chỉ role receiving
```

### Layout Regions

| Region ID | Name | Position | Scrollable | Key Components | Responsive Behavior |
|-----------|------|----------|------------|----------------|---------------------|
| R1/R2 | Header + Nav | static | no | khung chung | — |
| R3 | Tiêu đề + hành động | static | no | `PageHeader`, `<Link href="/lots/new">` | nút ẩn với `trading` |
| R4 | Thanh lọc | static | no | `LotFilter` (`lot-filter.tsx:20-`) | `flex-wrap` |
| R5 | Bảng lô | static | ngang | `DataTable`, `StatusBadge`, `StatusAction` | tràn ngang khi hẹp |

## Child Routes

Không áp dụng — màn này là một route đơn, không phải khung chứa route con.

## User Flow

> **Scope:** within-screen interactions only.

### Happy Path

1. `receiving`/`trading` mở `/lots`.
2. Mặc định không lọc, sắp theo `received_at` giảm dần (mới nhất trước) (`lib/domain/lots.ts:61-64`).
3. Lọc theo trạng thái và/hoặc ngày nghiệp vụ → "Áp dụng" (GET).
4. (chỉ `receiving`) với lô đang `受付`/`下見済`, bấm "Mở bán" → thành công thì badge đổi sang "Đã mở bán"; thất bại thì hiện lỗi dưới nút, dòng không đổi.
5. `receiving` còn thấy nút "Nhập lô mới" ở góc trên → `SCR007_LotEntry`.

### Branches

| Decision point | Condition | Outcome on this screen | Source |
|----------------|-----------|------------------------|--------|
| Bước 3 | `status` lạ trong URL | bỏ qua lặng lẽ (`isLotStatus()`) | `lots/page.tsx:44` |
| Bước 4 | lô đã bị người khác mở bán trước | lỗi `invalidTransition` dưới nút | `lots/actions.ts:170-172` |
| Bước 2 | không có lô khớp | `EmptyState` | `lots/page.tsx:137` |

## Data Inventory

| Field | Display Label | Source | Format | Empty Behavior | Cross-ref |
|-------|---------------|--------|--------|----------------|-----------|
| `lotCode` | Mã lô / ロット番号 | `lots.lot_code` (UNIQUE, `LOT-YYYYMMDD-NNNN` sinh bởi `next_lot_code()`) | readonly text | — | `lots` |
| `businessDate` | Ngày nghiệp vụ / 業務日 | `lots.business_date` | `.toLocaleDateString()` | — | `lots` |
| `consignor` | Chủ hàng / 荷主 | tra `lots.consignor_id` qua `listParticipants()` (`lib/domain/master-data.ts:119-131`) | `{code} · {name}` | "" nếu không tra được | `participants` |
| `commodity` | Mặt hàng / 品目 | tra `lots.commodity_id` qua `listCommodities()` | `{code} · {tên theo locale}` — cookie `locale`: `vi`→`commodityNameVi`, `ja`→`commodityNameJa` | "" nếu không tra được | `commodities` |
| `quantity` | Số lượng ban đầu / 初期数量 | `lots.quantity` — bất biến sau khi tạo lô | số + đơn vị | — | `lots.quantity` |
| `soldQty` | Đã bán / 販売済み | `v_lot_availability.sold_qty` = tổng `trades.quantity` đã `confirmed` của lô — **KHÔNG phải cột lưu sẵn** (ranh giới cứng #2, (tài liệu gốc trong repo, không nộp) `db-schema.md` §3; ADR-003) | số + đơn vị | 0 nếu chưa bán | `v_lot_availability` |
| `remaining` | Còn lại / 残数量 | `v_lot_availability.remaining = quantity − soldQty`, **tính động mỗi lần đọc** | số + đơn vị | `= quantity` nếu chưa bán | `v_lot_availability` |
| `status` | Trạng thái / 状態 | `lots.status` — enum `lot_status`: `受付`(neutral) `下見済`(warning) `公開`(positive) `確定`(positive) `引渡し完了`(neutral) | `StatusBadge` | — | `lots.status` |

Cột/nhãn tone: `lots/page.tsx:20-26,52-95`.

## UI States

| State | Trigger | Visual Behavior | User Action Available | Source |
|-------|---------|----------------|-----------------------|--------|
| loading | — | không có — Server Component, không `loading.tsx` | none | `lots/page.tsx:35` |
| empty | không có lô khớp bộ lọc | "Không có lô hàng nào khớp bộ lọc." / 条件に一致するロットがありません。 | Xoá lọc | `lots/page.tsx:137` |
| success (Mở bán) | nút "Mở bán" thành công | không có banner thành công trong `StatusAction` — nhận biết qua badge dòng đổi thành "Đã mở bán" sau khi trang nạp lại (`revalidatePath("/lots")`) | — | `status-action.tsx:20-40`, `actions.ts:174` |
| error (nghiệp vụ) | 4 mã ở `## Validation & Error Feedback` | hiện ngay dưới nút của đúng dòng lô đó, cỡ chữ nhỏ (`text-xs`), `role="alert"` | bấm lại / tải lại | `status-action.tsx:33-37` |
| error (mạng) | mã `network` (lỗi kết nối hoặc DB lỗi) | như trên; không có gì để "giữ lại" vì đây là 1 cú bấm nút, không phải form nhập | bấm lại | `actions.ts:167-169` |
| error (tải) | `listLots()` throw | trang lỗi mặc định Next.js (không `error.tsx`) | tải lại | `lib/domain/lots.ts:70` |
| saving | `isPending` | nút "Mở bán" disabled | none | `status-action.tsx:28` |
| read-only | lô thuộc ngày nghiệp vụ đã chốt sổ → UPDATE bị trigger `guard_closed_day` chặn (`P0009`) | **suy ra từ code:** `publishLot` chỉ phân biệt `error` (→ `network`) và 0 dòng (→ `invalidTransition`), nên lỗi trigger rơi vào nhánh `network` (xem `### Unresolved Questions`) | — | `actions.ts:159-172`, `0010_closed_day_guard.sql:45-47` |

## Validation & Error Feedback

### A) Client-side

Bản thân `SCR008` không có form ghi — chỉ bộ lọc GET + nút hành động nhỏ.

| Field | Type | Required | Constraints | Async Check | Error Message |
|-------|------|----------|-------------|-------------|---------------|
| bộ lọc trạng thái | select, 5 lựa chọn + "Tất cả" | no | giá trị lạ bị bỏ qua lặng lẽ (`isLotStatus()` kiểm) | không | — |
| bộ lọc ngày nghiệp vụ | `input type="date"` | no | không kiểm định dạng riêng — `<input type="date">` tự ép `YYYY-MM-DD`; rỗng = không lọc | không | — |

#### Field Labels & Defaults *(mục bổ sung)*

| Field | Nhãn VI | Nhãn JA | Mặc định | Nguồn dữ liệu |
|---|---|---|---|---|
| bộ lọc trạng thái | Trạng thái | 状態 | rỗng = tất cả | `searchParams.status` |
| bộ lọc ngày nghiệp vụ | Ngày nghiệp vụ | 業務日 | rỗng | `searchParams.businessDate` |
| nút "Mở bán" (chỉ `receiving`) | Mở bán | 公開 | hiện khi `status ∈ {受付, 下見済}` | chuyển `lot_status` sang `公開` qua Server Action `publishLot` |

#### Mã lỗi hiển thị (nút "Mở bán")

Lỗi của nút "Mở bán" thuộc Server Action `publishLot`/`status-action.tsx` — nằm **trong phạm vi màn** vì nhãn lỗi render ngay trong bảng:

| Mã | Điều kiện kích hoạt | Thông báo VI | Thông báo JA | Hiện ở đâu | Chặn gửi? |
|---|---|---|---|---|:---:|
| `forbidden` | người gọi không phải `receiving` (phòng thủ tại Server Action, dù UI đã ẩn nút với role khác) | "Chức danh của bạn không được phép nhập lô hàng." | "あなたの職掌ではロット入力を行えません。" | dưới nút "Mở bán" của đúng dòng, `text-xs` | ● |
| `invalidTransition` | lô không còn ở trạng thái `受付`/`下見済` lúc gọi (ví dụ vừa bị mở bán bởi người khác) | "Lô này không ở trạng thái có thể mở bán." | "このロットは公開できる状態ではありません。" | nt | ● |
| `network` | lỗi kết nối khi gọi Server Action (hoặc lỗi DB, kể cả `P0009` ngày đã chốt) | "Chưa lưu được do lỗi kết nối. Dữ liệu trên form vẫn được giữ nguyên, hãy thử lưu lại." | "通信エラーで保存できませんでした。入力内容は保持されています。もう一度保存してください。" | nt | ● |
| `validation` | lỗi khác không khớp 3 mã trên | "Kiểm tra lại các trường được đánh dấu bên dưới." | "下記の項目を確認してください。" | nt | ● |

4 mã trên là nhãn literal tự định nghĩa trong `t.lots.errors` (kiểu `LotGeneralErrorCode`, `lots/actions.ts:22`) — **KHÔNG phải mã `P000x` của RPC** ở (tài liệu gốc trong repo, không nộp) `db-schema.md` §4, vì "mở bán" là UPDATE trực tiếp lên `lots` qua RLS, không qua RPC. Riêng `P0009` (ngày đã chốt) chỉ xuất hiện ở tầng trigger và không được map riêng.

### B) Server-side

#### Mở bán lô
- **Endpoint:** Server Action `publishLot` (`app/(app)/lots/actions.ts:144-176`) → `UPDATE lots SET status='公開' WHERE id = :lotId AND status IN ('受付','下見済')` (điều kiện trạng thái đặt trong mệnh đề `.in()` của UPDATE — đọc và ghi cùng một câu lệnh, không có khoảng hở)
- **Request:** `lotId` (hidden input)
- **Success:** `{ status: "success" }` → `revalidatePath("/lots")`
- **Errors:** `forbidden` | `network` (lỗi DB hoặc thiếu `lotId`) | `invalidTransition` (0 dòng bị cập nhật)
- **Trigger:** click nút "Mở bán" của một dòng
- **Source:** `app/(app)/lots/actions.ts:144-176`, `app/(app)/lots/status-action.tsx:20-40`

## Interaction Patterns

- **Bấm "Mở bán" chỉ ảnh hưởng đúng dòng đó; lỗi hiện ngay dưới nút** — source: `status-action.tsx:23-38`.
- **"Áp dụng"/"Xoá lọc" tải lại bằng GET** — source: `lot-filter.tsx:31-`.
- Không có yêu cầu bàn phím riêng (không thuộc 3 màn ⌨️).

## Accessibility

| Aspect | Status | Notes |
|--------|--------|-------|
| ARIA roles/labels | partial | `<label htmlFor>` cho bộ lọc; lỗi dưới nút `role="alert"`; `<th scope="col">` |
| Keyboard navigation | supported | link/select/nút theo DOM |
| Focus management | unmanaged | |
| Screen reader compatibility | unknown | trạng thái dòng phân biệt bằng badge có chữ, không chỉ màu |

## Conditional Rendering

| Condition | Type | Renders | Hidden | Notes |
|-----------|------|---------|--------|-------|
| `user.role === "receiving"` | auth | cột "Thao tác" + nút "Nhập lô mới" | — | `lots/page.tsx:97,113`; **chỉ là ẩn/hiện UI**, tầng chặn ghi thật ở `publishLot` |
| `status ∈ {受付, 下見済}` | ui-state | nút `StatusAction` | — | `lots/page.tsx:18,102` |
| `rows.length === 0` | ui-state | `EmptyState` | `<table>` | `data-table.tsx:24` |

## Component Variants

Không áp dụng — màn không có component đổi hình dạng theo biến thể; khác biệt theo chức danh/trạng thái đã mô tả ở Conditional Rendering.

## Security Surface

| Guard | Type | Consequence if bypassed |
|-------|------|------------------------|
| `ROUTE_POLICY { prefix: "/lots", roles: ["receiving","trading"] }` | role | role khác → `/403` (`route-policy.ts:19`) |
| RLS `lots` SELECT `receiving`+`trading` (khớp route policy) | permission | `0009_rls_policies.sql:44-45` |
| Phân biệt ghi/không-ghi chỉ là ẩn/hiện cột "Thao tác" theo `user.role === "receiving"` — KHÔNG phải route hay RLS riêng | role | `trading` thấy bảng nhưng không có nút; component `StatusAction` không tự kiểm lại |
| **Tầng chặn ghi THẬT**: Server Action `publishLot` `user.role !== "receiving"` → `forbidden`; RLS `lots_update` chỉ `receiving` | role/permission | `actions.ts:148-151`, `0009_rls_policies.sql:48-51` |
| Trigger `guard_closed_day` (`P0009`) | permission | UPDATE lô thuộc ngày đã chốt bị chặn — `0010_closed_day_guard.sql:45-47` |

## Source References

1. Page/View: `app/(app)/lots/page.tsx:35-141`
2. Truy vấn: `lib/domain/lots.ts:57-114` (`listLots`)
3. Bộ lọc: `app/(app)/lots/lot-filter.tsx:20-`
4. Nút "Mở bán": `app/(app)/lots/status-action.tsx:20-40`; Server Action `app/(app)/lots/actions.ts:144-176`
5. Tra tên: `lib/domain/master-data.ts:110-131`
6. Phân quyền: `lib/auth/route-policy.ts:19`; RLS `0009_rls_policies.sql:44-51`; view `0006_derived_views.sql:6-`
7. i18n: `lib/i18n/{vi,ja}/lots.ts`, `common.lotStatus`

## Source Walkthrough

1. **File:** `lib/domain/lots.ts:57-114` — bắt đầu từ `listLots`: số "đã bán/còn lại" đọc từ view, không từ cột.
2. **File:** `app/(app)/lots/page.tsx:35-141` — dựng cột, ẩn/hiện cột Thao tác theo role.
3. **File:** `app/(app)/lots/actions.ts:144-176` — `publishLot`.
4. **File:** `app/(app)/lots/status-action.tsx:20-40` — nút theo dòng.

### Call Hierarchy

```text
LotsPage -> listLots -> lots + v_lot_availability + listParticipants + listCommodities -> DataTable -> StatusBadge / StatusAction -> publishLot
```

### Read Calls *(mục bổ sung)*

| Thao tác | Cách gọi | Tham số | Trả về |
|---|---|---|---|
| Đọc danh sách lô | `select` từ `lots` qua `listLots()` | `status?`, `businessDate?` | mảng cơ bản (chưa có tên chủ hàng/mặt hàng/số còn lại) |
| Đọc số còn lại | `select` từ `v_lot_availability` theo danh sách `lot_id` vừa lấy | `lot_id IN (...)` | `sold_qty`, `remaining` — **ranh giới cứng #2: không bao giờ đọc từ cột lưu sẵn trên `lots`** |
| Tra tên chủ hàng/mặt hàng | `listParticipants()`/`listCommodities()` | không | map theo `id` |
| Mở bán (ghi) | UPDATE trực tiếp `lots.status` qua RLS (`publishLot`) | `lotId` | — |

### Dependencies & Risks *(mục bổ sung)*

- Phụ thuộc `SCR007` chạy trước để có dữ liệu; phụ thuộc `v_lot_availability` tồn tại và bật `security_invoker` (nếu không, RLS gốc của `trades` không áp dụng qua view) — `0006_derived_views.sql:6-8`.
- **Rủi ro hiểu sai nghiêm trọng:** cột "Đã bán"/"Còn lại" KHÔNG lưu trên bảng `lots` — nếu LAB-5 break task kiểu "update cột remaining khi chốt giao dịch" là SAI hoàn toàn kiến trúc; số này luôn tính lại từ `v_lot_availability` mỗi lần đọc (ADR-003).
- "Mở bán" là UPDATE trực tiếp (không qua RPC) — khác luồng ghi của `SCR010`/`SCR011`/`SCR014` (đều qua RPC transaction). Đây là ngoại lệ có chủ ý vì `lots` CÓ policy INSERT/UPDATE cho `receiving` (tài liệu gốc trong repo, không nộp: `db-schema.md` §6), không phải sai sót kiến trúc.
- `listLots` lấy **toàn bộ** `participants` + `commodities` rồi map phía JS (`lots.ts:74-91`) — chấp nhận ở quy mô seed.
- Ma trận quyền (9 role, PREMISE C-09): `sys_admin` ✗/✗; `receiving` ●/● (ghi = "Mở bán"); `trading` ●/✗; `reconcile` ✗/✗ (cần nới RLS `lots` nếu muốn xem — xem `SCR009`); 5 role đề xuất (`party_admin`, `rule_admin`, `ops_admin`, `market_manager`, `internal_audit`) ✗/✗. `can_approve` không liên quan.
- Link chéo: [F003](../../features/F003_LotsAndInspection/screens.md); flow FL-01; ADR-003, ADR-004.

### Unresolved Questions *(mục bổ sung)*

- Hành vi "Mở bán" khi lô thuộc ngày nghiệp vụ đã chốt sổ (`P0009` qua trigger `guard_closed_day`) — **đã suy ra từ code (2026-10-06)** là rơi vào mã `network` (`actions.ts:167-169`), nhưng chưa test tay trên deploy để xác nhận; cần quyết định có thêm nhánh lỗi riêng không.
- Không phân trang — chấp nhận được ở quy mô demo, cần quyết định ngưỡng khi dữ liệu lớn (RFP nêu 350–600 lô/ngày).
- Thông báo `forbidden` dùng chung chuỗi "Chức danh của bạn không được phép nhập lô hàng." (`t.lots.errors.forbidden`) cho cả nút "Mở bán" — hơi lệch nghĩa; tương tự `network` nhắc "dữ liệu trên form" dù nút này không có form. Đúng với code hiện tại; nên tách chuỗi riêng khi làm lại.
