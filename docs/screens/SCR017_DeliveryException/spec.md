# SCR017_DeliveryException — Screen Spec

> Alias LAB-1: **SC-17** · 引渡例外の記録 · Ghi ngoại lệ giao hàng

**Screen**: SCR017: DeliveryException (Ghi ngoại lệ giao hàng)
**Feature**: [F005_DeliveryTracking](../../features/F005_DeliveryTracking/screens.md) — tính năng LAB-1 `F05-05`
**Type**: composite (form lập phiếu + danh sách chờ duyệt + khối duyệt; có bước duyệt maker-checker)
**Route**: `/deliveries/exceptions/new` (lập phiếu) · `/deliveries/exceptions` (danh sách chờ duyệt) · `/deliveries/exceptions/[id]` (duyệt) *(đề xuất)*
**Generated**: 2026-10-06
**Trạng thái**: Thiết kế mới, chưa xây (LAB-4)
**Role**: `receiving` (maker — lập phiếu) · `reconcile` có cờ `can_approve` (checker — duyệt; PREMISE C-09)
**User story**: `US-18` (xác nhận giao hàng khi có tranh chấp)

## Purpose

Nhân viên bộ phận giao hàng ở cầu cảng lập phiếu ngay lúc phát sinh, nhân viên đối chiếu có cờ `can_approve` duyệt trong giờ hành chính: ghi lại **4 loại ngoại lệ giao hàng** — giao thiếu (`引渡不足`) · giao thừa (`引渡超過`) · trả hàng (`返却`) · huỷ một phần (`一部取消`) — kèm **lý do và người xác nhận**, rồi đưa qua **bước duyệt của bộ phận đối chiếu khi có lệch số lượng**.

Nguyên tắc tuyệt đối từ khách (`CAP-HUMAN-01` điểm 4): khi có tranh chấp về số lượng, **người xác nhận, không phải hệ thống tự quyết**. Màn này chỉ ghi lại phán quyết của người cùng chứng cứ — nó không suy ra ai đúng, không tự điều chỉnh số liệu, không gợi ý loại ngoại lệ.

## Screen Layout

Ba trạng thái trang theo ba route: (1) form lập phiếu một cột (maker); (2) danh sách phiếu chờ duyệt có bộ lọc trạng thái; (3) trang duyệt hiển thị khối lập chỉ đọc, ảnh hưởng dự kiến và khối quyết định. Chưa có code nên chưa có breakpoint chốt; vì maker làm ở cầu cảng với găng tay, (đề xuất) mục tiêu chạm lớn, không affordance chỉ hiện khi hover.

### Layout Sketch

```
┌──────────────────────────────────────────────────────┐
│ R1: Header ứng dụng + nav (static)                │
├──────────────────────────────────────────────────────┤
│ R2: Banner kết quả/lỗi (role=status | alert)         │
├────────────────────────┬─────────────────────────────┤
│ R3: Form lập phiếu     │ R4: Khối duyệt (checker)    │
│  (/exceptions/new)     │  - - (/exceptions/[id]) - - │
│  loại · lần giao ·     │  khối lập chỉ đọc · ảnh     │
│  số lượng · lý do …    │  hưởng · Duyệt/Từ chối      │
├────────────────────────┴─────────────────────────────┤
│ R5: Danh sách phiếu chờ duyệt (/exceptions)          │
└──────────────────────────────────────────────────────┘
```

### Layout Regions

| Region ID | Name | Position | Scrollable | Key Components | Responsive Behavior |
|-----------|------|----------|------------|----------------|---------------------|
| R1 | Header + nav | static | no | khung `(app)` chung | always |
| R2 | Banner kết quả/lỗi | static | no | banner `role="status"`/`role="alert"` | always |
| R3 | Form lập phiếu | static | yes | nhóm radio loại ngoại lệ, select lần giao, ô số lượng, textarea | fluid, route `/new` |
| R4 | Khối duyệt | static | yes | khối chỉ đọc, radio quyết định, textarea | route `/[id]` |
| R5 | Danh sách chờ duyệt | static | yes | `DataTable`, bộ lọc `state` | route `/exceptions` |

## Child Routes

Không áp dụng — màn này là một route đơn, không phải khung chứa route con.

## User Flow

> Phạm vi: tương tác trong màn. Điều hướng liên màn xem [screen-flow](../../generated/screen-flow.md).

