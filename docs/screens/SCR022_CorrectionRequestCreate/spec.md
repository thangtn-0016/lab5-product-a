# SCR022_CorrectionRequestCreate — Screen Spec

> Alias LAB-1: **SC-22** · 修正申請票の作成 · Tạo phiếu xin điều chỉnh

**Screen**: SCR022: CorrectionRequestCreate (Tạo phiếu xin điều chỉnh)
**Feature**: [F007_PostCloseCorrection](../../features/F007_PostCloseCorrection/screens.md) — tính năng LAB-1 `F07-01`
**Type**: atomic (form gửi phiếu có hộp thoại xác nhận)
**Route**: `/corrections/new` *(đề xuất)*
**Generated**: 2026-10-06
**Trạng thái**: Thiết kế mới, chưa xây (LAB-4)
**Role**: `reconcile` (maker — lập phiếu; kể cả `reconcile` có cờ `can_approve`, đề xuất)
**User story**: `US-16` (và là đích đến của `US-12` — người bị chặn `P0009` được dẫn sang đây)

> **Ký hiệu nguồn dùng trong cả ba file F07.** `[KH]` = ràng buộc **bắt buộc theo tài liệu khách / LAB-1**, có dẫn nguồn. `[ĐX]` = **đề xuất của tài liệu này**, chưa được khách xác nhận. Mọi tên bảng, tên cột, tên RPC và mã lỗi `P0020`+ trong file này đều là `[ĐX]` — chúng chưa tồn tại trong `db-schema.md`.

## Purpose

Nhân viên bộ phận đối chiếu (`reconcile`, 照合事務局) phát hiện sai sau 10:00 JST thì không sửa được nữa — `guard_closed_day` chặn ở tầng trigger (`P0009`) và ghi log. Màn này mở đường sửa **hợp lệ duy nhất** cho số liệu thuộc một ngày nghiệp vụ (業務日) đã chốt: đây là nơi họ **xin** sửa.

Điểm phải khắc vào đầu người triển khai: màn này **không sửa gì cả**. Nó chỉ tạo một bản ghi yêu cầu. Số liệu gốc đứng yên tuyệt đối cho tới khi có người thứ hai duyệt ở SCR024 `[KH — US-16, FR-CORR-01]`.

Bối cảnh dùng: giờ hành chính, bàn giấy, máy cá nhân. Không phải màn hiện trường — không có ràng buộc găng tay/2 giờ sáng.

## Screen Layout

Trang form một cột trong khung ứng dụng chung: banner cảnh báo thường trực ở đầu form, rồi ba khối A (chọn bản ghi đích) · B (số trước, ảnh chụp chỉ đọc) · C (số sau, đề nghị) và nút "Gửi phiếu" mở hộp thoại xác nhận chặn luồng. Chưa có code nên chưa có breakpoint chốt; bàn giấy/máy cá nhân nên không có ràng buộc cỡ chạm.

### Layout Sketch

```
┌──────────────────────────────────────────────────────┐
│ R1: Header ứng dụng + nav (static)                │
├──────────────────────────────────────────────────────┤
│ R2: Banner vàng cố định "Phiếu này KHÔNG sửa số…"    │
├──────────────────────────────────────────────────────┤
│ R3: Khối A — loại bản ghi · mã · ngày NV · đã chốt   │
│ R4: Khối B — số trước (chỉ đọc)   - - (sau khi tra) -│
│ R5: Khối C — kiểu · số sau · Δ · lý do · chứng cứ    │
│     [Gửi phiếu]                                      │
└──────────────────────────────────────────────────────┘
   ┌─ R6: Hộp thoại xác nhận (nhắc append-only + ngày ghi nhận) ─┐
```

### Layout Regions

