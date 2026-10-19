# SCR016_DeliveryDetail — Screen Spec

> Alias LAB-1: **SC-16** · 取引別 引渡し詳細 · Chi tiết giao hàng theo giao dịch

**Screen**: SCR016: DeliveryDetail (Chi tiết giao hàng theo giao dịch)
**Feature**: [F005_DeliveryTracking](../../features/F005_DeliveryTracking/screens.md) — tính năng LAB-1 `F05-03` `F05-04` `F05-06`
**Type**: composite (khối đầu trang + bảng lần giao + hộp thoại chốt hoàn tất)
**Route**: `/deliveries/trade/[tradeId]` *(đề xuất)*
**Generated**: 2026-10-06
**Trạng thái**: Thiết kế mới, chưa xây (LAB-4)
**Role**: `receiving` (xem + chốt hoàn tất) · `reconcile` (chỉ xem)
**User story**: `US-08` (chốt hoàn tất giao hàng) · hỗ trợ `US-07` (biết còn lại bao nhiêu) và `US-18` (xác nhận khi có tranh chấp).

## Purpose

Người giao hàng (`receiving`) mở **một giao dịch** để thấy đủ từng lần giao đã thực hiện, lũy kế, số còn lại — rồi **chốt hoàn tất giao hàng** khi đã giao đủ, để giao dịch đi tiếp vào đối chiếu và không bị treo.

Nút "Chốt hoàn tất" là chỗ nghiêm ngặt thứ hai của hệ thống, sau cổng chốt giao dịch: nó kiểm **đồng thời hai điều** — số lượng khớp quy tắc 精算 hiện hành (`BR-DEL-03`), **và** tư cách người nhận còn hiệu lực. Điểm dễ bỏ sót mà `US-08` nêu đích danh: quyền tham gia phải còn hạn **cả ở khâu giao hàng**, không chỉ khâu chốt giao dịch.

Màn này cũng là chỗ hiện thực `F05-06`: mở một bản đối chiếu/精算 là truy ngược được các lần giao liên quan.

## Screen Layout

Trang chi tiết trong khung ứng dụng chung: khối đầu trang (giao dịch + lũy kế + quy tắc 精算 áp dụng), nút hành động, bảng từng lần giao sắp theo `seq` tăng dần; hộp thoại "Chốt hoàn tất giao hàng" nổi phía trên. Chưa có code nên chưa có breakpoint chốt.

### Layout Sketch

```
┌──────────────────────────────────────────────────────┐
│ R1: Header ứng dụng + nav (static)                │
├──────────────────────────────────────────────────────┤
│ R2: Khối 3.1: GD · ngày · người nhận · tư cách ·     │
│     lô · tổng · đã giao · còn lại · số lần · 精算    │
│     [Ghi lần giao] [Ghi ngoại lệ] [Chốt hoàn tất]    │
├──────────────────────────────────────────────────────┤
│ R3: Bảng từng lần giao (seq tăng dần)                │
└──────────────────────────────────────────────────────┘
   ┌─ R4: Hộp thoại "Chốt hoàn tất giao hàng" (modal) ─┐
   │ checklist 3 dòng · tên người chốt · ghi chú · ☐   │
   └───────────────────────────────────────────────────┘
```

### Layout Regions

| Region ID | Name | Position | Scrollable | Key Components | Responsive Behavior |
|-----------|------|----------|------------|----------------|---------------------|
| R1 | Header + nav | static | no | khung `(app)` chung | always |
| R2 | Khối đầu trang + hành động | static | no | danh sách mô tả, badge, 3 nút | fluid |
| R3 | Bảng từng lần giao | static | yes | `DataTable`, `StatusBadge` | fluid |
| R4 | Hộp thoại chốt hoàn tất | floating (modal) | no | checklist, ô tên, textarea, checkbox | conditional |

## Child Routes

Không áp dụng — màn này là một route đơn, không phải khung chứa route con.

## User Flow

> Phạm vi: tương tác trong màn. Điều hướng liên màn xem [screen-flow](../../generated/screen-flow.md).

### Happy Path

