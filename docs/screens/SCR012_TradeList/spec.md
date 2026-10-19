# SCR012_TradeList — Screen Spec

> Alias LAB-1: **SC-12** · 取引一覧 · Danh sách giao dịch

**Screen**: SCR012_TradeList: Danh sách giao dịch
**Feature**: F004_TradeRecording — [screens.md](../../features/F004_TradeRecording/screens.md)
**Type**: atomic
**Route**: `/trades`
**Generated**: 2026-10-06
**Trạng thái**: Đã xây ở LAB-3
**Tính năng LAB-1**: `F04-04` `F04-07`
**User story**: không có US riêng trực tiếp — nguồn tra cứu chung hỗ trợ `US-03`/`US-06` (xem lại giao dịch đã chốt) và `US-10` (đối chiếu)
**Ảnh**: [sc12-trades-vi.png](../../design/screenshots/sc12-trades-vi.png)

## Purpose

Xem toàn bộ giao dịch (thỏa thuận + đấu giá) đã ghi nhận, lọc theo 4 tiêu chí. `trading` dùng để kiểm lại giao dịch vừa chốt; `reconcile` dùng làm nguồn đối chiếu cuối ngày.

## Screen Layout

Khung `AppLayout`; nội dung xếp dọc: `PageHeader`, thanh lọc `TradeFilter` (GET form: ngày nghiệp vụ, mã người tham gia, trạng thái, loại giao dịch + nút Áp dụng/Xoá lọc), rồi `DataTable` 9 cột (`app/(app)/trades/page.tsx:90-104`). Bảng cuộn ngang khi hẹp.

### Layout Sketch

```
┌───────────────────────────────────────────────────────────────┐
│ R1: AppHeader + R2: AppNav (khung chung)                      │
├───────────────────────────────────────────────────────────────┤
│ R3: PageHeader "Danh sách giao dịch"                          │
│ R4: TradeFilter [Ngày 📅][Mã người tham gia][Trạng thái▾]     │
│                 [Loại▾] [Áp dụng] [Xoá lọc]                   │
│ R5: DataTable Mã GD|Ngày|Loại|Lô|Người mua|SL|Đơn giá|Thành   │
│                tiền|Trạng thái                                │
└───────────────────────────────────────────────────────────────┘
```

### Layout Regions

| Region ID | Name | Position | Scrollable | Key Components | Responsive Behavior |
|-----------|------|----------|------------|----------------|---------------------|
| R1/R2 | Header + Nav | static | no | khung chung | — |
| R3 | Tiêu đề | static | no | `PageHeader` | always |
| R4 | Thanh lọc | static | no | `TradeFilter` (`trade-filter.tsx:20-106`) | `flex-wrap` |
| R5 | Bảng giao dịch | static | ngang | `DataTable`, `StatusBadge` | tràn ngang khi hẹp |

## Child Routes

Không áp dụng — màn này là một route đơn, không phải khung chứa route con.

## User Flow

> **Scope:** within-screen interactions only.

### Happy Path

1. `trading`/`reconcile` mở `/trades`.
2. Mặc định không lọc, sắp theo `confirmed_at` giảm dần (`lib/domain/trades.ts:70`).
3. Nhập/chọn 1 hoặc nhiều trong 4 bộ lọc → "Áp dụng".
4. "Xoá lọc" → về `/trades` không query string.

### Branches

| Decision point | Condition | Outcome on this screen | Source |
|----------------|-----------|------------------------|--------|
| Bước 3 | gõ sai mã người tham gia | bảng rỗng với thông báo chung, không có gợi ý "không tìm thấy mã này" riêng | `lib/domain/trades.ts:76-80` |
| Bước 3 | `status`/`tradeType` lạ trong URL | bỏ qua lặng lẽ | `trades/page.tsx:20-26` |

## Data Inventory