| Region ID | Name | Position | Scrollable | Key Components | Responsive Behavior |
|-----------|------|----------|------------|----------------|---------------------|
| R1 | Header + nav | static | no | khung `(app)` chung | always |
| R2 | Banner cảnh báo thường trực | sticky | no | banner vàng, không đóng/ẩn được | always |
| R3 | Khối A chọn bản ghi đích | static | no | select, text, readonly | fluid |
| R4 | Khối B số trước | static | no | readonly; ẩn khi chưa chọn bản ghi | conditional |
| R5 | Khối C số sau + nút gửi | static | yes | number, textarea, nút | conditional (hiện sau khi tra bản ghi) |
| R6 | Hộp thoại xác nhận | floating (modal) | no | cảnh báo + `effective_business_date` | conditional |

## Child Routes

Không áp dụng — màn này là một route đơn, không phải khung chứa route con.

## User Flow

> Phạm vi: tương tác trong màn. Điều hướng liên màn xem [screen-flow](../../generated/screen-flow.md).

### Happy Path

1. Người đối chiếu vào từ SCR012 (danh sách giao dịch), SCR018, hoặc từ thông báo `P0009`/`P0002` của SCR008/SCR013 — các màn đó đưa `?target=` sẵn `[KH — US-12: "thông báo hướng người dùng sang đường xin điều chỉnh"]`.
2. Form mở, banner cảnh báo hiện ngay, trước cả khi gõ gì.
3. Chọn loại bản ghi, nhập mã bản ghi → hệ thống tra bản ghi, điền Khối B và `target_business_date`, `closed_at`, `effective_business_date`.
4. Chọn kiểu điều chỉnh: `reverse` → Khối C ẩn `after.quantity`, `after.unit_price`; `delta.*` hiện giá trị âm của số gốc. `delta` → nhập số sau; `delta.*` cập nhật theo từng phím gõ.
5. Nhập lý do, chứng cứ.
6. Bấm "Gửi phiếu" → hộp thoại xác nhận → gọi RPC.
7. **Nhánh thành công:** phiếu vào `pending`, hiện ở SCR023 của người duyệt. **Bản ghi gốc không đổi một ký tự nào.**
8. **Nhánh thất bại:** banner đỏ, hộp thoại đóng, dữ liệu còn nguyên.
9. Rút phiếu khi chưa ai duyệt → `/corrections/{id}` có nút "Rút phiếu" → `cancelled`, vẫn lưu dấu vết.

### Branches

| Decision point | Condition | Outcome on this screen | Source |
|----------------|-----------|------------------------|--------|
| Bước 3 | ngày chưa chốt | `P0020`, dừng tại đây | thiết kế LAB-4 |
| Bước 3 | đã có phiếu chờ duyệt cho bản ghi | `P0021`, dừng tại đây | thiết kế LAB-4 |
| Bước 4 | `correction_type = reverse` | ẩn `after.quantity`/`after.unit_price`, hiện Δ âm | thiết kế LAB-4 |
| Bước 6 | `P0001`/`P0005`/`P0022`/`P0002` | banner đỏ tự nhận focus, dữ liệu giữ nguyên | thiết kế LAB-4 |
| Bước 9 | phiếu chưa ai duyệt | nút "Rút phiếu" → `cancelled` | thiết kế LAB-4 |

## Data Inventory

> Chỉ field chỉ đọc. Ô nhập nằm ở `## Validation & Error Feedback`.

### Khối A — bản ghi đích (chỉ đọc)

| Field | Display Label | Source | Format | Empty Behavior | Cross-ref |
|-------|---------------|--------|--------|----------------|-----------|
| `target_business_date` | Ngày nghiệp vụ của bản ghi / 対象業務日 | `trades.business_date` (tra tự động từ mã bản ghi) | date; **phải có `business_days.status = 'closed'`** — chưa chốt thì không đi đường này, xem `P0020` | ẩn khi chưa chọn bản ghi | trades |
| `closed_at` | Đã chốt lúc / 締め時刻 | `business_days.closed_at` | datetime | "—" khi ngày chưa chốt | business_days |

### Khối B — số trước (ảnh chụp, chỉ đọc)

Đọc tại thời điểm **mở form** và gửi kèm khi lưu vào `correction_requests.snapshot_before jsonb`. Ảnh chụp này là căn cứ để SCR024 phát hiện số liệu đã bị đổi giữa chừng (`P0024`).

