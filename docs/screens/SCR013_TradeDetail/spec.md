# SCR013_TradeDetail — Screen Spec

> Alias LAB-1: **SC-13** · 取引詳細 · Chi tiết giao dịch

**Screen**: SCR013: TradeDetail (Chi tiết giao dịch)
**Feature**: [F004_TradeRecording](../../features/F004_TradeRecording/screens.md) — tính năng LAB-1 `F04-03` `F04-04` `F04-06`
**Type**: composite (khối số liệu + hộp thoại huỷ + hai tab)
**Route**: `/trades/[id]` *(đề xuất)*
**Generated**: 2026-10-06
**Trạng thái**: Thiết kế mới, chưa xây (LAB-4)
**Role**: `trading` (xem + huỷ) · `reconcile` (chỉ xem)
**User story**: `US-20` (tra lại một giao dịch cũ và thấy đủ ai làm gì). Huỷ giao dịch không có US riêng — truy về `FR-AITAI-03`.

## Purpose

Nhân viên giao dịch (`trading`) kiểm lại ngay sau khi chốt, và bộ phận đối chiếu (`reconcile`) mở để giải thích một con số của ngày hôm trước: xem đầy đủ số liệu một giao dịch, **huỷ giao dịch khi ngày nghiệp vụ chưa chốt sổ** (kèm lý do bắt buộc) và xem tab "Lịch sử sửa" gồm số cũ · số mới · lý do (`F04-06`, `FR-SERI-03`).

Ranh giới cứng của màn này: **sau khi ngày nghiệp vụ đã chốt sổ, không có đường sửa nào ở đây** — mọi thay đổi đi qua phiếu điều chỉnh có duyệt (F007, SCR022 → SCR024). Nút huỷ biến mất, không chỉ bị `disabled`.

## Screen Layout

Trang chi tiết một cột trong khung ứng dụng chung (header + nav theo chức danh): đầu trang là tiêu đề kèm mã giao dịch và các badge; khối số liệu chỉ đọc; khối tiến độ giao hàng có liên kết sang SCR016; dưới cùng là hai tab "Lần giao" · "Lịch sử sửa". Nút "Huỷ giao dịch" mở hộp thoại (modal) nổi phía trên trang. Chưa có code nên chưa có breakpoint chốt — (đề xuất) bố cục fluid như SCR012.

### Layout Sketch

```
┌──────────────────────────────────────────────────────┐
│ R1: Header ứng dụng + nav (static)                │
├──────────────────────────────────────────────────────┤
│ R2: Tiêu đề · mã GD · badge loại/trạng thái/ngày     │
│     [Huỷ giao dịch]  (chỉ khi confirmed + ngày open) │
│     - - - dải "Ngày nghiệp vụ đã chốt sổ" - - -      │
├──────────────────────────────────────────────────────┤
│ R3: Khối số liệu giao dịch (3.1, chỉ đọc)            │
├──────────────────────────────────────────────────────┤
│ R4: Tiến độ giao hàng → link SCR016                  │
├──────────────────────────────────────────────────────┤
│ R5: Tab [Lần giao] [Lịch sử sửa]                     │
└──────────────────────────────────────────────────────┘
   ┌─ R6: Hộp thoại "Huỷ giao dịch" (modal) ─┐
   │ ảnh hưởng · lý do · gõ lại mã · [Huỷ][Xác nhận huỷ] │
   └──────────────────────────────────────────┘
```

### Layout Regions

| Region ID | Name | Position | Scrollable | Key Components | Responsive Behavior |
|-----------|------|----------|------------|----------------|---------------------|
| R1 | Header + nav | static | no | khung `(app)` chung | always |
| R2 | Tiêu đề + hành động | static | no | `PageHeader`, `StatusBadge`, nút huỷ, dải chốt sổ | fluid |
| R3 | Số liệu giao dịch | static | yes | danh sách mô tả (dòng 1–12) | fluid |
| R4 | Tiến độ giao hàng | static | yes | dòng 13 + link SCR016 | fluid |
| R5 | Tab Lần giao / Lịch sử sửa | static | yes | bảng lần giao, bảng `audit_log` | fluid |
| R6 | Hộp thoại huỷ | floating (modal) | no | textarea lý do, ô gõ lại mã, ảnh hưởng dự kiến | conditional (mở khi bấm huỷ) |

## Child Routes

