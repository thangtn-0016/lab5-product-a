# SCR024_RequestDetailApproval — Screen Spec

> Alias LAB-1: **SC-24** · 申請票詳細・承認 · Chi tiết phiếu & duyệt

**Screen**: SCR024: RequestDetailApproval (Chi tiết phiếu & duyệt)
**Feature**: [F007_PostCloseCorrection](../../features/F007_PostCloseCorrection/screens.md) — tính năng LAB-1 `F07-02` `F07-03` `F07-04`
**Type**: composite (thông tin phiếu + so sánh trước/sau + khối quyết định + bản ghi điều chỉnh đã sinh)
**Route**: `/corrections/[id]` *(đề xuất)*
**Generated**: 2026-10-06
**Trạng thái**: Thiết kế mới, chưa xây (LAB-4)
**Role**: `reconcile` (xem) · `reconcile` có cờ `can_approve` *(đề xuất)* (duyệt, chỉ khi `requested_by <> auth.uid()`)
**User story**: `US-17`

> **Ký hiệu nguồn.** `[KH]` = bắt buộc theo tài liệu khách / LAB-1, có dẫn nguồn. `[ĐX]` = đề xuất của tài liệu này. Bảng `correction_requests`, `correction_entries`, 2 RPC và mọi mã `P0020`+ đều là `[ĐX]`.

## Purpose

Nơi một con người quyết định, và nơi nguyên tắc **chỉ thêm, không sửa** được thi hành. Người duyệt (承認者) đọc lý do, đọc chứng cứ, **so số trước với số sau cạnh nhau** `[KH — screen-list SC-24]`, rồi duyệt hoặc từ chối.

Duyệt xong, hệ thống **không chạm vào bản ghi gốc**. Nó sinh một hoặc hai bản ghi điều chỉnh mới (reverse/delta) ở ngày nghiệp vụ đang mở, và màn hình phải nói rõ ra điều đó `[KH — screen-list SC-24: "hiện rõ bản ghi gốc vẫn nguyên, bản ghi điều chỉnh mới được tạo"]`.

LAB-1 xếp đây là **màn khó nhất của cả hệ thống** cùng với SCR018, và khuyến nghị làm sau cùng trong LAB-3. Function-list gọi F07 là *"nhóm khó nhất về kỹ thuật"* nằm trên đường găng.

## Screen Layout

Trang chi tiết một cột trong khung ứng dụng chung với bốn khối xếp dọc: A (thông tin phiếu) · B (so sánh trước/sau) · C (quyết định — chỉ khi đủ điều kiện) · D (bản ghi điều chỉnh đã sinh — chỉ khi đã duyệt). "Gửi quyết định" mở hộp thoại xác nhận chặn luồng. Chưa có code nên chưa có breakpoint chốt.

### Layout Sketch

```
┌──────────────────────────────────────────────────────┐
│ R1: Header ứng dụng + nav (static)                │
├──────────────────────────────────────────────────────┤
│ R2: Banner kết quả/lỗi (role=status | alert)         │
├──────────────────────────────────────────────────────┤
│ R3: Khối A — thông tin phiếu (chỉ đọc)               │
├──────────────────────────────────────────────────────┤
│ R4: Khối B — so sánh trước | sau | chênh lệch        │
│     - - cảnh báo lệch ảnh chụp (nếu has_drift) - -   │
├──────────────────────────────────────────────────────┤
│ R5: Khối C — quyết định [Gửi quyết định]             │
│     - - chỉ khi pending + can_approve + khác maker - │
├──────────────────────────────────────────────────────┤
│ R6: Khối D — bản ghi điều chỉnh (chỉ khi approved)   │
└──────────────────────────────────────────────────────┘
   ┌─ R7: Hộp thoại xác nhận (δ tiền + ngày ghi nhận) ─┐
```

### Layout Regions