| Field | Display Label | Source | Format | Empty Behavior | Cross-ref |
|-------|---------------|--------|--------|----------------|-----------|
| `before.quantity` | Số lượng hiện tại / 現在の数量 | `trades.quantity` / `deliveries.quantity` | number | — | trades, deliveries |
| `before.unit_price` | Đơn giá hiện tại / 現在の単価 | `trades.unit_price` | number; chỉ hiện khi loại bản ghi = `trades` | ẩn | trades |
| `before.amount` | Thành tiền hiện tại / 現在の金額 | `trades.amount` | công thức `quantity × unit_price` (`trades.amount` là cột GENERATED) | — | trades |

### Khối C — số sau, vùng tính ra (chỉ đọc)

| Field | Display Label | Source | Format | Empty Behavior | Cross-ref |
|-------|---------------|--------|--------|----------------|-----------|
| `delta.quantity` | Chênh lệch số lượng / 数量差分 | tính ở client, **server tính lại** | `after.quantity − before.quantity`; `reverse` → `−before.quantity` | — | computed |
| `delta.amount` | Chênh lệch thành tiền / 金額差分 | tính ở client, **server tính lại** | `after.quantity × after.unit_price − before.amount`; `reverse` → `−before.amount` | — | computed |
| `effective_business_date` | Ngày nghiệp vụ ghi nhận điều chỉnh / 計上業務日 *(đề xuất)* | `business_days` (tra tự động) | `= min(business_days.business_date)` có `status='open'` và `≥ current_date`; **không bao giờ bằng `target_business_date`** | — | business_days |

**`effective_business_date` là mấu chốt của append-only.** Bản ghi điều chỉnh rơi vào **kỳ/ngày đang mở**, không bao giờ rơi vào ngày đã chốt `[KH — FR-INC-03, architecture.md §3.1]`. Đặt được nó bằng `target_business_date` là đã phá `BR-CLOSE-01`: in lại báo cáo kỳ cũ sẽ ra số khác số đã công bố.

### Read Calls *(mục bổ sung)*

| Thao tác | Cách gọi | Tham số | Trả về |
|---|---|---|---|
| Tra bản ghi đích + số trước | `select` từ `v_correction_target` *(view mới, `[ĐX]`)* | `target_table`, `target_code` | `{record_id, business_date, day_status, closed_at, quantity, unit_price, amount}` |
| Tra ngày ghi nhận | `select` từ `business_days` `where status='open' and business_date >= current_date order by business_date limit 1` | | một `business_date` |

## UI States

| State | Trigger | Visual Behavior | User Action Available | Source |
|-------|---------|----------------|-----------------------|--------|
| empty | chưa chọn bản ghi đích (mã trống) | chỉ hiện Khối A; Khối B và C ẩn | nhập mã | thiết kế LAB-4 |
| loading | sau khi nhập mã, đang tra bản ghi đích | khung xương ở Khối B; nút Gửi disabled | none | thiết kế LAB-4 |
| **warning (thường trực)** | **luôn luôn, từ lúc mở form** | banner vàng cố định đầu form `[KH — US-16, screen-list SC-22]`: *"Phiếu này KHÔNG sửa số liệu đã chốt. Số liệu gốc giữ nguyên cho tới khi có người khác duyệt; khi được duyệt, hệ thống tạo một bản ghi điều chỉnh mới ở ngày nghiệp vụ {effective_business_date}."* — không đóng được, không ẩn được | none | thiết kế LAB-4 |
| success | RPC trả `{request_code}` | banner xanh `role="status"`: *"Đã gửi phiếu {request_code}, đang chờ duyệt. Số liệu gốc chưa thay đổi."* + liên kết tới `/corrections/{id}`; form chuyển chỉ đọc, focus về banner | mở phiếu | thiết kế LAB-4 |
| error (nghiệp vụ) | RPC từ chối | banner đỏ `role="alert"` **tự nhận focus**; **toàn bộ dữ liệu đã gõ giữ nguyên** | sửa | thiết kế LAB-4 |
| error (mạng) | Server Action throw | banner đỏ + nút "Thử lại"; **dữ liệu đã gõ giữ nguyên** — mất một phiếu đã soạn 10 phút là lỗi không chấp nhận được | thử lại | thiết kế LAB-4 |
| read-only | mở lại phiếu đã gửi | toàn bộ field chỉ đọc + khối trạng thái duyệt | Rút phiếu (khi chưa duyệt) | thiết kế LAB-4 |
| saving | sau khi xác nhận gửi | nút `disabled` | none | thiết kế LAB-4 |