Không áp dụng — màn này là một route đơn, không phải khung chứa route con.

## User Flow

> Phạm vi: tương tác trong màn. Điều hướng liên màn xem [screen-flow](../../generated/screen-flow.md).

### Happy Path

1. Từ SCR012 bấm vào mã giao dịch → `/trades/[id]`.
2. Trang hiện khối số liệu (R3), khối tiến độ giao hàng (R4, dòng 13) với liên kết sang SCR016, và hai tab "Lần giao" · "Lịch sử sửa" (R5).
3. **Nhánh huỷ** (chỉ `trading`, chỉ khi ngày chưa chốt và trạng thái còn `confirmed`): bấm "Huỷ giao dịch" → hộp thoại R6.
4. Hộp thoại hiện ảnh hưởng dự kiến (dòng 16), yêu cầu lý do + gõ lại mã giao dịch.
5. Bấm "Xác nhận huỷ" → Server Action kiểm role → `rpc('cancel_trade', ...)`.
6. RPC trong một transaction: `SELECT ... FOR UPDATE` dòng giao dịch → kiểm role (`P0005`) → kiểm ngày chốt (`P0002`) → kiểm **không tồn tại dòng giao hàng nào** của giao dịch: `exists(select 1 from deliveries where trade_id = …)` (`P0062`) — mọi `deliveries.status`, không dùng `v_trade_delivery_status.delivery_count` làm điều kiện chặn → `UPDATE trades` đặt `status='cancelled'`, `cancel_reason`, `cancelled_at`, `cancelled_by` → ghi `audit_log` (`action='cancel'`, `old_value`/`new_value`/`reason`).
7. **Số còn lại của lô tự hồi** — không có bước hoàn số lượng nào: `v_lot_availability` chỉ cộng `trades.quantity` của dòng `status='confirmed'`, nên dòng vừa chuyển `cancelled` rơi ra khỏi phép tính ngay lần đọc kế tiếp (ADR-003).
8. Thất bại → hộp thoại giữ nguyên, banner đỏ nêu đúng lý do và lối đi thay thế (SCR022 hoặc SCR017).

### Branches

| Decision point | Condition | Outcome on this screen | Source |
|----------------|-----------|------------------------|--------|
| Bước 3 | `business_days.status='closed'` hoặc `trades.status='cancelled'` | nút huỷ ẩn hẳn; nếu do chốt sổ hiện dải "Ngày nghiệp vụ đã chốt sổ — mọi điều chỉnh đi qua SCR022" | thiết kế LAB-4 |
| Bước 3 | role `reconcile` | không có nút huỷ (chỉ xem) | thiết kế LAB-4 |
| Bước 6 | `P0002` / `P0005` / `P0062` | hộp thoại không đóng, banner đỏ kèm liên kết SCR022 / SCR017 | thiết kế LAB-4 |
| Bước 6 | giao dịch vừa bị người khác huỷ | banner "đã được huỷ lúc {cancelled_at}", trang tự nạp lại | thiết kế LAB-4 |

## Data Inventory

> Chỉ field chỉ đọc. Ô nhập của hộp thoại huỷ nằm ở `## Validation & Error Feedback`. Cross-ref bảng `trades` — `entities.md`.