| Region ID | Name | Position | Scrollable | Key Components | Responsive Behavior |
|-----------|------|----------|------------|----------------|---------------------|
| R1 | Header + nav | static | no | khung `(app)` chung | always |
| R2 | Banner kết quả/lỗi | static | no | `role="status"` / `role="alert"` | always |
| R3 | Khối A thông tin phiếu | static | no | danh sách mô tả | fluid |
| R4 | Khối B so sánh trước/sau | static | yes | bảng ba cột, cảnh báo lệch | fluid |
| R5 | Khối C quyết định | static | no | select, textarea, nút | conditional |
| R6 | Khối D bản ghi điều chỉnh | static | yes | bảng `correction_entries`, dòng chữ bảo toàn | conditional |
| R7 | Hộp thoại xác nhận | floating (modal) | no | nhắc `delta_amount`, `effective_business_date` | conditional |

## Child Routes

Không áp dụng — màn này là một route đơn, không phải khung chứa route con.

## User Flow

> Phạm vi: tương tác trong màn. Điều hướng liên màn xem [screen-flow](../../generated/screen-flow.md).

### Happy Path

1. Người duyệt vào từ SCR023.
2. Màn tải: đọc phiếu, **đọc lại bản ghi gốc**, so với `snapshot_before`. Nếu lệch → cảnh báo lệch hiện ngay, trước khi người duyệt đọc gì khác.
3. Đọc Khối A (lý do, chứng cứ), mở `evidence_ref` kiểm chứng cứ ở tab khác.
4. Đọc Khối B — ba cột trước/sau/chênh lệch `[KH]`.
5. Chọn `decision`; `rejected` → `decision_note` thành bắt buộc.
6. Bấm "Gửi quyết định" → hộp thoại xác nhận → gọi RPC.
7. **Nhánh duyệt:** trong **một transaction** — `select … for update` dòng phiếu; kiểm `pending`, kiểm `decided_by <> requested_by`, kiểm `can_approve`, đọc lại bản ghi gốc và so với `snapshot_before`, tra `effective_business_date`; rồi `INSERT` vào `correction_entries` (1 dòng với `delta`, 2 dòng với `reverse` + lập lại) và `UPDATE` phiếu sang `approved`. **Không có câu `UPDATE` nào chạm `trades`/`deliveries`.**
8. **Nhánh từ chối:** chỉ `UPDATE` phiếu sang `rejected` kèm `decision_note`. Không sinh bản ghi nào.
9. Màn hiện Khối D, có liên kết ngược `source_request` về phiếu và liên kết tới bản ghi gốc để tự kiểm.
10. Hệ quả xuôi dòng: SCR018 và SCR025 của ngày/kỳ `effective_business_date` cộng thêm các dòng `correction_entries`. Báo cáo của `target_business_date` **không đổi một con số nào** `[KH — FR-INC-03]`.

### Branches

| Decision point | Condition | Outcome on this screen | Source |
|----------------|-----------|------------------------|--------|
| Bước 2 | `value_before` ≠ `snapshot_before` | `drift_warning` hiện; RPC sẽ bắn `P0024` | thiết kế LAB-4 |
| Bước 5 | người xem là `requested_by` | Khối C không render, thay bằng thông báo `V-03`; kết thúc | thiết kế LAB-4 |
| Bước 5 | không có `can_approve` | Khối C không render; kết thúc | thiết kế LAB-4 |
| Bước 5 | `decision = rejected` | `decision_note` bắt buộc | thiết kế LAB-4 |
| Bước 7 | `P0001`/`P0005`/`P0023`–`P0027`/`P0002` | banner đỏ, `decision_note` giữ nguyên | thiết kế LAB-4 |

## Data Inventory

> Chỉ field chỉ đọc. Ô nhập của Khối C nằm ở `## Validation & Error Feedback`.

### Khối A — thông tin phiếu (chỉ đọc)