## Validation & Error Feedback

### A) Client-side

| Field | Type | Required | Constraints | Async Check | Error Message |
|-------|------|----------|-------------|-------------|---------------|
| `target_table` / 対象レコード種別 *(đề xuất)* | select | yes | ∈ `trades` `deliveries` `incentive_results`; mặc định `trades` | không | xem Error Catalogue |
| `target_code` / 対象レコード番号 *(đề xuất)* | text | yes | #`trades` → phải tồn tại trong `trades.trade_code`; #`deliveries` → `trades.trade_code` + `deliveries.seq`; #`incentive_results` → phải tồn tại trong `v_incentive_results.result_code`; mặc định từ query `?target=` nếu đến từ SCR012/SCR018 | `v_correction_target` | `P0001` |
| `correction_type` / 訂正区分 *(đề xuất)* | select | yes | ∈ `reverse` (đảo toàn bộ bản ghi gốc) `delta` (ghi phần chênh lệch); mặc định `delta` | không | xem Error Catalogue |
| `after.quantity` / 訂正後の数量 | number | yes khi `delta` | `> 0`; ≤ 3 chữ số thập phân (khớp `numeric(14,3)`); với `trades`: `after.quantity ≤ v_lot_availability.remaining + before.quantity`; mặc định bằng `before.quantity` | không | xem Error Catalogue |
| `after.unit_price` / 訂正後の単価 | number | yes khi `delta` và `trades` | `> 0`; ≤ 2 chữ số thập phân; mặc định bằng `before.unit_price` | không | xem Error Catalogue |
| `reason` / 理由 | text (nhiều dòng) | yes `[KH]` | 10–1000 ký tự | không | xem Error Catalogue |
| `evidence_ref` / 証跡 | text | yes `[KH]` | ≤ 300 ký tự; URL hoặc số hiệu văn bản | không | xem Error Catalogue |

**`reason` và `evidence_ref` bắt buộc là yêu cầu khách, không phải lựa chọn thiết kế** — `US-16`: *"Bắt buộc nêu lý do và đính kèm chứng cứ"*; `FR-CORR-01`: *"kèm lý do + chứng cứ đính kèm + trạng thái phê duyệt"*.

**Đã cân nhắc và bỏ:** một ô checkbox *"tôi hiểu thao tác này không sửa số cũ"*. Bỏ vì `US-16` chỉ đòi *màn hình hiện rõ cảnh báo*, không đòi xác nhận — thêm một ô bắt buộc nữa chỉ làm người dùng tích cho qua (KISS).

### B) Server-side

#### Lập phiếu
- **Endpoint:** `rpc('create_correction_request')` *(đề xuất)*
- **Request:** `p_target_table, p_target_record_id, p_correction_type, p_after jsonb, p_snapshot_before jsonb, p_reason, p_evidence_ref`
- **Success:** `jsonb {request_id, request_code, effective_business_date}` → banner xanh, form chỉ đọc
- **Errors:** `P0001` · `P0005` · `P0020` · `P0021` · `P0022` · `P0002` (chi tiết ở Error Catalogue)
- **Trigger:** "Gửi phiếu" → hộp thoại xác nhận
- **Source:** chưa có code — màn chỉ đặc tả ở LAB-4

#### Rút phiếu
- **Endpoint:** `rpc('cancel_correction_request')` *(đề xuất)*
- **Request:** `p_request_id`
- **Success:** `jsonb {decision_state}`
- **Errors:** theo Error Catalogue
- **Trigger:** nút "Rút phiếu" ở `/corrections/{id}`
- **Source:** chưa có code — màn chỉ đặc tả ở LAB-4