| Field | Display Label | Source | Format | Empty Behavior | Cross-ref |
|-------|---------------|--------|--------|----------------|-----------|
| `tradeCode` | Mã giao dịch / 取引番号 | `trades.trade_code` | text; duy nhất (`UNIQUE`) | luôn có | trades |
| `tradeType` | Loại / 種別 | `trades.trade_type` | badge, enum `negotiated` (相対) · `auction` (せり) | luôn có | trades |
| `status` | Trạng thái / 状態 | `trades.status` | badge enum `trade_status`: `confirmed` (positive) · `cancelled` (danger) — **chỉ 2 giá trị**, xem Dependencies and Risks | luôn có | trades |
| `businessDate` | Ngày nghiệp vụ / 業務日 | `trades.business_date` | date | luôn có | trades |
| `dayStatus` | Trạng thái ngày / 業務日の状態 | `business_days.status` | badge `open` · `closed`; `closed` ⇒ ẩn nút huỷ | luôn có | business_days |
| `lotCode` | Lô / ロット | `trades.lot_id` → `lots.lot_code` | link → SCR009 | luôn có | lots |
| `buyer` | Người mua / 買受人 | `trades.buyer_participant_id` → `participants` | `{name} ({code})` + link → SCR006 | luôn có | participants |
| `quantity` | Số lượng / 数量 | `trades.quantity` | number | luôn có | trades |
| `unitPrice` | Đơn giá / 単価 | `trades.unit_price` | number | luôn có | trades |
| `amount` | Thành tiền / 金額 | `trades.amount` | number; cột **GENERATED** `= quantity * unit_price`, không có đường ghi tay | luôn có | trades |
| `confirmedAt` / `confirmedBy` | Dấu vết hệ thống / システム記録（日時・実行者） | `trades.confirmed_at`, `trades.confirmed_by` | ai gọi API, lúc nào | luôn có | trades |
| `auctionSessionAt` / `auctionConfirmedByName` | Phiên đấu giá: thời điểm · người xác nhận / せり: 開催日時・確認者 | `trades.auction_session_at`, `.auction_confirmed_by_name` | chỉ hiện khi `auction`; `NULL` bắt buộc khi `negotiated`, khác `NULL` khi `auction` (`CONSTRAINT TRIGGER`, `P0008`) | ẩn khi `negotiated` | trades |
| `deliveredQty` / `remaining` | Đã giao · còn lại / 引渡し済み・残数量 | `v_trade_delivery_status.delivered_qty`, `.remaining` | number, **tính mỗi lần đọc**, không phải cột lưu sẵn (ADR-003) | 0 / `= quantity` | v_trade_delivery_status |
| `impactPreview` | Ảnh hưởng sau khi huỷ / 取消後の影響 | tính từ `v_lot_availability` | "Lô {lot_code} sẽ tăng số còn lại từ {remaining} lên {remaining + quantity}" — **chỉ để người dùng đọc**, không phải phép kiểm | — | v_lot_availability |
| `occurredAt` | Lúc nào / 日時 | `audit_log.occurred_at` | datetime, sắp giảm dần | — | audit_log |
| `actor` | Ai làm / 実行者 | `audit_log.actor_id` | tra bằng truy vấn **thứ hai** vào `app_users`, không embed PostgREST | — | audit_log |
| `action` | Loại thao tác / 操作種別 | `audit_log.action` | `insert` · `update` · `cancel` · `blocked_write` | — | audit_log |
| `oldValue` / `newValue` | Số cũ · Số mới / 変更前 · 変更後 | `audit_log.old_value`, `.new_value` | cặp khoá–giá trị từ `jsonb`; khoá không đổi thì **không hiện** | "—" | audit_log |
| `reason` | Lý do / 理由 | `audit_log.reason` *(đề xuất)* | `audit_log` **chưa có cột `reason`** — đọc tạm `new_value->>'reason'` | "—" | audit_log |

### Read Calls *(mục bổ sung)*

| Thao tác | Cách gọi | Tham số | Trả về |
|---|---|---|---|
| Đọc giao dịch | `select` từ `trades` + embed `lots(lot_code)` + `participants(code, name)` | `id` | `TradeDetailRow` hoặc `null` → `notFound()` |
| Đọc tiến độ giao hàng | `select` từ `v_trade_delivery_status` (chỉ để hiển thị dòng 13; **không** dùng để quyết định cho huỷ) | `trade_id` | `delivered_qty`, `remaining`, `delivery_count` |
| Đọc trạng thái ngày | `select` từ `business_days` | `business_date` | `status`, `closed_at` |
| Đọc lịch sử sửa | `select` từ `audit_log` lọc `table_name='trades' AND record_id = id`, rồi truy vấn **thứ hai** vào `app_users` tra tên người thao tác | `record_id` | mảng, mới nhất trước |

## UI States