| Field | Display Label | Source | Format | Empty Behavior | Cross-ref |
|-------|---------------|--------|--------|----------------|-----------|
| `request_code` | Mã phiếu / 申請番号 *(đề xuất)* | `correction_requests.request_code` | dạng `COR-YYYYMMDD-0001` | luôn có | correction_requests *(mới, đề xuất)* |
| `target` | Bản ghi đích / 対象レコード *(đề xuất)* | `correction_requests.target_table`, `.target_record_id` | liên kết sang SCR012/SCR013 của bản ghi gốc | luôn có | trades, deliveries |
| `target_business_date` | Ngày nghiệp vụ của bản ghi / 対象業務日 | `correction_requests.target_business_date` | luôn `closed`; hiện kèm `closed_at` | luôn có | business_days |
| `correction_type` | Kiểu điều chỉnh / 訂正区分 *(đề xuất)* | `correction_requests.correction_type` | `reverse` \| `delta` | luôn có | correction_requests |
| `reason` | Lý do / 理由 | `correction_requests.reason` | hiện toàn văn, **không cắt ngắn** | luôn có | correction_requests |
| `evidence_ref` | Chứng cứ đính kèm / 証跡 | `correction_requests.evidence_ref` | liên kết mở tab mới khi khớp `^https?://` | luôn có | correction_requests |
| `requested_by_name` | Người 起票 / 起票者 | `app_users.full_name` | text | "—" nếu RLS chặn — xem Dependencies and Risks | app_users |
| `requested_at` | Lúc 起票 / 起票日時 | `correction_requests.requested_at` | JST | luôn có | correction_requests |
| `decision_state` | Trạng thái / 申請状態 *(đề xuất)* | `correction_requests.decision_state` | badge ∈ `pending` `approved` `rejected` `cancelled` | luôn có | correction_requests |

### Khối B — so sánh trước / sau `[KH — yêu cầu bắt buộc của màn]`

Một bảng ba cột, mỗi dòng là một trường bị đổi. Trường không đổi **không hiện dòng** — để mắt người duyệt rơi đúng vào chỗ thay đổi.

| Field | Display Label | Source | Format | Empty Behavior | Cross-ref |
|-------|---------------|--------|--------|----------------|-----------|
| `field_label` | Trường / 項目 | khoá của `snapshot_before` | ∈ Số lượng · Đơn giá · Thành tiền | — | correction_requests |
| `value_before` | Trước điều chỉnh / 訂正前 | `trades`/`deliveries` hiện tại | đọc **lại từ bản ghi gốc lúc mở màn**, không đọc từ `snapshot_before` | — | trades, deliveries |
| `value_after` | Sau điều chỉnh / 訂正後 | `correction_requests.proposed_after jsonb` | `reverse` → hiện `0` ở mọi dòng | — | correction_requests |
| `value_delta` | Chênh lệch / 差分 | `v_correction_diff.delta` | `value_after − value_before`; âm hiện dấu `−` và tông đỏ; **server tính, không tính ở client** | — | v_correction_diff *(mới, đề xuất)* |
| `effective_business_date` | Ngày nghiệp vụ ghi nhận / 計上業務日 *(đề xuất)* | `correction_requests.effective_business_date` | `business_days.status='open'`; **luôn khác `target_business_date`** | — | business_days |
| `drift_warning` | Cảnh báo lệch ảnh chụp / 現値相違の警告 *(đề xuất)* | so sánh ở server | hiện **chỉ khi** `value_before ≠ snapshot_before` tương ứng | ẩn | correction_requests |

**`value_before` phải đọc lại, không dùng `snapshot_before`.** `snapshot_before` là ảnh chụp lúc 起票, có thể đã cũ hàng giờ. Người duyệt phải thấy số **đang thật**. Nếu hai số khác nhau thì `drift_warning` hiện và RPC sẽ bắn `P0024` — không được duyệt dựa trên một bức ảnh cũ.

### Khối C — vùng chỉ đọc (dấu vết người duyệt)

| Field | Display Label | Source | Format | Empty Behavior | Cross-ref |
|-------|---------------|--------|--------|----------------|-----------|
| `decided_by` | Người duyệt / 決裁者 | `app_users.id` | **phải khác `requested_by`**; phải có `can_approve`; mặc định người đang đăng nhập | — | app_users |
| `decided_at` | Lúc duyệt / 決裁日時 | `correction_requests.decided_at` | JST, **do server đặt** — không nhận từ client | — | correction_requests |

### Khối D — bản ghi điều chỉnh đã sinh (chỉ hiện khi `decision_state = 'approved'`) `[KH — F07-03, F07-04]`