| Field | Display Label | Source | Format | Empty Behavior | Cross-ref |
|-------|---------------|--------|--------|----------------|-----------|
| `tradeCode` | Mã giao dịch / 取引番号 | `trades.trade_code` (UNIQUE, sinh bởi `next_trade_code()`) | readonly text | — | `trades` |
| `businessDate` | Ngày nghiệp vụ / 業務日 | `trades.business_date` | **chuỗi ISO thô, KHÔNG format qua `.toLocaleDateString()`** (khác `SCR003`/`SCR008`) | — | `trades` |
| `tradeType` | Loại / 種別 | `trades.trade_type` | dịch qua `common.tradeType`; enum `negotiated`/`auction` | — | `trades` |
| `lotCode` | Lô / ロット | `trades.lot_id` → embed PostgREST `lots(lot_code)` | text | "" nếu embed không khớp | `lots` |
| `buyer` | Người mua / 買参人 | `trades.buyer_participant_id` → embed `participants(code, name)` | `{tên} ({mã})` | "" cho cả hai phần nếu không khớp | `participants` |
| `quantity` | Số lượng / 数量 | `trades.quantity` | `.toLocaleString()` | — | `trades` |
| `unitPrice` | Đơn giá / 単価 | `trades.unit_price` | `.toLocaleString()` | — | `trades` |
| `amount` | Thành tiền / 金額 | `trades.amount` = `quantity * unit_price` — cột **GENERATED** của Postgres, không bao giờ ghi tay | `.toLocaleString()` | — | `trades.amount` |
| `status` | Trạng thái / 状態 | `trades.status` | badge positive (`confirmed`) / danger (`cancelled`) | — | `trades` |

Cột: `trades/page.tsx:46-88`.

## UI States

| State | Trigger | Visual Behavior | User Action Available | Source |
|-------|---------|----------------|-----------------------|--------|
| loading | — | không có — Server Component, không `loading.tsx` | none | `trades/page.tsx:29` |
| empty | không có giao dịch khớp bộ lọc | "Chưa có giao dịch nào khớp bộ lọc." / 条件に一致する取引がありません。 | Xoá lọc | `trades/page.tsx:102` |
| error | `listTrades()` `throw` khi Supabase lỗi, không bắt, không `error.tsx` | rơi xuống trang lỗi mặc định Next.js | tải lại | `lib/domain/trades.ts:84` |
| saving / success | không áp dụng (không ghi trên màn này) | — | — | — |
| read-only | không áp dụng — luôn chỉ đọc | — | — | — |

## Validation & Error Feedback

### A) Client-side

Không có form ghi trên màn này. Bộ lọc `buyerCode` có hành vi đặc biệt — gõ sai mã không phải lỗi validate.

| Field | Type | Required | Constraints | Async Check | Error Message |
|-------|------|----------|-------------|-------------|---------------|
| bộ lọc `businessDate` | `input type="date"` | no | rỗng = không lọc | không | — |
| bộ lọc `buyerCode` | text, placeholder "VD: PTA-01" | no | mã không tồn tại trong `participants.code` → **trả danh sách rỗng, không báo lỗi** ("Mã người tham gia không tồn tại → không có dòng nào khớp, không phải lỗi hệ thống" — comment trong code) | `findParticipantByCode()` phía server | — (xem bảng dưới) |
| bộ lọc `status` | select, 2 lựa chọn + "Tất cả" | no | chỉ nhận `confirmed`/`cancelled`, giá trị khác bị bỏ qua (`parseStatus()`) | không | — |
| bộ lọc `tradeType` | select, 2 lựa chọn + "Tất cả" | no | chỉ nhận `negotiated`/`auction`, giá trị khác bị bỏ qua (`parseTradeType()`) | không | — |

#### Field Labels & Defaults *(mục bổ sung)*

| Field | Nhãn VI | Nhãn JA | Mặc định | Nguồn dữ liệu |
|---|---|---|---|---|
| bộ lọc `businessDate` | Ngày nghiệp vụ | 業務日 | rỗng | `searchParams.businessDate` |
| bộ lọc `buyerCode` | Người tham gia (mã) | 参加者コード | rỗng | `searchParams.buyerCode`, tra qua `findParticipantByCode()` |
| bộ lọc `status` | Trạng thái | 状態 | rỗng = tất cả | `searchParams.status` |
| bộ lọc `tradeType` | Loại giao dịch | 取引種別 | rỗng = tất cả | `searchParams.tradeType` |