Ghi **bắt buộc qua RPC `SECURITY DEFINER`** — `correction_requests` không có policy INSERT, đúng khuôn ADR-002.

**Server tính lại `delta.quantity`, `delta.amount`, `effective_business_date` — không tin client.** Client tính chỉ để hiển thị tức thời. Nếu server tin con số client gửi lên thì `delta` trở thành dữ liệu người dùng nhập, và cả nguyên tắc reverse/delta mất giá trị kiểm toán. Cùng tinh thần ADR-003: số dẫn xuất không bao giờ là thứ được gửi vào.

#### Error Catalogue *(mục bổ sung)*

| Mã | Điều kiện kích hoạt | Thông báo VI | Thông báo JA | Hiện ở đâu | Chặn gửi? |
|---|---|---|---|---|:---:|
| `V-01` | ô bắt buộc để trống | Mục này bắt buộc. | 必須項目です。 | dưới ô | ● |
| `V-02` | `reason` < 10 ký tự | Lý do phải từ 10 ký tự — phiếu này là chứng cứ kiểm toán. | 理由は10文字以上で入力してください。 | dưới ô | ● |
| `V-03` | `evidence_ref` để trống | Phải đính kèm chứng cứ — không có chứng cứ thì phiếu không duyệt được. | 証跡の添付は必須です。 | dưới ô | ● |
| `V-04` | `after.quantity ≤ 0` hoặc `after.unit_price ≤ 0` | Số lượng và đơn giá phải lớn hơn 0. | 数量・単価は0より大きい値を入力してください。 | dưới ô | ● |
| `V-05` | `correction_type`=`delta` và cả `after.quantity` lẫn `after.unit_price` đều bằng số trước | Chưa có gì thay đổi — không cần lập phiếu. | 変更点がありません。 | dưới nhóm Khối C | ● |
| `P0001` | `target_code` không tra ra bản ghi nào | Không tìm thấy bản ghi「{target_code}」. | 「{target_code}」が見つかりません。 | banner đầu form | ● |
| `P0005` | role ≠ `reconcile` | Chức danh của bạn không được phép lập phiếu điều chỉnh. | あなたの職掌では修正申請票を作成できません。 | banner đầu form | ● |
| `P0020` *(mới, `[ĐX]`)* | ngày nghiệp vụ của bản ghi đích **chưa chốt** | Ngày {business_date} chưa chốt sổ — hãy sửa trực tiếp, không cần phiếu điều chỉnh. | 業務日{business_date}は未締めです。修正申請票は不要です。 | banner đầu form | ● |
| `P0021` *(mới, `[ĐX]`)* | đã tồn tại phiếu `decision_state='pending'` trên cùng `(target_table, target_record_id)` | Bản ghi này đang có phiếu {request_code} chờ duyệt. | このレコードには承認待ちの申請票{request_code}があります。 | banner đầu form | ● |
| `P0022` *(mới, `[ĐX]`)* | server tính lại thấy `delta.quantity = 0` và `delta.amount = 0` | Phiếu không tạo ra thay đổi nào. | 差分が発生しない申請は登録できません。 | banner đầu form | ● |
| `P0002` | `effective_business_date` tra ra lại vừa bị chốt giữa chừng | Ngày ghi nhận {business_date} vừa được chốt sổ. Hãy tải lại trang. | 計上業務日{business_date}が締められました。再読み込みしてください。 | banner đầu form | ● |

Rẽ nhánh theo `error.code`; thông báo dựng từ `error.details` đã `JSON.parse` (`{business_date}`, `{request_code}`, `{target_code}`) — **không so khớp chuỗi `error.message`** (db-schema.md §4).

`P0021` là `[ĐX]` và có thể bị khách bác: hai phiếu chồng nhau trên cùng một giao dịch về lý thuyết cộng dồn được, nhưng lúc đó `SCR024` phải so số trước với kết quả của phiếu kia chứ không phải với số gốc — độ phức tạp nhân lên mà không có yêu cầu nào đòi. Chặn một phiếu mỗi lúc là lựa chọn KISS, ghi ở Unresolved Questions.

## Interaction Patterns