| Field | Display Label | Source | Format | Empty Behavior | Cross-ref |
|-------|---------------|--------|--------|----------------|-----------|
| `entry_type` | Loại bản ghi / 仕訳区分 *(đề xuất)* | `correction_entries.entry_type` | `reverse` (đảo bản ghi gốc) · `delta` (ghi chênh lệch) | — | correction_entries *(mới, đề xuất)* |
| `entry_business_date` | Ngày nghiệp vụ / 業務日 | `correction_entries.business_date` | `= effective_business_date`; **không bao giờ = `target_business_date`** | — | correction_entries |
| `entry_quantity` | Số lượng / 数量 | `correction_entries.quantity` | `reverse` → giá trị âm của bản ghi gốc | — | correction_entries |
| `entry_amount` | Thành tiền / 金額 | `correction_entries.amount` | như trên | — | correction_entries |
| `source_request` | Sinh ra từ phiếu / 起票元申請 | `correction_entries.source_request_id` | liên kết ngược về `request_code` — **bắt buộc có** `[KH — F07-04, US-17]` | — | correction_requests |
| `original_untouched` | Xác nhận bản ghi gốc / 元レコードの保全 *(đề xuất)* | — | dòng chữ cố định kèm liên kết tới bản ghi gốc để tự kiểm | — | — |

**`original_untouched` là yêu cầu hiển thị, không phải dữ liệu.** Dòng chữ cố định: *"Bản ghi gốc {target_code} của ngày {target_business_date} không thay đổi. Điều chỉnh được ghi thành bản ghi mới ở ngày {effective_business_date}."* `[KH — screen-list SC-24]`.

### Read Calls *(mục bổ sung)*

| Thao tác | Cách gọi | Tham số | Trả về |
|---|---|---|---|
| Đọc phiếu + so sánh trước/sau | `select` từ `v_correction_diff` *(view mới, `[ĐX]`)* | `request_id` | phiếu + mảng `{field, before, after, delta}` + cờ `has_drift` |
| Đọc bản ghi điều chỉnh đã sinh | `select` từ `correction_entries where source_request_id = ...` | `request_id` | mảng dòng |
| Đọc cờ quyền người xem | `select can_approve from app_users where id = auth.uid()` | | boolean |

Đọc dùng `select` qua view `security_invoker = true`.

## UI States

| State | Trigger | Visual Behavior | User Action Available | Source |
|-------|---------|----------------|-----------------------|--------|
| empty | không áp dụng — URL luôn trỏ một phiếu cụ thể | `P0001` nếu không tra ra | — | thiết kế LAB-4 |
| loading | mở trang | khung xương Khối A + B; Khối C chưa render cho tới khi biết `can_approve` và `requested_by` | none | thiết kế LAB-4 |
| success (duyệt) | RPC trả `{entries[]}` | banner xanh `role="status"`: *"Đã duyệt phiếu {request_code}. Hệ thống đã tạo {n} bản ghi điều chỉnh ở ngày {effective_business_date}. Bản ghi gốc không thay đổi."*; Khối C biến mất, **Khối D xuất hiện ngay bên dưới**; focus chuyển về banner | dismiss | thiết kế LAB-4 |
| success (từ chối) | RPC trả `{decision_state:'rejected'}` | banner trung tính: *"Đã từ chối phiếu {request_code}. Không có bản ghi nào được tạo."*; Khối D **không** xuất hiện | dismiss | thiết kế LAB-4 |
| error (nghiệp vụ) | RPC từ chối | banner đỏ `role="alert"` **tự nhận focus**; `decision_note` đã gõ **giữ nguyên**; hộp thoại xác nhận không tự đóng | sửa | thiết kế LAB-4 |
| error (mạng) | Server Action throw | banner trong hộp thoại + nút "Thử lại"; **`decision_note` giữ nguyên** | thử lại | thiết kế LAB-4 |
| read-only | `decision_state ≠ 'pending'`, hoặc người xem không có `can_approve`, hoặc người xem là `requested_by` | Khối A + B + D đầy đủ; thay chỗ Khối C bằng dấu vết hai người (起票者/決裁者) hoặc `V-03` | xem | thiết kế LAB-4 |
| saving | sau khi xác nhận gửi quyết định | nút `disabled` | none | thiết kế LAB-4 |