| State | Trigger | Visual Behavior | User Action Available | Source |
|-------|---------|----------------|-----------------------|--------|
| empty (lịch sử) | giao dịch chưa có dòng `audit_log` nào (dữ liệu seed cũ) | tab lịch sử: "Chưa có thay đổi nào được ghi vết." / 変更履歴はまだありません。 | — | thiết kế LAB-4 |
| loading / saving | sau khi bấm "Xác nhận huỷ" | nút `disabled`, nhãn "Đang huỷ…" | none | thiết kế LAB-4 |
| success | RPC trả về | hộp thoại đóng; banner xanh `role="status"` "Đã huỷ giao dịch {trade_code}."; badge đổi "Đã huỷ"; tab lịch sử thêm dòng mới; khối số còn lại của lô cập nhật | dismiss | thiết kế LAB-4 |
| error (nghiệp vụ) | `P0002` / `P0005` / `P0062` | hộp thoại **không đóng**, banner đỏ `role="alert"` tự nhận focus, lý do đã gõ giữ nguyên | sửa / đóng | thiết kế LAB-4 |
| error (mạng) | Server Action throw | như trên; không điều hướng, không mất dữ liệu | thử lại | thiết kế LAB-4 |
| read-only | `business_days.status='closed'` **hoặc** `trades.status='cancelled'` | nút "Huỷ giao dịch" **ẩn hẳn** (không `disabled`); nếu do chốt sổ hiện dải "Ngày nghiệp vụ đã chốt sổ — mọi điều chỉnh đi qua SCR022" | xem | thiết kế LAB-4 |

## Validation & Error Feedback

### A) Client-side

| Field | Type | Required | Constraints | Async Check | Error Message |
|-------|------|----------|-------------|-------------|---------------|
| `cancelReason` / 取消理由 | textarea | yes | `10 ≤ độ dài ≤ 1000`; lưu `trades.cancel_reason` *(cột mới, đề xuất)* | không | `V-reason` Vui lòng nêu lý do huỷ, tối thiểu 10 ký tự. |
| `confirmCode` / 確認のため取引番号を入力 | text | yes | phải **khớp chính xác** `trades.trade_code` (phân biệt hoa thường) | không | `V-confirm` Mã giao dịch gõ lại không khớp. |

### B) Server-side

#### Huỷ giao dịch
- **Endpoint:** `rpc('cancel_trade')` *(mới, đề xuất)* qua Server Action
- **Request:** `p_trade_id, p_reason`
- **Success:** `jsonb {trade_code, cancelled_at, lot_remaining_after}` → đóng hộp thoại, banner xanh, badge "Đã huỷ"
- **Errors:** `P0001` · `P0002` · `P0005` · `P0062` (chi tiết ở Error Catalogue)
- **Trigger:** nút "Xác nhận huỷ"
- **Source:** chưa có code — màn chỉ đặc tả ở LAB-4

Huỷ **bắt buộc qua RPC**: `trades` không có policy INSERT/UPDATE/DELETE qua RLS (tài liệu gốc trong repo, không nộp: `db-schema.md` §6), và việc đổi trạng thái + ghi `audit_log` phải nằm trong cùng một transaction.

#### Error Catalogue *(mục bổ sung)*

| Mã | Điều kiện kích hoạt | Thông báo VI | Thông báo JA | Hiện ở đâu | Chặn gửi? |
|---|---|---|---|---|:---:|
| `V-reason` | `cancelReason` < 10 ký tự | Vui lòng nêu lý do huỷ, tối thiểu 10 ký tự. | 取消理由を10文字以上で入力してください。 | dưới ô trong hộp thoại | ● |
| `V-confirm` | `confirmCode` ≠ `trade_code` | Mã giao dịch gõ lại không khớp. | 入力された取引番号が一致しません。 | dưới ô | ● |
| `P0001` | `id` trong URL không có trong `trades.id` | Không tìm thấy giao dịch. | 該当する取引が見つかりません。 | trang 404 (`notFound()`) | ● |
| `P0002` | ngày nghiệp vụ đã chốt sổ (kiểm **trong RPC**) | Ngày nghiệp vụ {business_date} đã chốt sổ lúc {closed_at}. Huỷ giao dịch sau chốt sổ phải lập phiếu điều chỉnh (SCR022). | 業務日{business_date}は{closed_at}に締められています。締め後の取消は訂正申請（SCR022）から行ってください。 | banner trong hộp thoại, kèm liên kết sang SCR022 | ● |
| `P0005` | role ≠ `trading` | Chức danh của bạn không được phép huỷ giao dịch. | あなたの職掌では取引を取り消せません。 | banner | ● |
| `P0062` *(mới)* | giao dịch đã có ít nhất một lần giao: `exists(select 1 from deliveries where trade_id = …)` (không dùng `v_trade_delivery_status.delivery_count` làm điều kiện: view chỉ đếm `completed`/`exception`, `0006_derived_views.sql:35-36`, nên bỏ sót lần giao ở trạng thái khác) — `details: {delivery_count, delivered_qty}` (đếm trực tiếp từ `deliveries`) | Giao dịch này đã có {delivery_count} lần giao ({delivered_qty} {unit}). Phải xử lý ngoại lệ giao hàng (SCR017) trước khi huỷ. | この取引はすでに{delivery_count}回（{delivered_qty} {unit}）引渡し済みです。先に引渡例外（SCR017）を処理してください。 | banner trong hộp thoại, kèm liên kết sang SCR017 | ● |
| (đã huỷ) | `trades.status` đã là `cancelled` lúc gọi (ai đó vừa huỷ trước) | Giao dịch này đã được huỷ lúc {cancelled_at}. | この取引は{cancelled_at}にすでに取り消されています。 | banner; trang tự nạp lại | ● |
| (network) | Server Action throw | Mất kết nối — chưa huỷ được. Lý do đã gõ vẫn giữ nguyên. | 通信が切断され、取消できませんでした。入力した理由は保持されています。 | banner trong hộp thoại | ● |