### Happy Path

1. Từ SCR016 bấm "Ghi ngoại lệ" (mang sẵn mã giao dịch), hoặc mở thẳng `/deliveries/exceptions/new`.
2. Chọn 1 trong 4 loại → chọn lần giao liên quan (trừ `partial_cancel`) → hệ thống hiện số dự kiến.
3. Nhập số thực tế → hệ thống tính `deltaQty` và hiện ngay dòng cảnh báo "Phiếu này sẽ cần bộ phận đối chiếu duyệt" khi `delta ≠ 0`.
4. Nhập lý do, người xác nhận tại hiện trường, thời điểm phát sinh, (tuỳ chọn) chứng cứ → "Gửi phiếu".
5. RPC `submit_delivery_exception`: kiểm role → kiểm ngày chốt (`P0002`) → kiểm hướng số lượng theo loại → **nhánh `delta = 0`**: insert thẳng dòng `deliveries` (`status='exception'`) + `audit_log`, trả `auto_applied`; **nhánh `delta ≠ 0`**: insert `delivery_exception_requests` ở `pending`, **không đụng `deliveries`**.
6. `reconcile` mở `/deliveries/exceptions`, lọc `pending`. Nút duyệt chỉ hiện khi người xem có `app_users.can_approve = true`; phiếu do chính mình lập **không hiện nút duyệt**. `reconcile` không có cờ vẫn xem được danh sách và phiếu.
7. Mở phiếu → đọc khối lập + ảnh hưởng dự kiến → chọn Duyệt / Từ chối (+ ý kiến) → gửi.
8. RPC `decide_delivery_exception` trong một transaction: kiểm role `reconcile` (`P0005`) → kiểm cờ `can_approve` **và** maker ≠ checker (cùng mã `P0065`) → kiểm ngày chốt (`P0002`) → kiểm tư cách người nhận (`P0003`) → nếu **duyệt**: insert dòng `deliveries` tương ứng (bỏ qua trần `P0006` cho `over`, xem Dependencies and Risks) và đặt `state='approved'`; nếu **từ chối**: chỉ đặt `state='rejected'` → ghi `audit_log` cả hai nhánh.
9. Số liệu ở SCR015/SCR016/SCR018 đổi **ngay sau bước 8**, không sớm hơn.

### Branches

| Decision point | Condition | Outcome on this screen | Source |
|----------------|-----------|------------------------|--------|
| Bước 3 | `deltaQty ≠ 0` | cảnh báo "cần duyệt"; gửi → `pending` | thiết kế LAB-4 |
| Bước 5 | `deltaQty = 0` | `auto_applied`, không cần duyệt | thiết kế LAB-4 |
| Bước 6 | người xem là `reconcile` nhưng không có `can_approve` | không hiện nút duyệt, chỉ xem (RPC chặn `P0065`) | thiết kế LAB-4 |
| Bước 6 | phiếu do chính checker lập | không hiện nút duyệt (và RPC chặn `P0065`) | thiết kế LAB-4 |
| Bước 7 | `decision = reject` | ý kiến người duyệt bắt buộc | thiết kế LAB-4 |
| Bước 8 | `P0002`/`P0003`/`P0005`/`P0065` | banner đỏ, dữ liệu giữ nguyên | thiết kế LAB-4 |

## Data Inventory

> Chỉ field chỉ đọc. Ô nhập của khối lập và khối duyệt nằm ở `## Validation & Error Feedback`.

| Field | Display Label | Source | Format | Empty Behavior | Cross-ref |
|-------|---------------|--------|--------|----------------|-----------|
| `expectedQty` | Số lượng dự kiến / 予定数量 | `deliveries.quantity` / `v_trade_delivery_status.remaining` | lấy từ lần giao được chọn, hoặc `= remaining` khi `partial_cancel` | — | deliveries |
| `deltaQty` | Chênh lệch / 差異 | tính ra | `= actualQty − expectedQty` cho `short`/`over`; `= −actualQty` cho `returned`/`partial_cancel`. **Khác 0 ⇒ phiếu bắt buộc qua duyệt** | — | computed |
| `requestState` | Trạng thái phiếu / 起票状態 | `delivery_exception_requests.state` | badge ∈ `pending` 承認待ち · `approved` 承認済 · `rejected` 否認 · `auto_applied` 自動反映 (khi `deltaQty = 0`) | `pending` | delivery_exception_requests *(mới, đề xuất)* |
| `requestedBy` · `requestedAt` | Người đề nghị · lúc / 起票者・起票日時 | `app_users.full_name`, `.requested_at` | text + datetime | — | app_users |
| `impactPreview` | Ảnh hưởng sau khi duyệt / 承認後の影響 | tính từ `v_trade_delivery_status` | "Lũy kế đã giao: {delivered} → {delivered + delta}; còn lại: {remaining} → {remaining − delta}" — **chỉ để đọc**, không phải phép kiểm | — | v_trade_delivery_status |