## Validation & Error Feedback

### A) Client-side

Khối C — chỉ hiện khi `decision_state = 'pending'` và người xem đủ điều kiện:

| Field | Type | Required | Constraints | Async Check | Error Message |
|-------|------|----------|-------------|-------------|---------------|
| `decision` / 決裁 | select | yes | ∈ `approved` `rejected`; lưu `correction_requests.decision_state` | không | `V-01` (xem Error Catalogue) |
| `decision_note` / 決裁意見 | text (nhiều dòng) | yes khi `decision = rejected` | 10–1000 ký tự khi bắt buộc; ≤ 1000 khi không; lưu `correction_requests.decision_note` | không | `V-02` (xem Error Catalogue) |

### B) Server-side

#### Duyệt / từ chối
- **Endpoint:** `rpc('decide_correction_request')` *(đề xuất)*
- **Request:** `p_request_id, p_decision, p_note`
- **Success:** `jsonb {decision_state, entries[], effective_business_date}` → banner xanh/trung tính, Khối D (nếu duyệt)
- **Errors:** `P0001` · `P0005` · `P0023` · `P0024` · `P0025` · `P0026` · `P0027` · `P0002` (chi tiết ở Error Catalogue)
- **Trigger:** "Gửi quyết định" → hộp thoại xác nhận
- **Source:** chưa có code — màn chỉ đặc tả ở LAB-4

Ghi **bắt buộc qua RPC `SECURITY DEFINER`** — `correction_requests` và `correction_entries` không có policy ghi, đúng khuôn ADR-002.

**Năm phép kiểm nằm trong thân RPC, cùng một transaction với `select … for update`:** `pending` · `decided_by <> requested_by` · `can_approve` · bản ghi gốc khớp `snapshot_before` · có ngày nghiệp vụ đang mở. Để ngoài RPC phép nào là để ngỏ phép đó — hai người duyệt đồng thời sẽ lọt qua mọi phép kiểm ở tầng ứng dụng.

**`correction_entries` phải append-only ở tầng database**, không chỉ ở quy ước: không policy UPDATE/DELETE, và một trigger chặn `UPDATE`/`DELETE` kể cả của owner — cùng vai trò lưới an toàn với `guard_closed_day`. Không có nó thì "không ghi đè lịch sử" chỉ là lời hứa của tầng ứng dụng, đúng thứ ADR-004 đã bác.

#### Error Catalogue *(mục bổ sung)*

| Mã | Điều kiện kích hoạt | Thông báo VI | Thông báo JA | Hiện ở đâu | Chặn gửi? |
|---|---|---|---|---|:---:|
| `V-01` | ``decision`` chưa chọn | Phải chọn duyệt hoặc từ chối. | 承認・却下のいずれかを選択してください。 | dưới ô | ● |
| `V-02` | ``decision` = rejected` mà ``decision_note`` < 10 ký tự | Từ chối phải nêu lý do từ 10 ký tự. | 却下理由は10文字以上で入力してください。 | dưới ô | ● |
| `V-03` | người xem là `requested_by`, phiếu `pending` | Bạn không duyệt được phiếu do chính mình lập. | 自身が起票した申請は決裁できません。 | thay chỗ Khối C | ● — ẩn hẳn khối |
| `P0001` | `id` trên URL không tra ra phiếu nào | Không tìm thấy phiếu này. | 指定の申請票が見つかりません。 | banner đầu trang | ● |
| `P0005` | role ≠ `reconcile` | Chức danh của bạn không được phép mở phiếu điều chỉnh. | あなたの職掌では修正申請票を開けません。 | `/403` | ● |
| `P0023` *(mới, `[ĐX]`)* | `decided_by = requested_by` | Người duyệt phải khác người 起票. | 決裁者は起票者と異なる必要があります。 | banner đầu phiếu | ● |
| `P0024` *(mới, `[ĐX]`)* | số liệu bản ghi gốc đã đổi so với `snapshot_before` | Số liệu gốc đã thay đổi từ lúc lập phiếu. Hãy tải lại và kiểm tra lại phiếu. | 申請後に元データが変更されました。再読み込みのうえご確認ください。 | banner đầu phiếu | ● |
| `P0025` *(mới, `[ĐX]`)* | `decision_state ≠ 'pending'` lúc gửi (người khác vừa xử lý) | Phiếu đã được {decided_by_name} xử lý lúc {decided_at}. | この申請は{decided_at}に{decided_by_name}が処理済みです。 | banner đầu phiếu | ● |
| `P0026` *(mới, `[ĐX]`)* | không có `business_days` nào `status='open'` và `≥ current_date` để ghi bản ghi điều chỉnh | Chưa có ngày nghiệp vụ đang mở để ghi nhận điều chỉnh. | 計上可能な未締めの業務日がありません。 | banner đầu phiếu | ● |
| `P0027` *(mới, `[ĐX]`)* | người duyệt không có cờ `can_approve` | Bạn không có thẩm quyền duyệt phiếu điều chỉnh. | 修正申請の決裁権限がありません。 | banner đầu phiếu | ● |
| `P0002` | `effective_business_date` vừa bị chốt giữa lúc duyệt | Ngày ghi nhận {business_date} vừa được chốt sổ. Hãy tải lại. | 計上業務日{business_date}が締められました。再読み込みしてください。 | banner đầu phiếu | ● |
| (network) | Server Action throw | Mất kết nối — chưa gửi được quyết định. Kiểm tra kết nối rồi thử lại. | ネットワークエラーのため決裁を送信できませんでした。 | trong hộp thoại xác nhận | ● |