- **Nút "Gửi phiếu" mở hộp thoại xác nhận chặn luồng, nhắc lại cảnh báo append-only và `effective_business_date`** — đây là điểm `CAP-HUMAN-01` *"phê duyệt sửa dữ liệu nhạy cảm"*: một Enter lỡ tay không được phép khởi động một luồng duyệt; thiết kế LAB-4.
- **`delta.*` cập nhật theo từng phím gõ, nhưng server tính lại** — thiết kế LAB-4.
- Không có yêu cầu bàn phím riêng — không thuộc ba màn keyboard-first (SCR007, SCR010, SCR014).

## Accessibility

| Aspect | Status | Notes |
|--------|--------|-------|
| ARIA roles/labels | partial (đặc tả) | banner thành công `role="status"`, lỗi `role="alert"`; banner cảnh báo thường trực cần `role="note"` (đề xuất) |
| Keyboard navigation | supported (đặc tả) | form chuẩn; hộp thoại xác nhận |
| Focus management | managed (đặc tả) | banner lỗi tự nhận focus; sau thành công focus về banner |
| Screen reader compatibility | unknown | chưa có bản dựng để kiểm |

## Conditional Rendering

| Condition | Type | Renders | Hidden | Notes |
|-----------|------|---------|--------|-------|
| role = `reconcile` | auth | form | — | role khác bị `/403`; bypass: RPC `P0005` |
| chưa chọn bản ghi đích | hardcoded-id | Khối A | Khối B, C | |
| `correction_type = reverse` | hardcoded-id | `delta.*` âm | `after.quantity`, `after.unit_price` | |
| loại bản ghi ≠ `trades` | hardcoded-id | — | `before.unit_price`, `after.unit_price` | |
| luôn luôn | hardcoded-id | banner cảnh báo thường trực | — | không đóng/ẩn được |

## Component Variants

Không áp dụng — màn không có component đổi hình dạng theo biến thể; khác biệt theo chức danh/trạng thái đã mô tả ở Conditional Rendering.

## Security Surface

| Guard | Type | Consequence if bypassed |
|-------|------|------------------------|
| `lib/auth/route-policy.ts` thêm `{ prefix: "/corrections", roles: ["reconcile"] }` **trước** entry `{ prefix: "/", roles: "all" }` (bảng duyệt theo thứ tự khai báo) → `proxy.ts` đẩy về `/403` *(đề xuất)* | role | role sai mở được màn |
| RLS `correction_requests` (SELECT cho `reconcile`, **không có policy ghi** — đúng khuôn `trades`/`deliveries` của ADR-002) | permission | ghi ngoài RPC |
| RPC `create_correction_request` | permission | `[UNVERIFIED]` — chưa tồn tại, là yêu cầu thiết kế |

Quyền theo chức danh: `sys_admin` ✗ — quản trị kỹ thuật không lập phiếu nghiệp vụ · `reconcile` xem + ghi — là **maker** (起票者) `[KH — screen-list SC-22: chức danh Đối chiếu]` · `reconcile` có cờ `can_approve` *(đề xuất)* xem + ghi — lập được, nhưng phiếu mình lập thì mình không duyệt (xem SCR023) · `trading` · `receiving` ✗.

## Source References

Chưa có code — màn chỉ đặc tả ở LAB-4. Tài liệu nguồn đã đọc:

1. (tài liệu gốc trong repo, không nộp) `sc-22-tao-phieu-dieu-chinh.md` — spec 10 mục gốc
2. `lib/auth/route-policy.ts:17-27` — `ROUTE_POLICY` hiện có (chưa có `/corrections`)
3. `supabase/migrations/0010_closed_day_guard.sql:6` — `guard_closed_day` (nguồn của `P0009`)

## Source Walkthrough

Chưa có code — màn chỉ đặc tả ở LAB-4.

### Call Hierarchy

```text
(thiết kế) /corrections/new page -> v_correction_target + business_days (đọc)
   -> Server Action -> rpc create_correction_request | cancel_correction_request
```

## Dependencies & Risks *(mục bổ sung)*