Mọi mã đọc từ `PostgrestError.code`; `details` là chuỗi JSON phải `JSON.parse()` trước khi dựng câu.

## Interaction Patterns

- **Hộp thoại huỷ đóng bằng `Escape` và trả focus về nút đã mở nó** — thiết kế LAB-4.
- Không có yêu cầu bàn phím riêng khác (màn này không thuộc ba màn keyboard-first).
- **Lỗi nghiệp vụ trong hộp thoại tự nhận focus (`role="alert"`)** — thiết kế LAB-4.

## Accessibility

| Aspect | Status | Notes |
|--------|--------|-------|
| ARIA roles/labels | partial (đặc tả) | banner thành công `role="status"`, lỗi `role="alert"`; chưa đặc tả nhãn tab |
| Keyboard navigation | supported (đặc tả) | `Escape` đóng hộp thoại; tab order chuẩn trình duyệt |
| Focus management | managed (đặc tả) | trả focus về nút mở hộp thoại; banner lỗi tự nhận focus |
| Screen reader compatibility | unknown | chưa có bản dựng để kiểm |

## Conditional Rendering

| Condition | Type | Renders | Hidden | Notes |
|-----------|------|---------|--------|-------|
| role = `trading` và `trades.status='confirmed'` và ngày `open` | auth | nút "Huỷ giao dịch" | — | bypass: Server Action (`P0005`) và RPC `cancel_trade` vẫn chặn |
| `business_days.status='closed'` | hardcoded-id | dải "đã chốt sổ" + link SCR022 | nút huỷ | ẩn hẳn, không `disabled` |
| `trades.status='cancelled'` | hardcoded-id | badge "Đã huỷ" | nút huỷ | |
| `trade_type='auction'` | hardcoded-id | khối phiên đấu giá (dòng 12) | — | `negotiated` thì ẩn |
| role = `reconcile` | auth | toàn bộ khối chỉ đọc | nút huỷ | |

## Component Variants

Không áp dụng — màn không có component đổi hình dạng theo biến thể; khác biệt theo chức danh/trạng thái đã mô tả ở Conditional Rendering.

## Security Surface

| Guard | Type | Consequence if bypassed |
|-------|------|------------------------|
| `ROUTE_POLICY` prefix `/trades` = `["trading","reconcile"]` (khớp sẵn `/trades/[id]`, **không cần đổi route policy**) | role | `receiving` bị `/403` dù RLS `trades` đã mở SELECT từ migration `0013` — giữ cách tách hai tầng của SCR012; `sys_admin` không qua |
| RLS `trades` SELECT = `trading` + `reconcile` (+ `receiving` do `0013`) | permission | đọc ngoài chức danh bị RLS chặn |
| Server Action: `currentUser.role !== "trading"` → `P0005` | role | tầng hai chặn huỷ |
| RPC `cancel_trade` tự kiểm role + ngày chốt + `exists(deliveries)` (P0062) | permission | tầng ba; `[UNVERIFIED]` — RPC chưa tồn tại, đây là yêu cầu thiết kế |
| `audit_log` RLS SELECT hiện **chỉ `sys_admin`** | permission | tab "Lịch sử sửa" trả rỗng cho `trading`/`reconcile` nếu không mở quyền (xem Unresolved Questions) |