Rẽ nhánh theo `error.code`; thông báo dựng từ `error.details` đã `JSON.parse` (`{decided_at}`, `{decided_by_name}`, `{business_date}`, `{requested_by}`) — **không so khớp `error.message`** (`db-schema.md` §4).

`P0024` và `P0025` là hai mã dễ bị bỏ sót nhất. Cả hai sinh ra từ cùng một thực tế: giữa lúc mở phiếu và lúc bấm duyệt, thế giới đã đổi. Không có chúng thì hai người duyệt cùng lúc sẽ sinh **hai** bộ bản ghi điều chỉnh cho cùng một phiếu — và append-only làm sai thành vĩnh viễn, không xoá lại được.

## Interaction Patterns

- **"Duyệt" và "Từ chối" đều mở hộp thoại xác nhận chặn luồng, nhắc lại `delta_amount` và `effective_business_date`** — đây là điểm thứ ba của `CAP-HUMAN-01` (*"phê duyệt sửa dữ liệu nhạy cảm — hệ thống không tự duyệt"*) `[KH]`; thiết kế LAB-4.
- **Không có `Ctrl+Enter`, không có phím tắt duyệt nhanh, có chủ ý** — thao tác này sinh bản ghi không xoá lại được; thiết kế LAB-4.
- Không có yêu cầu bàn phím riêng khác — không thuộc ba màn keyboard-first (SCR007, SCR010, SCR014).

## Accessibility

| Aspect | Status | Notes |
|--------|--------|-------|
| ARIA roles/labels | partial (đặc tả) | banner thành công `role="status"`, lỗi `role="alert"`; chênh lệch âm cần dấu `−` kèm màu |
| Keyboard navigation | supported (đặc tả) | hộp thoại xác nhận; không phím tắt duyệt |
| Focus management | managed (đặc tả) | focus chuyển về banner sau thành công; banner lỗi tự nhận focus |
| Screen reader compatibility | unknown | chưa có bản dựng để kiểm |

## Conditional Rendering

| Condition | Type | Renders | Hidden | Notes |
|-----------|------|---------|--------|-------|
| `decision_state = pending` ∧ người xem có `can_approve` ∧ `requested_by <> auth.uid()` | auth | Khối C | — | bypass: RPC `P0023`/`P0005` vẫn chặn |
| người xem là `requested_by` | auth | thông báo `V-03` | Khối C | |
| không có `can_approve` | auth | Khối A + B + D | Khối C | |
| `decision_state = approved` | hardcoded-id | Khối D + `original_untouched` | Khối C | |
| `decision_state = rejected` | hardcoded-id | banner "Không có bản ghi nào được tạo" | Khối D | |
| `has_drift` | hardcoded-id | `drift_warning` | — | RPC `P0024` |