1. Từ SCR015 bấm mã giao dịch, hoặc từ SCR013 bấm "Xem các lần giao" → `/deliveries/trade/[tradeId]`.
2. Trang hiện khối đầu trang (lũy kế, còn lại, quy tắc 精算 áp dụng) và bảng từng lần giao (sắp theo `seq` tăng dần).
3. `receiving` thấy nút "Ghi lần giao" (sang SCR014, mang sẵn mã giao dịch) và nút "Ghi ngoại lệ" (sang SCR017).
4. Khi đủ điều kiện, bấm "Chốt hoàn tất giao hàng" → hộp thoại hiện checklist 3 dòng.
5. Nhập tên người chốt, (tuỳ chọn) ghi chú, tick xác nhận → "Xác nhận chốt hoàn tất".
6. RPC `complete_trade_delivery` trong một transaction: `SELECT ... FOR UPDATE` dòng giao dịch → kiểm role (`P0005`) → kiểm ngày chốt (`P0002`) → `_assert_buyer_qualified` (`P0003`) → đọc quy tắc 精算 tại `as_of = business_date` rồi kiểm dung sai (`P0063`) → kiểm không còn lần giao `pending`/`in_transit` (`P0063` nhánh b) → ghi `trades.delivery_completed_at/_by/_by_name/_settlement_rule_version` → ghi `audit_log`.
7. Thành công → khối đầu trang đổi badge; thất bại → hộp thoại giữ nguyên, banner nêu đúng điều kiện nào trượt.

### Branches

| Decision point | Condition | Outcome on this screen | Source |
|----------------|-----------|------------------------|--------|
| Bước 4 | giao dịch chưa có lần giao nào | bảng "Chưa có lần giao nào."; nút chốt hoàn tất `disabled` | thiết kế LAB-4 |
| Bước 4 | ngày đã chốt sổ hoặc đã chốt hoàn tất rồi | nút chốt ẩn hẳn; bảng vẫn xem được đầy đủ | thiết kế LAB-4 |
| Bước 6 | `P0002`/`P0003`/`P0005`/`P0063` | hộp thoại không đóng, banner đỏ nêu điều kiện trượt | thiết kế LAB-4 |
| Bước 3 | role `reconcile` | không có 3 nút hành động | thiết kế LAB-4 |

## Data Inventory

> Chỉ field chỉ đọc. Các ô nhập của hộp thoại chốt hoàn tất nằm ở `## Validation & Error Feedback`.

### Khối đầu trang — giao dịch và lũy kế

| Field | Display Label | Source | Format | Empty Behavior | Cross-ref |
|-------|---------------|--------|--------|----------------|-----------|
| `tradeCode` | Mã giao dịch / 取引番号 | `trades.trade_code` | link → SCR013 | luôn có | trades |
| `businessDate` · `dayStatus` | Ngày nghiệp vụ · trạng thái / 業務日・状態 | `trades.business_date`, `business_days.status` | date + badge; `closed` ⇒ ẩn nút chốt hoàn tất | luôn có | trades, business_days |
| `buyer` | Người nhận / 買受人 | `participants` theo `trades.buyer_participant_id` | `{name} ({code})` + link → SCR006 | luôn có | participants |
| `buyerQualification` | Hiệu lực tư cách / 資格の有効性 | `participant_qualifications` | badge: `active` ∧ (`valid_to` NULL ∨ `valid_to ≥ current_date`) → "Còn hiệu lực" (NULL = không thời hạn, đề xuất G9) (positive); ngược lại "Hết hiệu lực từ {valid_to}" (danger) — **chỉ hiển thị**, phép chặn thật nằm trong RPC | luôn có | participant_qualifications |
| `lotCode` · `commodity` | Lô · mặt hàng / ロット・品目 | `lots`, `commodities` | link → SCR009 | luôn có | lots |
| `quantity` | Tổng số lượng giao dịch / 取引総数量 | `v_trade_delivery_status.quantity` | number + đơn vị | luôn có | v_trade_delivery_status |
| `deliveredQty` | Lũy kế đã giao / 引渡し済み（累積） | `v_trade_delivery_status.delivered_qty` | `= Σ deliveries.quantity WHERE status IN ('completed','exception')` — lần giao ngoại lệ **vẫn tính vào** vì là số đã thực giao | 0 | v_trade_delivery_status |
| `remaining` | Còn lại / 残数量 | `v_trade_delivery_status.remaining` | `= quantity − delivered_qty`, **tính mỗi lần đọc**, không phải cột lưu sẵn (ADR-003) | `= quantity` | v_trade_delivery_status |
| `deliveryCount` | Số lần đã giao / 引渡し回数 | `v_trade_delivery_status.delivery_count` | `≥ 3` ⇒ banner hổ phách (ngưỡng giả định, như SCR014) | 0 | v_trade_delivery_status |
| `settlementRule` | Quy tắc 精算 áp dụng / 適用精算ルール | bảng quy tắc có phiên bản *(SCR037, đề xuất)* | `{mã version} · hiệu lực từ {発効日}`; tra theo `as_of = trades.business_date`, **tuyệt đối không theo `now()`** (tài liệu gốc trong repo, không nộp: `architecture.md` §3.1) | — | rule versions |