### Read Calls *(mục bổ sung)*

| Thao tác | Cách gọi | Tham số | Trả về |
|---|---|---|---|
| Tra giao dịch + lũy kế | `select` từ `trades` ⨝ `v_trade_delivery_status` | `tradeCode` | `trade_id`, `quantity`, `delivered_qty`, `remaining` |
| Đọc các lần giao để chọn `relatedSeq` | `select` từ `deliveries` lọc `trade_id` | `trade_id` | mảng `{seq, quantity, status}` |
| Đọc danh sách phiếu chờ duyệt | `select` từ `delivery_exception_requests` *(mới)* + truy vấn thứ hai vào `app_users` | `state`, `businessDate?` | mảng phiếu |

## UI States

| State | Trigger | Visual Behavior | User Action Available | Source |
|-------|---------|----------------|-----------------------|--------|
| empty | danh sách chờ duyệt không có phiếu nào | "Không có phiếu ngoại lệ nào chờ duyệt." / 承認待ちの例外申請はありません。 | — | thiết kế LAB-4 |
| loading / saving | sau khi gửi phiếu / gửi quyết định | nút `disabled`, nhãn "Đang gửi…" | none | thiết kế LAB-4 |
| success (lập, `delta = 0`) | RPC trả `state='auto_applied'` | banner xanh "Đã ghi ngoại lệ lần giao thứ {seq}. Không có lệch số lượng nên không cần duyệt."; form xoá sạch, focus về ô mã giao dịch | dismiss | thiết kế LAB-4 |
| success (lập, `delta ≠ 0`) | RPC trả `state='pending'` | banner xanh "Đã gửi phiếu ngoại lệ {request_code} tới bộ phận đối chiếu. **Số liệu giao hàng chưa thay đổi cho tới khi được duyệt.**"; form xoá sạch | dismiss | thiết kế LAB-4 |
| success (duyệt) | RPC trả về | banner xanh "Đã duyệt phiếu {request_code}. Lũy kế đã giao cập nhật thành {delivered_after}." | dismiss | thiết kế LAB-4 |
| error (nghiệp vụ) | `P0001`…`P0065` | banner đỏ `role="alert"` tự nhận focus; dữ liệu giữ nguyên | sửa | thiết kế LAB-4 |
| error (mạng) | Server Action throw | như trên; gọi thẳng Server Action trong `onSubmit` + `useTransition` + `try/catch`, không dùng `useActionState` (cùng lý lẽ SCR007/SCR014) | thử lại | thiết kế LAB-4 |
| read-only | ngày nghiệp vụ đã chốt sổ, **hoặc** phiếu đã `approved`/`rejected` | ẩn nút gửi; phiếu đã quyết định vẫn xem được đầy đủ (là chứng cứ của `US-18`) | xem | thiết kế LAB-4 |

## Validation & Error Feedback

### A) Client-side

Khối lập (maker, `receiving`):