## Component Variants

Không áp dụng — màn không có component đổi hình dạng theo biến thể; khác biệt theo chức danh/trạng thái đã mô tả ở Conditional Rendering.

## Security Surface

| Guard | Type | Consequence if bypassed |
|-------|------|------------------------|
| `lib/auth/route-policy.ts` + `proxy.ts` (route) *(đề xuất)* | role | role sai mở được màn |
| RLS SELECT trên `correction_requests` (dữ liệu) | permission | đọc ngoài chức danh |
| **bên trong RPC `decide_correction_request`** (bất biến nghiệp vụ) | permission | tầng duy nhất đáng tin: `US-17` ghi thẳng *"Người xin và người duyệt luôn là hai người khác nhau — **cưỡng chế ở tầng dữ liệu**"* `[KH]`; theo ADR-004, bất biến đặt ở Server Action là quy ước code, người viết RPC sau không có nó; `[UNVERIFIED]` — RPC chưa tồn tại |

Quyền theo chức danh: `sys_admin` ✗ · `reconcile` xem — xem được mọi phiếu; **không có khối duyệt** · `reconcile` có cờ `can_approve` *(đề xuất)* xem + ghi — khối duyệt hiện **chỉ khi** `requested_by <> auth.uid()` · `trading` · `receiving` ✗. Chặn ở **ba tầng**: route → RLS → RPC.

## Source References

Chưa có code — màn chỉ đặc tả ở LAB-4. Tài liệu nguồn đã đọc:

1. (tài liệu gốc trong repo, không nộp) `sc-24-chi-tiet-phieu-va-duyet.md` — spec 10 mục gốc
2. `lib/auth/route-policy.ts:17-27` — `ROUTE_POLICY` hiện có (chưa có `/corrections`)
3. (tài liệu gốc trong repo, không nộp) `adr.md` — ADR-002, ADR-004

## Source Walkthrough

Chưa có code — màn chỉ đặc tả ở LAB-4.

### Call Hierarchy

```text
(thiết kế) /corrections/[id] page -> v_correction_diff + correction_entries + app_users.can_approve (đọc)
   -> Server Action -> rpc decide_correction_request -> correction_entries (append-only) + correction_requests
```

## Dependencies & Risks *(mục bổ sung)*

- **Cần trước:** `SCR022` (phiếu), `SCR023` (đường vào), bảng `correction_requests` + `correction_entries`, RPC `decide_correction_request`, cờ `app_users.can_approve`.
- **Cần đồng bộ:** `v_daily_reconciliation` (`SCR018`) và bảng tiền khuyến khích (`SCR025`) phải **gộp `correction_entries` vào số của ngày/kỳ `effective_business_date`**. Làm màn này mà không sửa view thì bản ghi điều chỉnh nằm chết trong bảng, không vào báo cáo nào — đúng cái bẫy "dữ liệu chết".
- **Dữ liệu phải có sẵn:** hai tài khoản `reconcile` (một có `can_approve`), một ngày đã chốt có giao dịch, một ngày đang mở. Seed hiện thiếu cả hai điều kiện đầu (`db-schema.md` §8).
- **Dễ hiểu sai #1 — nghiêm trọng nhất:** duyệt xong chạy `UPDATE trades SET quantity = ...`. Màn hình vẫn "đúng", báo cáo vẫn khớp, và **toàn bộ lý do tồn tại của F07 bị xoá** — in lại báo cáo kỳ cũ sẽ ra số mới. Phép thử một câu: nếu trong RPC duyệt có bất kỳ `UPDATE`/`DELETE` nào trên `trades`/`deliveries` thì đã sai.
- **Dễ hiểu sai #2:** ghi `correction_entries.business_date = target_business_date`. Lúc đó trigger `guard_closed_day` sẽ chặn (`P0009`) — và phản xạ sai là *nới trigger*. Trigger đúng; cái sai là ngày. Bản ghi điều chỉnh thuộc về **kỳ sau** `[KH — FR-INC-03, INC-EDGE-02]`.
- **Dễ hiểu sai #3:** so sánh Khối B dựng từ `snapshot_before`. Người duyệt sẽ duyệt một bức ảnh cũ. Phải đọc lại bản ghi gốc (`value_before`) và so (`drift_warning`, `P0024`).
- **Dễ hiểu sai #4:** tin vào việc `SCR023` đã ẩn nút. `US-17` đòi cưỡng chế ở tầng dữ liệu `[KH]`.
- **Rủi ro RLS:** ``requested_by_name`` và `{decided_by_name}` của `P0025` có thể ra "—" vì RLS `app_users` chỉ cho tự xem mình + `sys_admin`. Người duyệt không biết đang duyệt phiếu của ai — xem `SCR023` Dependencies and Risks, phải xử lý trước khi màn này có nghĩa.
- **Rủi ro tương thích:** `trades.amount` là cột `GENERATED`. `correction_entries` nếu sao chép khuôn đó thì `reverse` không ghi được số âm nếu CHECK ràng buộc `> 0`. Phải thiết kế `correction_entries` cho phép **số âm** ngay từ migration đầu — sửa sau là sửa dữ liệu đã sinh.
- **Rủi ro vận hành:** không có thao tác hoàn tác. Duyệt nhầm thì cách sửa duy nhất là **lập một phiếu điều chỉnh ngược** qua `SCR022`. Giao diện phải nói điều này trong hộp thoại xác nhận, không để người dùng tự phát hiện.