#### Mã lỗi hiển thị

| Mã | Điều kiện kích hoạt | Thông báo VI | Thông báo JA | Hiện ở đâu | Chặn gửi? |
|---|---|---|---|---|:---:|
| — | `buyerCode` gõ không khớp mã người tham gia nào | (không có thông báo riêng — dùng chung "Chưa có giao dịch nào khớp bộ lọc.") | 条件に一致する取引がありません。 | vùng bảng (`EmptyState`) | không áp dụng — GET luôn "thành công", đây không phải lỗi validate |

Màn này không có mã `P00xx` nào phát sinh.

### B) Server-side

N/A — không có hành động submit/ghi. Đường đọc xem `### Data Calls`.

## Interaction Patterns

- **"Áp dụng" tải lại trang với query string mới; "Xoá lọc" là link về `/trades`** — GET form thường, không cần JS client — source: `app/(app)/trades/trade-filter.tsx:30-103`.
- **Gõ sai mã người tham gia cho ra bảng rỗng chứ không báo lỗi** — source: `lib/domain/trades.ts:76-80`.
- Không có yêu cầu bàn phím riêng.

## Accessibility

| Aspect | Status | Notes |
|--------|--------|-------|
| ARIA roles/labels | partial | `<label htmlFor>` cho cả 4 ô lọc (`trade-filter.tsx:36,49,62,78`); `<th scope="col">` |
| Keyboard navigation | supported | thứ tự DOM |
| Focus management | unmanaged | trang tải lại hoàn toàn khi lọc |
| Screen reader compatibility | unknown | chưa kiểm |

## Conditional Rendering

| Condition | Type | Renders | Hidden | Notes |
|-----------|------|---------|--------|-------|
| role ∉ {`trading`,`reconcile`} | auth | — | cả route | `receiving` bị `/403` dù RLS cho đọc `trades` (xem `## Security Surface`) |
| `status === "confirmed"` | ui-state | badge positive | badge danger | `trades/page.tsx:84` |
| `rows.length === 0` | ui-state | `EmptyState` | `<table>` | `data-table.tsx:24-26` |

## Component Variants

Không áp dụng — màn không có component đổi hình dạng theo biến thể; khác biệt theo chức danh/trạng thái đã mô tả ở Conditional Rendering.

## Security Surface

| Guard | Type | Consequence if bypassed |
|-------|------|------------------------|
| `ROUTE_POLICY { prefix: "/trades", roles: ["trading","reconcile"] }` — `proxy.ts`/`canAccess()` (đứng sau 2 entry cụ thể `/trades/negotiated`, `/trades/auction`) | role | role khác → `/403` (`lib/auth/route-policy.ts:20-22`) |
| RLS `trades_select`: `trading`+`reconcile` (+ `receiving` từ migration `0013`) | permission | `0009_rls_policies.sql:54-55`; `0013_trades_select_receiving.sql:16-21` |
| **Hai tầng không đồng bộ, có chủ ý:** route chặn `receiving` ở tầng trên dù RLS đã mở SELECT cho `receiving` (sai lệch D-05) — `receiving` **không bao giờ thấy màn này**, dữ liệu `trades` chỉ được đọc ngầm cho `SCR001`/`SCR014` | permission | đọc cả `ROUTE_POLICY` lẫn RLS mới đúng ai thật sự thấy màn |

Ma trận quyền (9 role, PREMISE C-09): `sys_admin` ✗/✗; `receiving` ✗ (route)/✗; `trading` ●/✗ (chỉ xem — ghi giao dịch thuộc `SCR010`/`SCR011`); `reconcile` ●/✗; 5 role đề xuất (`party_admin`, `rule_admin`, `ops_admin`, `market_manager`, `internal_audit`) ✗/✗. `can_approve` không liên quan.

## Source References

1. Page/View: `app/(app)/trades/page.tsx:29-105`
2. Bộ lọc: `app/(app)/trades/trade-filter.tsx:20-106`
3. Truy vấn: `lib/domain/trades.ts:61-86`; tra mã `lib/domain/master-data.ts:62-73`
4. Phân quyền: `lib/auth/route-policy.ts:22`; RLS `0009_rls_policies.sql:54-55`, `0013_trades_select_receiving.sql:16-21`
5. Component dùng chung: `components/ui/data-table.tsx:23-63`, `status-badge.tsx:16-24`
6. i18n: `lib/i18n/{vi,ja}/trades.ts`, `common.tradeType`