| Field | Type | Required | Constraints | Async Check | Error Message |
|-------|------|----------|-------------|-------------|---------------|
| `tradeCode` / 取引番号 | text | yes | phải tồn tại trong `trades.trade_code`; `trades.status` phải là `confirmed`; mặc định từ SCR016 nếu đi từ đó | tra `trades.id` | `P0001` (xem Error Catalogue) |
| `exceptionType` / 例外種別 | radio, **4 lựa chọn** | yes | ∈ `short` 引渡不足 · `over` 引渡超過 · `returned` 返却 · `partial_cancel` 一部取消 (enum `delivery_exception`); **không có giá trị mặc định**, người phải chọn | không | `V-required` |
| `relatedSeq` / 対象回次 | select | yes (khi `short`/`over`/`returned`) | phải tồn tại trong `deliveries.seq` của giao dịch đó; `partial_cancel` áp cho phần **chưa giao** nên để trống | đọc `deliveries` | `V-required` |
| `actualQty` / 実数量 | number (`step=0.001`) | yes | `≥ 0`; `short` ⇒ `< expectedQty` · `over` ⇒ `> expectedQty` · `returned` ⇒ `0 < actualQty ≤ expectedQty` (số trả về) · `partial_cancel` ⇒ `0 < actualQty ≤ remaining` (số huỷ) | không | `V-qty-direction` |
| `reason` / 理由 | textarea | yes | `10 ≤ độ dài ≤ 1000` ký tự; **không có danh sách lý do dựng sẵn ép chọn** — xem Unresolved Questions | không | `V-reason` |
| `confirmedByName` / 現場確認者 | text | yes | `2 ≤ độ dài ≤ 100`; **khác** tài khoản đăng nhập (máy dùng chung theo ca) | không | `V-required` |
| `occurredAt` / 発生日時 | datetime-local | yes | `≤ now()`; phần ngày phải `= trades.business_date`; mặc định `now()` làm tròn phút | không | `V-date` |
| `evidenceRef` / 証跡 | text | no | số hiệu biên bản hoặc URL, `≤ 300` ký tự | không | — |

Khối duyệt (checker, `reconcile`):

| Field | Type | Required | Constraints | Async Check | Error Message |
|-------|------|----------|-------------|-------------|---------------|
| `decision` / 判定 | radio, 2 lựa chọn | yes | ∈ `approve` 承認 · `reject` 否認; mặc định rỗng | không | `V-required` |
| `decisionNote` / 判定理由 | textarea | yes khi `reject` | `10 ≤ độ dài ≤ 1000` khi `reject`; tuỳ chọn khi `approve` | không | `V-reason` |

### B) Server-side

#### Lập phiếu ngoại lệ
- **Endpoint:** `rpc('submit_delivery_exception')` *(mới, đề xuất)*
- **Request:** `p_trade_id, p_exception_type, p_related_seq, p_actual_qty, p_reason, p_confirmed_by_name, p_occurred_at, p_evidence_ref`
- **Success:** `jsonb {request_code, state, delta}` → banner xanh (`auto_applied` hoặc `pending`)
- **Errors:** `P0001` · `P0002` · `P0005` (chi tiết ở Error Catalogue)
- **Trigger:** nút "Gửi phiếu"
- **Source:** chưa có code — màn chỉ đặc tả ở LAB-4

#### Duyệt / từ chối
- **Endpoint:** `rpc('decide_delivery_exception')` *(mới, đề xuất)*
- **Request:** `p_request_id, p_decision, p_note`
- **Success:** `jsonb {request_code, state, delivered_after}` → banner xanh
- **Errors:** `P0002` · `P0003` · `P0005` · `P0065`
- **Trigger:** nút gửi quyết định
- **Source:** chưa có code — màn chỉ đặc tả ở LAB-4

Cả hai **bắt buộc qua RPC**: `deliveries` không có policy ghi qua RLS, và việc "đã duyệt ⇒ sinh dòng `deliveries`" phải nằm trong cùng một transaction với việc đổi trạng thái phiếu.

#### Error Catalogue *(mục bổ sung)*