- **Cần trước:** `SCR018` (phải có ngày đã chốt thì phiếu mới có nghĩa `[KH — function-list: "F07 chỉ có nghĩa sau khi F06 chốt được sổ"]`), bảng `correction_requests` + 2 RPC, cờ `app_users.can_approve` (dùng ở `SCR023`/`SCR024`).
- **Cần đồng bộ:** `SCR008`, `SCR012`, `SCR013`, `SCR018` phải có liên kết dẫn sang màn này. Thiếu liên kết thì `US-12` không đạt — người bị chặn không biết đi đâu.
- **Dữ liệu phải có sẵn:** ít nhất một `business_days` đã `closed` có giao dịch `confirmed`, và một `business_days` đang `open` để làm `effective_business_date`. Không có ngày mở thì không phiếu nào lập được.
- **Dễ hiểu sai #1 — lớn nhất:** triển khai `after.quantity`/`after.unit_price` thành `UPDATE trades SET quantity=...`. Đó chính xác là thứ cả nhóm F07 tồn tại để chặn. Màn này **chỉ INSERT vào `correction_requests`**, không chạm `trades`.
- **Dễ hiểu sai #2:** `effective_business_date` lấy bằng `now()::date`. Ngày hôm nay có thể là ngày nghỉ chợ (`SCR021`) hoặc chưa mở — phải tra `business_days`, không suy từ đồng hồ (`architecture.md` §3.1: *"một `now()` lọt vào tầng báo cáo là đủ phá BR-CLOSE-01 mà không có test nào đỏ"*).
- **Dễ hiểu sai #3:** cho người dùng sửa `target_business_date` hoặc `effective_business_date`. Cả hai là kết quả tra cứu, không phải ô nhập.
- **Rủi ro vận hành:** `snapshot_before` chụp lúc mở form có thể cũ hàng giờ nếu người dùng để tab mở. `SCR024` phải so lại trước khi duyệt (`P0024`), không được tin ảnh chụp.
- **Rủi ro nghiệm thu:** `evidence_ref` hiện là ô text. Nếu khách hiểu "đính kèm chứng cứ" là **upload file thật** thì phát sinh Supabase Storage + RLS cho bucket + quét virus — ngoài phạm vi đang mô tả. Ghi ở Unresolved Questions.

### Unresolved Questions *(mục bổ sung)*

1. **Số cấp duyệt — câu chặn lớn nhất của cả nhóm F07.** Câu 3 gửi khách (tài liệu LAB-1 `product-a-rfp-qa-part-a-client.md`) còn mở: đề xuất của LAB-1 là **một cấp** (trưởng bộ phận đối chiếu duyệt, maker ≠ checker, không có cấp thứ ba). Nếu khách phân cấp theo số tiền thì màn này phải thêm trường ngưỡng và định tuyến người duyệt, `SCR023` tách theo cấp, và `F07-02` nở thành 3–5 luồng.
2. **"Đính kèm chứng cứ" là đường dẫn hay file thật?** Quyết định này đổi ``evidence_ref`` từ một ô text thành cả một hệ thống lưu trữ. Tài liệu khách chỉ ghi *"添付"*.
3. **Điều chỉnh được những trường nào?** Tài liệu này giả định số lượng và đơn giá. Người mua, mã lô, ngày nghiệp vụ của giao dịch gốc thì sao — sửa được bằng `delta` hay bắt buộc `reverse` rồi lập giao dịch mới? `[ĐX]` hiện: đổi khoá (người mua/lô) **chỉ được qua `reverse`**.
4. **Một bản ghi được phép có mấy phiếu chờ duyệt?** Tài liệu này chặn ở một (`P0021`). Chưa có yêu cầu nào nói có hay không.
5. **Phiếu `pending` có hạn xử lý không?** Treo vô hạn thì `SCR023` đầy rác. Đề xuất: cảnh báo sau 3 ngày làm việc, không tự huỷ.
6. **Điều chỉnh `deliveries` có kéo theo điều chỉnh `trades` không?** Giao thiếu đã ghi `exception` rồi sửa lại số — hai bản ghi này liên đới nhau, tài liệu khách không nói.