## Source Walkthrough

1. **File:** `lib/domain/trades.ts:61-86` — `listTrades`: embed PostgREST + lọc 4 chiều.
2. **File:** `app/(app)/trades/page.tsx:29-105` — parse `searchParams`, dựng cột.
3. **File:** `app/(app)/trades/trade-filter.tsx:20-106` — GET form (không dùng `FilterBar` vì cần ô ngày + ô text).
4. **File:** `supabase/migrations/0013_trades_select_receiving.sql:1-21` — vì sao RLS và route lệch nhau.

### Call Hierarchy

```text
TradesPage -> listTrades -> trades (embed lots, participants) [+ findParticipantByCode] -> TradeFilter + DataTable -> StatusBadge
```

### Read Calls *(mục bổ sung)*

| Thao tác | Cách gọi | Tham số | Trả về |
|---|---|---|---|
| Tra mã người tham gia (khi lọc `buyerCode`) | `select` từ `participants` qua `findParticipantByCode()` | `code` | `participant.id` hoặc `null` |
| Đọc danh sách giao dịch | `select` từ `trades` kèm embed PostgREST `lots(lot_code)` + `participants(code, name)` qua `listTrades()` | `businessDate?`, `buyer_participant_id?` (suy từ `buyerCode`), `status?`, `tradeType?` | mảng `TradeRow` |

Select thẳng, dùng embed quan hệ PostgREST — khác `lib/domain/accounts.ts` (`SCR004`) vốn tránh embed vì có 2 FK trùng bảng; ở đây `trades.lot_id`/`trades.buyer_participant_id` trỏ tới 2 bảng khác nhau nên không nhập nhằng tên ràng buộc.

### Dependencies & Risks *(mục bổ sung)*

- **Phụ thuộc migration `0013` (sai lệch D-05, (tài liệu gốc trong repo, không nộp) `database-diagram.md`):** RLS bảng `trades` mở SELECT thêm cho `receiving` để `v_trade_delivery_status` (bật `security_invoker`) trả đúng dữ liệu cho `SCR001` (`tradesAwaitingDelivery`) và `SCR014` — thiếu migration này, hai màn đó luôn thấy số 0/rỗng dù có dữ liệu thật. Migration này **chỉ mở tầng đọc**; `receiving` vẫn không mở được route `/trades`.
- **Dễ hiểu sai:** nhìn riêng RLS sẽ tưởng `receiving` mở được `SCR012` — phải đọc CẢ `ROUTE_POLICY` lẫn RLS.
- Cột `businessDate` hiện chuỗi thô (không format), khác `SCR003`/`SCR008` — không nhất quán nhưng không phải lỗi chức năng.
- Cột `amount` là GENERATED column của Postgres — không có đường ghi tay nào; mọi nỗ lực "sửa" field này ở tầng ứng dụng là sai (ADR-004 — bất biến ở DB; xem thêm câu hỏi `correction_entries` số âm ở `SCR022`).
- Link chéo: [F004](../../features/F004_TradeRecording/screens.md); flow FL-01; ADR-002.

### Unresolved Questions *(mục bổ sung)*

- Không phân trang — RFP nêu 700–1.200 giao dịch/ngày, danh sách không lọc sẽ tải toàn bộ; cần quyết định ngưỡng phân trang/giới hạn mặc định.
- Có nên thống nhất format ngày (`businessDate`) giữa `SCR003`/`SCR008`/`SCR012` hay chấp nhận khác biệt hiện tại?
- `listTrades` không có `.range()`/`.limit()` (`lib/domain/trades.ts:61-86`); PostgREST giới hạn số dòng trả về (`supabase/config.toml`: `max_rows = 1000`) nên một ngày 700–1.200 giao dịch có thể bị cắt **im lặng** khi không lọc. Cần phân trang hoặc cảnh báo khi chạm giới hạn.