Quyền theo chức danh (9 role, PREMISE C-09): `sys_admin` ✗ · `receiving` ✗ (route) · `trading` xem + ghi (huỷ chưa chốt sổ) · `reconcile` chỉ xem — muốn sửa thì lập phiếu F007 · 5 role đề xuất (`party_admin`, `rule_admin`, `ops_admin`, `market_manager`, `internal_audit`) ✗. `can_approve` không liên quan: huỷ trước chốt sổ không cần duyệt (duyệt chỉ ở phiếu điều chỉnh sau chốt sổ, `SCR024`).

## Source References

Chưa có code — màn chỉ đặc tả ở LAB-4. Tài liệu nguồn đã đọc (mục có chú thích "không nộp" là tài liệu gốc trong repo):

1. (tài liệu gốc trong repo, không nộp) `sc-13-chi-tiet-giao-dich.md` — spec 10 mục gốc
2. (tài liệu gốc trong repo, không nộp) `adr.md` — ADR-003 (số còn lại là view dẫn xuất)
3. (tài liệu gốc trong repo, không nộp) `db-schema.md` — enum, policy, migration
4. `lib/auth/route-policy.ts:17-27` — `ROUTE_POLICY` hiện có (prefix `/trades` đã phủ `/trades/[id]`)

## Source Walkthrough

Chưa có code — màn chỉ đặc tả ở LAB-4.

### Call Hierarchy

```text
(thiết kế) /trades/[id] page -> trades + v_trade_delivery_status + audit_log (đọc)
                             -> Server Action huỷ -> rpc cancel_trade (đề xuất)
```

## Dependencies & Risks *(mục bổ sung)*

- Phụ thuộc SCR010/SCR011 để có giao dịch; phụ thuộc SCR018 (chốt sổ) quyết định màn này còn ghi được hay không.
- **Lệch giữa LAB-1 và prototype — phải quyết trước khi code:** `F04-04` nói 5 trạng thái giao dịch (nháp → chờ xác nhận → đã chốt → huỷ hoặc cải chính), nhưng enum `trade_status` chỉ có **2** giá trị (`confirmed`, `cancelled`) và `FR-AITAI-02` cấm tạo bản ghi ở trạng thái chờ ("không tạo rồi huỷ, không tạo ở trạng thái chờ — không tạo gì cả"). Hai tài liệu mâu thuẫn. Tài liệu này giữ 2 trạng thái của prototype vì nó là hệ quả trực tiếp của một yêu cầu P0; "cải chính" thuộc F07 và sinh **bản ghi mới**, không phải một trạng thái của dòng cũ.
- **Rủi ro hiểu sai nghiêm trọng:** huỷ giao dịch **không** cần (và không được) cập nhật bất kỳ cột tồn kho nào. Nếu LAB-5 tách ra task "hoàn số lượng về lô khi huỷ" là sai kiến trúc — số còn lại là view dẫn xuất.
- `audit_log` hiện **thiếu cột `reason`** so với `FR-AUDIT-01` (đòi đủ 5 trường: ai · lúc nào · số cũ · số mới · **lý do**). Tab "Lịch sử sửa" (Data Inventory) và SCR034 cùng vấp chỗ này — cần một migration thêm `reason text`, hoặc quy ước nhét lý do vào `new_value->>'reason'`. Đề xuất: thêm cột thật, vì quy ước không kiểm được.
- `audit_log` RLS SELECT hiện **chỉ `sys_admin`** — tab "Lịch sử sửa" sẽ trả rỗng cho `trading`/`reconcile` nếu không mở quyền. Cùng lớp lỗi với sai lệch D-05.

### Unresolved Questions *(mục bổ sung)*

- Giao dịch **đã có lần giao** thì huỷ thế nào? Tài liệu này chặn cứng bằng `P0062` và đẩy sang SCR017; phương án khác là cho huỷ phần chưa giao (huỷ một phần) — nhưng `delivery_exception` đã có sẵn giá trị `partial_cancel`, nên hai đường này **chồng nhau** và cần khách chọn một.
- Thêm cột `reason` vào `audit_log` hay dùng quy ước `new_value->>'reason'`? Quyết định này ảnh hưởng cả SCR034.
- Mở RLS SELECT của `audit_log` cho `trading`/`reconcile` ở mức nào — toàn bộ bảng, hay chỉ các dòng thuộc `table_name ∈ {trades, deliveries, lots}`? Mức sau an toàn hơn nhưng phải viết policy theo điều kiện.