### Unresolved Questions *(mục bổ sung)*

1. **Số cấp duyệt — điểm chặn lớn nhất, LAB-1 để ngỏ.** Câu 3 gửi khách (tài liệu LAB-1 `product-a-rfp-qa-part-a-client.md`): đề xuất của LAB-1 là **một cấp** (sau chốt sổ → trưởng bộ phận đối chiếu), maker ≠ checker, **không có cấp thứ ba**. Khách chưa xác nhận, và tài liệu 業務状態マトリクス khách nhắc tới thì chưa gửi. Nếu phân cấp theo số tiền (ví dụ `< 1.000.000 JPY` → trưởng phòng, `≥` → giám đốc) thì: Khối C tách theo cấp, cần một màn định nghĩa ngưỡng, `F07-02` nở thành 3–5 luồng `[KH — screen-list SCR024 ghi đúng hệ quả này]`.
2. **`reverse` sinh một dòng hay hai?** `[ĐX]` hiện: `reverse` = một dòng đảo (số âm); nếu người dùng muốn thay bằng số mới thì lập tiếp một phiếu `delta`. Phương án hai (đảo + lập lại trong cùng một phiếu) ít thao tác hơn nhưng khiến một phiếu sinh hai bản ghi khác bản chất. Chưa có yêu cầu nào quyết.
3. **Điều chỉnh tiền khuyến khích có đi chung luồng này không?** `FR-INC-03` nói *"sinh delta ở kỳ kế tiếp"* — cùng nguyên tắc, nhưng đơn vị kỳ là **tháng**, không phải **ngày nghiệp vụ**. ``effective_business_date`` lúc đó phải là kỳ, không phải ngày. Tài liệu này giả định dùng chung bảng, tách bằng `target_table='incentive_results'`; cần xác nhận.
4. **Phiếu bị từ chối có lập lại được không?** `[ĐX]` hiện: lập phiếu mới, phiếu cũ giữ nguyên làm dấu vết. Không có "sửa rồi gửi lại".
5. **Có cần xác thực hai lớp khi duyệt không?** `SCR002` của LAB-1 ghi *"xác thực hai lớp cho quản trị và chức danh có quyền duyệt"*. Nếu áp vào đây thì bước 8 có thêm một chặng.
6. **Bản ghi điều chỉnh có đi sang hệ thống kế toán (`IF-ACC-01`) không, và đi dưới trạng thái nào?** Hợp đồng dữ liệu đã chốt có `行ステータス ∈ {保留, 確認済み, 調整済み}` — `調整済み` nhiều khả năng chính là dòng sinh từ màn này, nhưng tài liệu khách không nối hai chỗ đó lại.