| Mã | Điều kiện kích hoạt | Thông báo VI | Thông báo JA | Hiện ở đâu | Chặn gửi? |
|---|---|---|---|---|:---:|
| `V-required` | thiếu một trong `tradeCode` · `exceptionType` · `actualQty` · `reason` · `confirmedByName` | Vui lòng nhập đủ loại ngoại lệ, số lượng thực tế, lý do và người xác nhận. | 例外種別・実数量・理由・確認者をすべて入力してください。 | banner đầu form, `aria-live="assertive"` | ● |
| `V-qty-direction` | số lượng không khớp hướng của loại ngoại lệ (`actualQty`) | Loại "{exception_type}" đòi số lượng thực tế {điều kiện}; giá trị đã nhập là {actual}. | 「{exception_type}」では実数量が{điều kiện}である必要があります（入力値：{actual}）。 | dưới ô số lượng | ● |
| `V-reason` | `reason` < 10 ký tự | Vui lòng nêu lý do, tối thiểu 10 ký tự. | 理由を10文字以上で入力してください。 | dưới ô | ● |
| `V-date` | phần ngày của `occurredAt` ≠ `trades.business_date` | Thời điểm phát sinh phải nằm trong ngày nghiệp vụ {business_date}. | 発生日時は業務日{business_date}内である必要があります。 | dưới ô | ● |
| `P0001` | `tradeCode` không tra được, hoặc `relatedSeq` không có | Không tìm thấy giao dịch hoặc lần giao đã chọn. | 該当する取引または回次が見つかりません。 | banner | ● |
| `P0002` | ngày nghiệp vụ đã chốt sổ (kiểm trong RPC) | Ngày nghiệp vụ {business_date} đã chốt sổ lúc {closed_at}. Ngoại lệ sau chốt sổ phải lập phiếu điều chỉnh (SCR022). | 業務日{business_date}は{closed_at}に締められています。締め後の例外は訂正申請（SCR022）から行ってください。 | banner, kèm liên kết SCR022 | ● |
| `P0003` | tư cách người nhận hết hiệu lực (kiểm khi **duyệt**, không kiểm lúc lập phiếu) | Người nhận {buyer_name} đã hết hạn tham gia từ {expired_on}, căn cứ {basis_label}. Liên hệ {contact_office}. | 買受人{buyer_name}は{expired_on}から資格が失効しています（根拠：{basis_label}）。{contact_office}にご連絡ください。 | banner màn duyệt | ● |
| `P0005` | lập phiếu mà role ≠ `receiving`, hoặc duyệt mà role ≠ `reconcile` (thiếu cờ `can_approve` là `P0065`, không phải mã này) | Chức danh của bạn không được phép thực hiện thao tác này. | あなたの職掌ではこの操作を実行できません。 | banner | ● |
| `P0064` *(mới)* | ghi **thẳng** một lần giao `status='exception'` có `deltaQty ≠ 0` qua `record_delivery` (SCR014) mà không có phiếu đã duyệt — `details: {exception_type, delta}` | Ngoại lệ "{exception_type}" làm lệch {delta} {unit}, phải lập phiếu ngoại lệ có duyệt (SCR017). | 「{exception_type}」は{delta} {unit}の差異を生じるため、承認付きの例外申請（SCR017）が必要です。 | banner của **SCR014**, kèm liên kết sang SCR017 | ● |
| `P0065` *(mới)* | người duyệt trùng người lập **hoặc** không có cờ `can_approve` (cùng cách `P0013` của `SCR036`) — `details: {reason: self_approve \| no_can_approve, requested_by, actor_id}` | `self_approve`: Phiếu do chính bạn lập — người duyệt phải khác người đề nghị. `no_can_approve`: Bạn chưa có thẩm quyền duyệt phiếu ngoại lệ. | `self_approve`: 自身が起票した申請は承認できません。起票者と承認者は別である必要があります。 `no_can_approve`: 承認権限がありません。 | banner màn duyệt; nút duyệt cũng **ẩn** sẵn ở danh sách với phiếu của chính mình | ● |
| (network) | Server Action throw | Mất kết nối — chưa gửi được. Dữ liệu đã nhập vẫn giữ nguyên. | 通信が切断され、送信できませんでした。入力内容は保持されています。 | banner | ● |

## Interaction Patterns

- **4 lựa chọn loại ngoại lệ là nhóm `radio` HTML gốc (Tab vào, mũi tên chọn), không dùng combobox tự quản focus** — vì maker làm ở cầu cảng với găng tay; thiết kế LAB-4.
- **`Escape` xoá form và đưa focus về ô mã giao dịch** — thiết kế LAB-4.
- **Không có affordance chỉ hiện khi hover** — thiết kế LAB-4.
- **Nhập số thực tế hiện ngay `deltaQty` và cảnh báo "cần duyệt" khi `delta ≠ 0`** — thiết kế LAB-4.

## Accessibility

| Aspect | Status | Notes |
|--------|--------|-------|
| ARIA roles/labels | partial (đặc tả) | banner `role="status"`/`role="alert"`; nhóm radio có `fieldset`/`legend` (đề xuất) |
| Keyboard navigation | supported (đặc tả) | radio HTML gốc; `Escape` xoá form |
| Focus management | managed (đặc tả) | focus về ô mã giao dịch sau thành công/`Escape`; banner lỗi tự nhận focus |
| Screen reader compatibility | unknown | chưa có bản dựng để kiểm |