### Bảng từng lần giao (một dòng = một `deliveries`)

| Field | Display Label | Source | Format | Empty Behavior | Cross-ref |
|-------|---------------|--------|--------|----------------|-----------|
| `seq` | Lần thứ / 回次 | `deliveries.seq` | number; duy nhất theo giao dịch (`UNIQUE (trade_id, seq)`) | — | deliveries |
| `quantity` | Số lượng lần này / 今回の数量 | `deliveries.quantity` | number `> 0` | — | deliveries |
| `runningTotal` | Lũy kế tới lần này / 当該回までの累積 | tính ra | `= Σ quantity của các lần có seq ≤ seq hiện tại` (tầng ứng dụng khi dựng bảng) | — | computed |
| `status` | Trạng thái lần giao / 回次の状態 | `deliveries.status` | badge enum `delivery_status`: `pending` 待機中 · `in_transit` 引渡し中 · `completed` 完了 · `exception` 例外 | — | deliveries |
| `exceptionType` | Loại ngoại lệ / 例外種別 | `deliveries.exception_type` | badge enum `delivery_exception`: `short` 引渡不足 · `over` 引渡超過 · `returned` 返却 · `partial_cancel` 一部取消; `NULL` khi `status ≠ 'exception'` (CHECK ở database) | "—" | deliveries |
| `reason` | Lý do / 理由 | `deliveries.reason` | text; bắt buộc khác rỗng khi `status='exception'` | "—" | deliveries |
| `deliveredAt` | Thời điểm giao / 引渡し日時 | `deliveries.delivered_at` | datetime | — | deliveries |
| `confirmedBy` | Người xác nhận / 確認者 | `app_users.full_name` theo `deliveries.confirmed_by` | text | — | app_users |

### Hộp thoại chốt hoàn tất — vùng chỉ đọc

| Field | Display Label | Source | Format | Empty Behavior | Cross-ref |
|-------|---------------|--------|--------|----------------|-----------|
| `checklist` | Điều kiện phải đạt / 完了条件 | tính ra | 3 dòng ✓/✗: (a) `|remaining| ≤ dung_sai` của quy tắc 精算 tại `as_of = business_date` — **mặc định `dung_sai = 0`** cho tới khi khách chốt; (b) tư cách người nhận còn hiệu lực tại `current_date`; (c) không còn lần giao nào ở `pending`/`in_transit` | — | computed |

### Cột ghi hoàn tất giao hàng trên `trades` *(cột mới, đề xuất — chốt ở PREMISE G3)*

Hoàn tất giao hàng ghi ở **cấp giao dịch**. Bộ cột này là nguồn chuẩn; `F005_DeliveryTracking/technical-spec.md` phải khớp:

| Cột | Ý nghĩa | Ghi khi |
|---|---|---|
| `trades.delivery_completed_at` | thời điểm chốt hoàn tất; `NULL` = chưa hoàn tất | RPC `complete_trade_delivery` |
| `trades.delivery_completed_by` | tài khoản gọi RPC (`app_users.id`) | cùng RPC |
| `trades.delivery_completed_by_name` | tên người chốt tại hiện trường (nhập tay ở hộp thoại) | cùng RPC |
| `trades.delivery_completed_settlement_rule_version` | phiên bản quy tắc 精算 dùng để kiểm dung sai | cùng RPC |

### Read Calls *(mục bổ sung)*

| Thao tác | Cách gọi | Tham số | Trả về |
|---|---|---|---|
| Đọc giao dịch + lũy kế | `select` từ `trades` ⨝ `v_trade_delivery_status` kèm embed `lots`, `participants` | `tradeId` | dòng theo khối đầu trang hoặc `null` → `notFound()` |
| Đọc hiệu lực tư cách người nhận | `select` từ `participant_qualifications` | `participant_id` | trạng thái + `valid_to` — **chỉ hiển thị** |
| Đọc từng lần giao | `select` từ `deliveries` lọc `trade_id`, sắp `seq` tăng dần, + truy vấn thứ hai vào `app_users` tra tên người xác nhận | `trade_id` | mảng theo bảng lần giao |
| Đọc quy tắc 精算 áp dụng | `select` từ bảng quy tắc có phiên bản *(SCR037, đề xuất)* với `as_of = trades.business_date` | `business_date` | `rule_version`, `tolerance`, `発効日` |

## UI States

| State | Trigger | Visual Behavior | User Action Available | Source |
|-------|---------|----------------|-----------------------|--------|
| empty | giao dịch chưa có lần giao nào | bảng "Chưa có lần giao nào." / 引渡し記録はまだありません。; nút chốt hoàn tất `disabled` | Ghi lần giao | thiết kế LAB-4 |
| loading / saving | sau khi bấm "Xác nhận chốt hoàn tất" | nút `disabled`, nhãn "Đang chốt…" | none | thiết kế LAB-4 |
| success | RPC trả về | hộp thoại đóng; banner xanh `role="status"` "Đã chốt hoàn tất giao hàng cho {trade_code}."; khối đầu trang hiện badge "Đã hoàn tất" + người/thời điểm chốt | dismiss | thiết kế LAB-4 |
| error (nghiệp vụ) | `P0002`/`P0003`/`P0005`/`P0063` | hộp thoại **không đóng**, banner đỏ `role="alert"` tự nhận focus, dữ liệu giữ nguyên | sửa / đóng | thiết kế LAB-4 |
| error (mạng) | Server Action throw | như trên, không điều hướng, không mất dữ liệu | thử lại | thiết kế LAB-4 |
| read-only | ngày đã chốt sổ **hoặc** đã chốt hoàn tất rồi | nút ẩn hẳn; bảng lần giao vẫn xem được đầy đủ (nguồn truy ngược của `F05-06`, không được khoá đọc) | xem | thiết kế LAB-4 |

## Validation & Error Feedback

### A) Client-side

| Field | Type | Required | Constraints | Async Check | Error Message |
|-------|------|----------|-------------|-------------|---------------|
| `completedByName` / 完了確認者 | text | yes | `2 ≤ độ dài ≤ 100`; tên người tại hiện trường, **khác** tài khoản đăng nhập (máy dùng chung theo ca); lưu `trades.delivery_completed_by_name` *(cột mới, đề xuất)* | không | `V-name` (xem Error Catalogue) |
| `note` / メモ | textarea | no | `≤ 1000` ký tự; ghi vào `audit_log` (`reason`) | không | — |
| `confirm` / 引渡し完了を確認します ("Tôi xác nhận đã giao đủ") | checkbox | yes | phải `true` mới bật nút gửi; mặc định tắt | không | `V-confirm` (xem Error Catalogue) |

### B) Server-side

#### Chốt hoàn tất giao hàng
- **Endpoint:** `rpc('complete_trade_delivery')` *(mới, đề xuất)* qua Server Action
- **Request:** `p_trade_id, p_completed_by_name, p_note`
- **Success:** `jsonb {trade_code, completed_at, rule_version}` → đóng hộp thoại, banner xanh, badge "Đã hoàn tất"
- **Errors:** `P0001` · `P0002` · `P0003` · `P0005` · `P0063` (chi tiết ở Error Catalogue)
- **Trigger:** nút "Xác nhận chốt hoàn tất"
- **Source:** chưa có code — màn chỉ đặc tả ở LAB-4