## Conditional Rendering

| Condition | Type | Renders | Hidden | Notes |
|-----------|------|---------|--------|-------|
| role = `receiving` | auth | form lập phiếu | khối duyệt | maker không tự duyệt phiếu của mình |
| role = `reconcile` | auth | danh sách chờ duyệt + khối duyệt (nút duyệt chỉ khi có `can_approve`) | form lập phiếu | bypass thiếu `can_approve`: RPC `P0065`; role khác `reconcile`: `P0005` |
| phiếu do chính người xem lập | auth | — | nút duyệt | RPC chặn `P0065` kể cả khi UI bị bypass |
| `exceptionType = partial_cancel` | hardcoded-id | — | select `relatedSeq` | áp cho phần chưa giao |
| `decision = reject` | hardcoded-id | `decisionNote` bắt buộc | — | |
| phiếu `approved`/`rejected` hoặc ngày đã chốt | hardcoded-id | chế độ chỉ đọc | nút gửi | |

## Component Variants

Không áp dụng — màn không có component đổi hình dạng theo biến thể; khác biệt theo chức danh/trạng thái đã mô tả ở Conditional Rendering.

## Security Surface

| Guard | Type | Consequence if bypassed |
|-------|------|------------------------|
| `ROUTE_POLICY` — `/deliveries` → `["receiving","reconcile"]`, kèm mục khớp chính xác `/deliveries/new` → `["receiving"]` xét trước (xem SCR015) | role | role sai mở được màn |
| RLS bảng `delivery_exception_requests` *(mới, đề xuất)* | permission | đọc/ghi ngoài chức danh |
| Role `reconcile` (`P0005`) + cờ `can_approve` (`P0065`) **bên trong RPC duyệt** | permission | duyệt bởi người không có thẩm quyền, dù cùng role |
| Maker ≠ checker **bên trong RPC duyệt** (`P0065`) | permission | tự duyệt phiếu của mình — vi phạm `起票・承認分離`; là bất biến nghiệp vụ, không phải quy ước giao diện (bài học ADR-004) |
| RPC `submit_delivery_exception` / `decide_delivery_exception` | permission | `[UNVERIFIED]` — chưa tồn tại, đây là yêu cầu thiết kế |

Quyền theo chức danh: `sys_admin` ✗ · `receiving` xem + ghi (lập phiếu, là **maker**) · `trading` ✗ · `reconcile` xem; ghi (duyệt, là **checker**) chỉ khi có cờ `can_approve` — LAB-1: "có lệch số lượng thì cần bộ phận đối chiếu xác nhận"; 5 role đề xuất (`party_admin`, `rule_admin`, `ops_admin`, `market_manager`, `internal_audit`) ✗. `receiving` gồm cả 配送部門 (PREMISE C-10).

## Source References

Chưa có code — màn chỉ đặc tả ở LAB-4. Tài liệu nguồn đã đọc (mục có chú thích "không nộp" là tài liệu gốc trong repo):

1. (tài liệu gốc trong repo, không nộp) `sc-17-ngoai-le-giao-hang.md` — spec 10 mục gốc
2. `lib/auth/route-policy.ts:17-27` — `ROUTE_POLICY` hiện có (dòng 23: `/deliveries` chỉ `receiving`)
3. `supabase/migrations/0007_rpc_functions.sql:114-166` — `record_delivery`, chặn `P0006` (dòng 151) khiến `over` không ghi được (xem Dependencies and Risks)

## Source Walkthrough

Chưa có code — màn chỉ đặc tả ở LAB-4.

### Call Hierarchy

```text
(thiết kế) /deliveries/exceptions/new -> rpc submit_delivery_exception -> deliveries | delivery_exception_requests
(thiết kế) /deliveries/exceptions/[id] -> rpc decide_delivery_exception -> deliveries (nếu duyệt)
```

## Dependencies & Risks *(mục bổ sung)*