Ghi **bắt buộc qua RPC**: `trades` không có policy ghi qua RLS, và phép kiểm tư cách + dung sai phải chạy trên trạng thái máy chủ trong một transaction có khoá dòng.

#### Error Catalogue *(mục bổ sung)*

| Mã | Điều kiện kích hoạt | Thông báo VI | Thông báo JA | Hiện ở đâu | Chặn gửi? |
|---|---|---|---|---|:---:|
| `V-name` | `completedByName` rỗng hoặc < 2 ký tự | Vui lòng nhập tên người chốt hoàn tất. | 完了確認者を入力してください。 | dưới ô trong hộp thoại | ● |
| `V-confirm` | chưa tick ô xác nhận | Vui lòng tick ô xác nhận trước khi chốt. | 確認チェックを入れてください。 | dưới ô | ● |
| `P0001` | `tradeId` không có trong `trades.id` | Không tìm thấy giao dịch. | 該当する取引が見つかりません。 | trang 404 (`notFound()`) | ● |
| `P0002` | ngày nghiệp vụ đã chốt sổ (kiểm **trong RPC**) | Ngày nghiệp vụ {business_date} đã chốt sổ lúc {closed_at}, không chốt hoàn tất thêm được. Lập phiếu điều chỉnh (SCR022). | 業務日{business_date}は{closed_at}に締められており、完了確認を行えません。訂正申請（SCR022）をご利用ください。 | banner trong hộp thoại | ● |
| `P0003` | tư cách người nhận hết hiệu lực — `details: {expired_on, basis_type, basis_label, contact_office}` | Người nhận {buyer_name} đã hết hạn tham gia từ {expired_on}, căn cứ {basis_label}. Liên hệ {contact_office} để xử lý. | 買受人{buyer_name}は{expired_on}から資格が失効しています（根拠：{basis_label}）。{contact_office}にご連絡ください。 | banner | ● |
| `P0005` | role ≠ `receiving` | Chức danh của bạn không được phép chốt hoàn tất giao hàng. | あなたの職掌では引渡し完了を確認できません。 | banner | ● |
| `P0063` *(mới)* | số lượng chưa khớp quy tắc 精算 — `details: {quantity, delivered, remaining, tolerance, rule_version}` | Đã giao {delivered}/{quantity} {unit}, còn lại {remaining} {unit} — vượt dung sai {tolerance} {unit} của quy tắc 精算 {rule_version}. Chưa chốt hoàn tất được. | {quantity} {unit}のうち{delivered} {unit}引渡し済み、残り{remaining} {unit}です — 精算ルール{rule_version}の許容差{tolerance} {unit}を超えています。完了確認できません。 | banner | ● |
| `P0063` (nhánh b) | còn lần giao ở `pending` hoặc `in_transit` — `details: {open_seqs[]}` | Lần giao thứ {open_seqs} chưa kết thúc. Cập nhật trạng thái các lần đó trước khi chốt. | 第{open_seqs}回の引渡しが未完了です。先に状態を更新してください。 | banner | ● |
| (đã chốt) | `trades.delivery_completed_at` đã khác `NULL` | Giao hàng của giao dịch này đã được chốt hoàn tất lúc {completed_at}. | この取引の引渡しは{completed_at}に完了確認済みです。 | banner; trang tự nạp lại | ● |
| (network) | Server Action throw | Mất kết nối — chưa chốt được. Thông tin đã nhập vẫn giữ nguyên. | 通信が切断され、完了確認できませんでした。入力内容は保持されています。 | banner | ● |

**Then chốt:** ba điều kiện ở `checklist` chỉ **hiển thị** để người dùng biết trước; cổng chặn thật là RPC. Không được biến checklist thành điều kiện `disabled` duy nhất của nút — cùng lý lẽ "Tiến độ giao hàng chỉ tham khảo" của SCR014. Nút có thể `disabled` theo checklist cho đỡ bấm nhầm, nhưng RPC vẫn phải kiểm lại đủ ba điều.

## Interaction Patterns

- **Hộp thoại đóng bằng `Escape` và trả focus về nút đã mở nó** — thiết kế LAB-4.
- **Mọi hành động trên từng dòng của bảng lần giao đều có đường bàn phím (người dùng đeo găng tay)** — thiết kế LAB-4. Không có yêu cầu phím tắt riêng.
- **Nút "Ghi lần giao" chuyển sang SCR014 mang sẵn mã giao dịch** — thiết kế LAB-4.

## Accessibility

| Aspect | Status | Notes |
|--------|--------|-------|
| ARIA roles/labels | partial (đặc tả) | banner thành công `role="status"`, lỗi `role="alert"`; checklist ✓/✗ cần chữ kèm dấu |
| Keyboard navigation | supported (đặc tả) | mọi hành động có đường bàn phím; `Escape` đóng hộp thoại |
| Focus management | managed (đặc tả) | trả focus về nút mở hộp thoại; banner lỗi tự nhận focus |
| Screen reader compatibility | unknown | chưa có bản dựng để kiểm |

## Conditional Rendering

| Condition | Type | Renders | Hidden | Notes |
|-----------|------|---------|--------|-------|
| role = `receiving` | auth | nút "Ghi lần giao", "Ghi ngoại lệ", "Chốt hoàn tất" | — | bypass: Server Action `P0005` + RPC vẫn chặn |
| role = `reconcile` | auth | chỉ phần xem | 3 nút hành động | đối chiếu không tự chốt hoàn tất thay bộ phận giao hàng |
| ngày `closed` hoặc đã chốt hoàn tất | hardcoded-id | badge "Đã hoàn tất"/dải chốt sổ | nút chốt hoàn tất | ẩn hẳn |
| giao dịch chưa có lần giao | hardcoded-id | "Chưa có lần giao nào." | — | nút chốt `disabled` |
| `deliveryCount ≥ 3` | hardcoded-id | banner hổ phách | — | ngưỡng giả định [NEEDS_DOMAIN_CONFIRMATION] |

## Component Variants

Không áp dụng — màn không có component đổi hình dạng theo biến thể; khác biệt theo chức danh/trạng thái đã mô tả ở Conditional Rendering.

## Security Surface

| Guard | Type | Consequence if bypassed |
|-------|------|------------------------|
| `ROUTE_POLICY` — cùng cấu hình SCR015: `/deliveries` → `["receiving","reconcile"]`, kèm mục khớp chính xác `/deliveries/new` → `["receiving"]` xét trước | role | role sai mở được màn |
| Server Action: `role !== "receiving"` → `P0005` | role | tầng hai chặn chốt |
| RPC `complete_trade_delivery` kiểm `current_app_role()` + ngày chốt + tư cách + dung sai | permission | tầng ba; `[UNVERIFIED]` — RPC chưa tồn tại, đây là yêu cầu thiết kế |

Quyền theo chức danh (9 role, PREMISE C-09): `sys_admin` ✗ · `receiving` xem + ghi (chốt hoàn tất; gồm cả 配送部門, PREMISE C-10) · `trading` ✗ (LAB-1 không liệt kê) · `reconcile` chỉ xem · 5 role đề xuất (`party_admin`, `rule_admin`, `ops_admin`, `market_manager`, `internal_audit`) ✗. `can_approve` không liên quan (chốt hoàn tất không phải thao tác duyệt maker-checker).

## Source References

Chưa có code — màn chỉ đặc tả ở LAB-4. Tài liệu nguồn đã đọc (mục có chú thích "không nộp" là tài liệu gốc trong repo):

1. (tài liệu gốc trong repo, không nộp) `sc-16-chi-tiet-giao-hang.md` — spec 10 mục gốc
2. `lib/auth/route-policy.ts:17-27` — `ROUTE_POLICY` hiện có (prefix `/deliveries` đã phủ cả `/deliveries/trade/[tradeId]`)
3. `supabase/migrations/0007_rpc_functions.sql:114-166` — `record_delivery` (RPC liên quan, đã xây)