- Phụ thuộc SCR014/SCR016 (có lần giao để tham chiếu) và SCR019/SCR020 (tranh chấp) — một phiếu bị từ chối thường mở ra một tranh chấp.
- **Khuyết tật của `v_trade_delivery_status` phải sửa trước khi làm màn này.** Công thức hiện tại là `delivered_qty = Σ quantity WHERE status IN ('completed','exception')` — **cộng mọi dòng ngoại lệ**. Đúng với `short` và `over` (số đã thực giao), nhưng **sai dấu** với `returned` và `partial_cancel`: hàng trả về và phần huỷ phải **trừ ra** khỏi lũy kế, không phải cộng vào. Không sửa thì mỗi lần trả hàng lại làm "đã giao" tăng lên. Sửa bằng một migration đổi công thức view (cộng `short`/`over`, trừ `returned`/`partial_cancel`) — **đây là thay đổi phá tương thích**, chạm SCR014, SCR015, SCR016 và SCR018.
- **`over` (giao thừa) không ghi được bằng đường hiện có.** `record_delivery` chặn `requested > remaining` bằng `P0006`, nên một lần giao thừa luôn bị từ chối. Vì vậy `over` **chỉ** vào hệ thống qua phiếu đã duyệt ở màn này, và RPC `decide_delivery_exception` phải bỏ qua trần `P0006` một cách tường minh — việc duyệt của bộ phận đối chiếu chính là lớp kiểm soát thay thế. Nếu LAB-5 bỏ sót chỗ này thì `over` thành tính năng chết.
- **Chồng lấn với SCR014 phải cắt dứt khoát.** SCR014 hiện cho ghi thẳng `status='exception'` + `exception_type` không qua duyệt nào. Quy tắc đề xuất: SCR014 chỉ giữ đường ghi thẳng cho ngoại lệ **không lệch số lượng**; mọi ngoại lệ có `delta ≠ 0` bị `record_delivery` từ chối bằng `P0064` và đẩy sang màn này. Không có quy tắc này thì bước duyệt 🔒 vô hiệu — ai cũng đi vòng qua SCR014.
- **Rủi ro hiểu sai:** phiếu ở `pending` **không** được tính vào `delivered_qty`. Nếu LAB-5 cho phiếu chờ duyệt ảnh hưởng số liệu ngay thì vi phạm đúng nguyên tắc của F07 ("số liệu gốc chưa đổi cho tới khi được duyệt").
- `partial_cancel` (huỷ một phần) **chồng chéo** với huỷ giao dịch ở SCR013 (`P0062`). Hai màn phải thống nhất: huỷ phần **chưa giao** đi qua đây; huỷ **toàn bộ giao dịch chưa giao gì** đi qua SCR013.

### Unresolved Questions *(mục bổ sung)*

- **Danh sách lý do dựng sẵn.** `FR-DEL-03` nói "cần danh sách lý do + người xác nhận ngoại lệ", nhưng danh sách đó chưa có trong bất kỳ tài liệu nào. Tài liệu này tạm dùng ô văn bản tự do; nếu khách cung cấp danh mục thì ô `reason` thành select + ô bổ sung, và cần một bảng danh mục có phiên bản.
- Ngưỡng nào thì **bắt buộc** duyệt? Tài liệu này chọn "mọi lệch khác 0". Nếu khách muốn ngưỡng dung sai (ví dụ lệch ≤ 1% thì tự áp dụng) thì ngưỡng đó phải là **dữ liệu có phiên bản**, không phải hằng số trong code (tài liệu gốc trong repo, không nộp: `architecture.md` §3.2).
- Một cấp duyệt hay nhiều cấp theo giá trị lệch? Phụ thuộc câu 3 gửi khách (tài liệu LAB-1 (tài liệu gốc trong repo, không nộp) `product-a-rfp-qa-part-a-client.md`) — cùng câu đang chặn F07.
- `reconcile` có được **vừa duyệt vừa lập** phiếu không khi chính họ phát hiện sai lệch lúc đối chiếu? Hiện thiết kế cấm `reconcile` lập phiếu; nếu mở ra thì `P0065` vẫn chặn họ tự duyệt phiếu của mình, nhưng cần ít nhất hai tài khoản `reconcile` có `can_approve` để hệ thống chạy được.
- Thiếu `can_approve` dùng chung mã `P0065` với maker = checker (cùng cách `P0013` ở `SCR036`), phân biệt bằng `details.reason`. Có cần tách thành hai mã (như `P0027` của `SCR024`) để người dùng và log phân biệt rõ hơn không?