## Source Walkthrough

Chưa có code — màn chỉ đặc tả ở LAB-4.

### Call Hierarchy

```text
(thiết kế) /deliveries/trade/[tradeId] page -> trades + v_trade_delivery_status + deliveries (đọc)
   -> Server Action chốt -> rpc complete_trade_delivery (đề xuất)
```

## Dependencies & Risks *(mục bổ sung)*

- Phụ thuộc SCR014 (có lần giao), SCR015 (đường vào), và bảng quy tắc 精算 có phiên bản của SCR037. **Không có quy tắc tại `as_of` thì không chốt hoàn tất được** — đây là phụ thuộc chặn, không phải phụ thuộc mềm.
- **Khuyết tật của prototype phải sửa trước khi làm màn này:** `record_delivery` hiện tự đặt `lots.status = 引渡し完了` khi `remaining = 0` (SCR014 User Flow bước 8). Nhưng `lots.status` là trạng thái **của lô**, còn một lô có thể bán cho **nhiều giao dịch** — giao đủ cho giao dịch thứ nhất sẽ lật trạng thái cả lô trong khi giao dịch thứ hai chưa giao gì. Trạng thái "đã giao xong" phải ở **cấp giao dịch** (`trades.delivery_completed_at`, cột mới), và `lots.status = 引渡し完了` chỉ được đặt khi **mọi** giao dịch `confirmed` của lô đó đã chốt hoàn tất.
- **Chồng chéo quyền:** tự động chuyển `引渡し完了` khi `remaining = 0` (prototype) **mâu thuẫn trực tiếp** với `F05-04`, vốn đòi một bước xác nhận của người có kèm kiểm tư cách. Giữ cả hai thì phép kiểm `P0003` bị vô hiệu hoá trên thực tế vì trạng thái đã tự chuyển từ trước. Hướng đã chốt (PREMISE G3): bỏ nhánh tự chuyển trong `record_delivery`; `lots.status = 引渡し完了` chỉ đặt khi mọi giao dịch `confirmed` của lô đã hoàn tất như gạch đầu dòng trên.
- **Rủi ro hiểu sai:** `delivered_qty` đã gồm cả lần giao `exception` — nên `remaining = 0` **không** có nghĩa "giao đúng đủ", mà là "tổng số đã thực giao bằng số giao dịch". Giao thiếu rồi ghi ngoại lệ bù vẫn có thể cho `remaining = 0`. Vì vậy checklist `checklist` cần cả điều kiện (c), không chỉ (a).
- Phép kiểm tư cách dùng `current_date` tại thời điểm gọi RPC (khuôn `_assert_buyer_qualified` của (tài liệu gốc trong repo, không nộp) `db-schema.md` §2), **không** dùng `business_date` — giữ nguyên để nhất quán với SCR010/SCR014, nhưng xem Unresolved Questions.

### Unresolved Questions *(mục bổ sung)*

- **"Khớp quy tắc 精算" nghĩa là gì?** `BR-DEL-03` không định lượng; RFP ghi nhận đây là một trong các điểm **chặn estimate** (`FR-DEL-05`, `EX-DEL-02`, B.8). Tài liệu này giả định `dung_sai = 0` (phải giao đúng đủ). Nếu khách cho dung sai theo tỷ lệ hoặc theo mặt hàng thì `P0063` và `checklist` phải viết lại.
- Tư cách người nhận xét theo `current_date` hay theo `business_date` của giao dịch? Giao hàng kéo dài qua ngày là bình thường (ADR-005), nên hai mốc này sẽ lệch nhau trong thực tế — khách cần chọn.
- Chốt hoàn tất rồi có **mở lại** được không khi phát hiện sai? Tài liệu này chọn không (đi qua F07); cần xác nhận vì nó định hình luôn ma trận quyền của SCR022.
- Ai được chốt hoàn tất khi đang có tranh chấp mở (SCR019/SCR020)? Chặn cứng, hay cho chốt kèm cảnh báo?
