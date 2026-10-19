# SCR009_LotDetail — Screen Spec

> Alias LAB-1: **SC-09** · ロット詳細 · Chi tiết lô hàng

**Screen**: SCR009_LotDetail: Chi tiết lô hàng
**Feature**: F003_LotsAndInspection — [screens.md](../../features/F003_LotsAndInspection/screens.md)
**Type**: composite *(REG001 thông tin lô · REG002 đánh giá cảm quan + lịch sử · REG003 hộp thoại điều chỉnh thuộc tính — xem [screen-list.md](../../generated/screen-list.md))*
**Route**: `/lots/[id]` *(đề xuất)*
**Generated**: 2026-10-06
**Trạng thái**: Thiết kế mới, chưa xây (LAB-3 không dựng màn này)
**Tính năng LAB-1**: `F03-02` `F03-03` `F03-05`
**User story**: `US-02` (ghi kết quả đánh giá cảm quan). Phần điều chỉnh thuộc tính không có US riêng — truy về `FR-LOT-04`.

## Purpose

Mở một lô để xem toàn bộ thuộc tính, **ghi kết quả đánh giá cảm quan (目利き)**, và **điều chỉnh thuộc tính lô có kiểm soát**. Người dùng: nhân viên tiếp nhận/đánh giá ở khu hiện trường (từ 02:00 JST, đeo găng tay, máy dùng chung theo ca); bộ phận đối chiếu mở để giải thích số liệu, chỉ xem.

Hai điều kiện tuyệt đối từ khách (`CAP-HUMAN-01` điểm 1): hệ thống **chỉ ghi lại** kết quả đánh giá — không gợi ý, không tự phân loại, không đọc ảnh; và mọi kết quả phải truy được **ai ghi, lúc nào** (`FR-LOT-02`).

## Screen Layout

Khung `AppLayout`; nội dung gồm: dải thông báo "Ngày nghiệp vụ đã chốt sổ" (khi `dayStatus = closed`), khối thông tin lô (3.1, chỉ đọc), khối "Kết quả đánh giá cảm quan" (form ghi + lịch sử, 3.2) và nút "Điều chỉnh thuộc tính" mở hộp thoại (3.3). Hộp thoại đóng được bằng `Escape`. *Bố cục là đề xuất, chưa có ảnh/wireframe.*

### Layout Sketch

```
┌───────────────────────────────────────────────────────────┐
│ R1: AppHeader + R2: AppNav (khung chung)                  │
├───────────────────────────────────────────────────────────┤
│ R3: PageHeader  "Lô LOT-YYYYMMDD-NNNN"   [Điều chỉnh ▸]   │
│ R4: Dải "Ngày nghiệp vụ đã chốt sổ… → SCR022" - - - - -   │
│ R5: Thông tin lô (REG001): mã, ngày, trạng thái ngày,     │
│     chủ hàng, mặt hàng, SL đầu, đã bán, còn lại, trạng    │
│     thái, thời điểm tiếp nhận                             │
│ R6: Kết quả đánh giá (REG002): [textarea][người][thời     │
│     điểm][☑ chuyển Đã xem hàng] [Ghi kết quả] + lịch sử   │
│   ┌ R7: Hộp thoại Điều chỉnh (REG003, nổi) - - - - - ┐    │
│   │ thuộc tính▾ | trước | sau | lý do | ☑ xác nhận    │    │
│   └───────────────────────────────────────────────────┘    │
└───────────────────────────────────────────────────────────┘
```

### Layout Regions

| Region ID | Name | Position | Scrollable | Key Components | Responsive Behavior |
|-----------|------|----------|------------|----------------|---------------------|
| R1/R2 | Header + Nav | static | no | khung chung | — |
| R3 | Tiêu đề + nút điều chỉnh | static | no | `PageHeader` | nút ẩn khi ngày đã chốt hoặc không phải `receiving` |
| R4 | Dải ngày đã chốt | static (có điều kiện) | no | banner + link `SCR022_CorrectionRequestCreate` | chỉ khi `closed` |
| R5 | Thông tin lô (REG001) | static | no | khối chỉ đọc | cột đơn |
| R6 | Đánh giá + lịch sử (REG002) | static | no | form + bảng lịch sử | ẩn form khi ngày đã chốt |
| R7 | Hộp thoại điều chỉnh (REG003) | floating | no | dialog, đóng bằng `Escape` | mở từ R3 |

## Child Routes

Không áp dụng — màn này là một route đơn, không phải khung chứa route con.

## User Flow

> **Scope:** within-screen interactions only.

### Happy Path

1. Từ `SCR008_LotList` bấm vào mã lô → `/lots/[id]`.
2. Trang hiện 3 khối: thông tin lô (3.1), kết quả đánh giá + lịch sử (3.2), nút "Điều chỉnh thuộc tính" (mở hộp thoại 3.3).
3. **Nhánh đánh giá:** gõ kết quả → tên người đánh giá → xác nhận thời điểm → (tuỳ chọn) bỏ tick "Chuyển sang Đã xem hàng" → "Ghi kết quả".
4. RPC `record_lot_evaluation` trong một transaction: kiểm role → kiểm ngày chốt (`P0002`) → kiểm trạng thái lô (`P0060`) → insert `lot_evaluations` → nếu `markAsPreviewed` và `status = 受付` thì `UPDATE lots.status = '下見済'` → ghi `audit_log`.
5. **Nhánh điều chỉnh:** mở hộp thoại → chọn thuộc tính → hệ thống hiện giá trị trước → nhập giá trị sau → nhập lý do → tick xác nhận → "Xác nhận điều chỉnh".
6. RPC `adjust_lot_attributes`: `SELECT ... FOR UPDATE` dòng lô → kiểm role → kiểm ngày chốt → nếu sửa `quantity` thì đọc `v_lot_availability.sold_qty` và chặn `P0061` → `UPDATE lots` → insert `lot_adjustments` (giá trị trước/sau/lý do/người thực hiện) → ghi `audit_log`.
7. Thành công → đóng hộp thoại, khối 3.1 nạp lại số mới, banner xanh.

### Branches

| Decision point | Condition | Outcome on this screen | Source |
|----------------|-----------|------------------------|--------|
| Bước 7 | thất bại | hộp thoại **không đóng**, banner đỏ trong hộp thoại, dữ liệu giữ nguyên | thiết kế |
| Bước 1 | `id` không có trong `lots.id` | trang 404 (`notFound()`), không phải banner (`P0001`) | thiết kế |
| Bước 3 | `lots.status ∉ {受付, 下見済}` | `P0060` — không ghi thêm đánh giá | thiết kế |

## Data Inventory

> Khối 3.1 (chỉ đọc) + cột lịch sử chỉ đọc. Field nhập ở `## Validation & Error Feedback`.

| Field | Display Label | Source | Format | Empty Behavior | Cross-ref |
|-------|---------------|--------|--------|----------------|-----------|
| `lotCode` (#1) | Mã lô / ロット番号 | `lots.lot_code` | định dạng `LOT-YYYYMMDD-NNNN`, duy nhất | — | `lots` |
| `businessDate` (#2) | Ngày nghiệp vụ / 業務日 | `lots.business_date` | date | — | `lots` |
| `dayStatus` (#3) | Trạng thái ngày / 業務日の状態 | `business_days.status` theo `lots.business_date` | badge ∈ `open` `closed` — `closed` ⇒ toàn màn chuyển chỉ đọc | — | `business_days` |
| `consignor` (#4) | Chủ hàng / 荷主 | `participants` theo `lots.consignor_id` | `{code} · {name}` | "" nếu không tra được | `participants` |
| `commodity` (#5) | Mặt hàng / 品目 | `commodities` theo `lots.commodity_id` | `{code} · {tên theo locale}` | "" nếu không tra được | `commodities` |
| `quantity` (#6) | Số lượng ban đầu / 初期数量 | `lots.quantity` | `numeric(14,3)`, `> 0` + đơn vị | — | `lots.quantity` |
| `soldQty` (#7) | Đã bán / 販売済み | `v_lot_availability.sold_qty` — **KHÔNG phải cột lưu sẵn**, tính mỗi lần đọc | số | 0 | `v_lot_availability` |
| `remaining` (#8) | Còn lại / 残数量 | `v_lot_availability.remaining = quantity − soldQty` (ranh giới cứng ADR-003) | số | `= quantity` | `v_lot_availability` |
| `status` (#9) | Trạng thái lô / 状態 | `lots.status` — enum `lot_status`: `受付` `下見済` `公開` `確定` `引渡し完了` | badge | — | `lots.status` |
| `receivedAt` (#10) | Thời điểm tiếp nhận / 受付日時 | `lots.received_at` | datetime | — | `lots` |
| lịch sử: `actor` (#15) | Tài khoản ghi / 記録アカウント | `app_users.full_name` theo `lot_evaluations.actor_id` (truy vấn thứ hai) | text | — | `app_users` |
| lịch sử: `recordedAt` (#16) | Lúc ghi vào hệ thống / システム記録日時 | `lot_evaluations.recorded_at` — dấu vết hệ thống, **không sửa được** | datetime | — | `lot_evaluations` |

Nhãn JA của màn này là **đề xuất** (không có trong `04_画面一覧`).

## UI States

| State | Trigger | Visual Behavior | User Action Available | Source |
|-------|---------|----------------|-----------------------|--------|
| empty | lô chưa có lần đánh giá nào | khối lịch sử hiện "Chưa ghi kết quả đánh giá nào." / 目利き結果はまだありません。 | ghi đánh giá | thiết kế |
| loading/saving | sau khi bấm "Ghi kết quả" / "Xác nhận điều chỉnh" | nút `disabled`, nhãn đổi "Đang ghi…" | none | thiết kế |
| success | RPC trả về | banner xanh `role="status"`: "Đã ghi kết quả đánh giá lúc {recorded_at}."; lịch sử thêm 1 dòng trên cùng; form xoá sạch; focus về ô "Kết quả đánh giá" | tiếp tục | thiết kế |
| error (nghiệp vụ) | `P0002`/`P0005`/`P0060`/`P0061` | banner đỏ `role="alert"`, tự nhận focus; **dữ liệu đã gõ giữ nguyên** | sửa | thiết kế |
| error (mạng) | Server Action throw | cùng banner đỏ; **dữ liệu đã gõ giữ nguyên** — gọi thẳng Server Action trong `onSubmit` + `useTransition` + `try/catch`, KHÔNG dùng `useActionState` (cùng lý lẽ `SCR007`/`SCR014`) | Thử lại | thiết kế |
| read-only | `business_days.status = 'closed'` của ngày lô | ẩn hẳn form đánh giá và nút điều chỉnh; hiện dải "Ngày nghiệp vụ đã chốt sổ — mọi sửa đổi đi qua phiếu điều chỉnh (SCR022)" + liên kết sang `SCR022` | mở `SCR022` | thiết kế |

## Validation & Error Feedback

### A) Client-side

| Field | Type | Required | Constraints | Async Check | Error Message |
|-------|------|----------|-------------|-------------|---------------|
| `resultNote` (#11) | textarea | yes | `10 ≤ độ dài ≤ 1000` ký tự; **không có gợi ý, không autocomplete, không ô tải ảnh** (`FR-SERI-01` cùng tinh thần) | không | `V-eval-required`/`V-eval-len` |
| `evaluatedBy` (#12) | text | yes | `2 ≤ độ dài ≤ 100`; **nhập tay tên người đánh giá tại hiện trường**, khác `actor_id` của phiên đăng nhập (máy dùng chung theo ca — cùng lý lẽ `auction_confirmed_by_name`, (tài liệu gốc trong repo, không nộp) `architecture.md` §3.6) | không | `V-eval-required` |
| `evaluatedAt` (#13) | datetime-local | yes | `≤ now()`; phần date phải `= lots.business_date` | không | `V-eval-required`/`V-eval-date` |
| `markAsPreviewed` (#14) | checkbox | no | chỉ bật khi `lots.status = 受付`; ẩn ở trạng thái khác | không | — |
| `targetField` (#17) | select | yes | ∈ `consignor_id` `commodity_id` `quantity` — ngoài 3 giá trị này bị RPC từ chối | không | — |
| `newValue` (#19) | text/number/select theo `targetField` | yes | khác `oldValue`; nếu `targetField='quantity'` thì `> 0` **và** `≥ v_lot_availability.sold_qty` (nếu không → `P0061`) | không | `P0061` |
| `reason` (#20) | textarea | yes | `10 ≤ độ dài ≤ 1000` ký tự | không | `V-adj-reason` |
| `confirm` (#21) | checkbox | yes | phải `true` mới bật nút gửi | không | — |

`oldValue` (#18, "Giá trị trước"): readonly, **đọc lại trong RPC bằng `SELECT ... FOR UPDATE`, không tin giá trị client gửi lên**, nguồn `lots.<targetField>`.

#### Field Labels & Defaults *(mục bổ sung)*

| # | Field | Nhãn VI | Nhãn JA *(đề xuất)* | Mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|
| 11 | `resultNote` | Kết quả đánh giá | 目利き結果 | rỗng | `lot_evaluations.result_note` |
| 12 | `evaluatedBy` | Người đánh giá | 評価者 | rỗng | `lot_evaluations.evaluated_by_name` |
| 13 | `evaluatedAt` | Thời điểm đánh giá | 評価日時 | `now()` làm tròn phút | `lot_evaluations.evaluated_at` |
| 14 | `markAsPreviewed` | Chuyển lô sang "Đã xem hàng" | 「下見済」に変更 | bật | điều khiển nhánh `lots.status → 下見済` trong RPC |
| 17 | `targetField` | Thuộc tính cần sửa | 修正対象項目 | rỗng | — |
| 18 | `oldValue` | Giá trị trước | 変更前 | từ `lots` | `lots.<targetField>` |
| 19 | `newValue` | Giá trị sau | 変更後 | rỗng | `lot_adjustments.new_value` |
| 20 | `reason` | Lý do điều chỉnh | 修正理由 | rỗng | `lot_adjustments.reason` |
| 21 | `confirm` | Tôi xác nhận điều chỉnh này | 上記内容で修正します | tắt | — |

Bảng `lot_evaluations` (#11–#16) và `lot_adjustments` (#17–#21) là **đề xuất**, chưa có trong schema.

#### Mã lỗi hiển thị

| Mã | Điều kiện kích hoạt | Thông báo VI | Thông báo JA | Hiện ở đâu | Chặn gửi? |
|---|---|---|---|---|:---:|
| `V-eval-required` | `resultNote` / `evaluatedBy` / `evaluatedAt` rỗng | Vui lòng nhập kết quả, tên người đánh giá và thời điểm. | 結果・評価者・日時を入力してください。 | banner đầu khối đánh giá, `aria-live="assertive"` | ● |
| `V-eval-len` | `resultNote` < 10 hoặc > 1000 ký tự | Kết quả đánh giá phải từ 10 đến 1.000 ký tự. | 目利き結果は10〜1,000文字で入力してください。 | dưới ô | ● |
| `V-eval-date` | phần ngày của `evaluatedAt` ≠ `lots.business_date` | Thời điểm đánh giá phải nằm trong ngày nghiệp vụ {business_date}. | 評価日時は業務日{business_date}内である必要があります。 | dưới ô | ● |
| `V-adj-reason` | `reason` của hộp thoại điều chỉnh < 10 ký tự | Vui lòng nêu lý do điều chỉnh, tối thiểu 10 ký tự. | 修正理由を10文字以上で入力してください。 | trong hộp thoại | ● |
| `P0001` | `id` trong URL không có trong `lots.id` | Không tìm thấy lô hàng. | 該当するロットが見つかりません。 | trang 404 (`notFound()`), không phải banner | ● |
| `P0002` | ngày nghiệp vụ của lô đã chốt sổ (kiểm **trong RPC** trước khi ghi) | Ngày nghiệp vụ {business_date} đã chốt sổ lúc {closed_at}. Muốn sửa phải lập phiếu điều chỉnh (SCR022). | 業務日{business_date}は{closed_at}に締められています。修正は訂正申請（SCR022）から行ってください。 | banner | ● |
| `P0005` | role ≠ `receiving` | Chức danh của bạn không được phép ghi vào lô hàng. | あなたの職掌ではロットへの記録を行えません。 | banner | ● |
| `P0060` *(mới)* | ghi đánh giá khi `lots.status ∉ {受付, 下見済}` — lô đã `公開`/`確定`/`引渡し完了` | Lô đang ở trạng thái {status}, không ghi thêm kết quả đánh giá được. | ロットは{status}状態のため、目利き結果を追加記録できません。 | banner | ● |
| `P0061` *(mới)* | điều chỉnh `quantity` xuống dưới số đã bán — `details: {requested, sold_qty, sold_to[]}` | Lô đã bán {sold_qty} {unit} cho {sold_to}; không hạ số lượng xuống {requested} {unit} được. | このロットは{sold_to}へ{sold_qty} {unit}販売済みです。{requested} {unit}へ引き下げられません。 | banner trong hộp thoại | ● |
| `P0009` | ghi trực tiếp vào `lots` của ngày đã chốt lọt qua trigger `guard_closed_day` | (dùng chung câu của `P0002`) | (nt) | banner | ● |
| (network) | Server Action throw | Mất kết nối — chưa ghi được. Dữ liệu đang nhập vẫn giữ nguyên. | 通信が切断され、記録できませんでした。入力内容は保持されています。 | banner | ● |

`error.details` là **chuỗi JSON** → `JSON.parse()` rồi dựng câu; không so khớp `error.message` (`lib/domain/rpc-errors.ts:46-66`).

### B) Server-side

#### Ghi kết quả đánh giá
- **Endpoint:** `rpc('record_lot_evaluation', { p_lot_id, p_result_note, p_evaluated_by_name, p_evaluated_at, p_mark_previewed })` *(RPC mới, đề xuất)*
- **Request:** như tham số RPC
- **Success:** `jsonb {evaluation_id, lot_status}` → banner xanh, lịch sử thêm dòng
- **Errors:** `P0001` | `P0002` | `P0005` | `P0060`
- **Trigger:** nút "Ghi kết quả"
- **Source:** thiết kế — chưa có code

#### Điều chỉnh thuộc tính lô
- **Endpoint:** `rpc('adjust_lot_attributes', { p_lot_id, p_field, p_new_value, p_reason })` *(RPC mới, đề xuất)*
- **Request:** như tham số RPC
- **Success:** `jsonb {old_value, new_value}` → đóng hộp thoại, khối 3.1 nạp lại
- **Errors:** `P0001` | `P0002` | `P0005` | `P0061`
- **Trigger:** nút "Xác nhận điều chỉnh"
- **Source:** thiết kế — chưa có code

Hai đường ghi **bắt buộc qua RPC** dù `lots` CÓ policy UPDATE cho `receiving` (`supabase/migrations/0009_rls_policies.sql:48-51`): cả hai thao tác phải ghi kèm bản ghi lịch sử trong **cùng một transaction**, và UPDATE trực tiếp qua RLS không bảo đảm điều đó. Đây là khác biệt có chủ ý so với nút "Mở bán" của `SCR008` (UPDATE trực tiếp, không cần lịch sử kèm theo).

## Interaction Patterns

- **Hộp thoại điều chỉnh đóng được bằng `Escape`; không có hành động nào chỉ hiện khi hover** (người dùng đeo găng tay) — source: thiết kế.
- **Chọn thuộc tính cần sửa thì hiện ngay giá trị trước** — source: thiết kế.
- **Không có yêu cầu bàn phím riêng** (ba màn ⌨️ bắt buộc là `SCR007`/`SCR010`/`SCR014`).

## Accessibility

| Aspect | Status | Notes |
|--------|--------|-------|
| ARIA roles/labels | unknown | chưa có code; yêu cầu: banner lỗi `role="alert"`, thành công `role="status"`, dùng `Field` (`components/ui/field.tsx:23-57`) |
| Keyboard navigation | unknown | hộp thoại đóng bằng `Escape` (yêu cầu) |
| Focus management | unknown | banner lỗi tự nhận focus; sau thành công focus về ô "Kết quả đánh giá" (yêu cầu) |
| Screen reader compatibility | unknown | — |

`[NO_A11Y_DETECTED]` — màn chưa có code; cần audit khi xây.

## Conditional Rendering

| Condition | Type | Renders | Hidden | Notes |
|-----------|------|---------|--------|-------|
| `business_days.status = 'closed'` | literal | dải "đã chốt sổ" + link `SCR022` | form đánh giá, nút điều chỉnh | mọi sửa đổi đi qua F007 |
| `lots.status = 受付` | literal | checkbox `markAsPreviewed` | (ẩn ở trạng thái khác) | |
| role `receiving` | auth | form ghi + nút điều chỉnh | — | `trading`, `reconcile`: chỉ xem |
| role ∉ {`receiving`,`trading`,`reconcile`} | auth | — | cả route | `/403` |

## Component Variants

Không áp dụng — màn không có component đổi hình dạng theo biến thể; khác biệt theo chức danh/trạng thái đã mô tả ở Conditional Rendering.

## Security Surface

| Guard | Type | Consequence if bypassed |
|-------|------|------------------------|
| `ROUTE_POLICY` prefix `/lots` hiện `["receiving","trading"]` → **phải thêm `"reconcile"`** (`lib/auth/route-policy.ts:19`) | role | nếu thiếu, `reconcile` bị `/403` dù LAB-1 liệt kê họ cho màn này |
| RLS `lots` SELECT hiện `receiving`+`trading` → **phải thêm `reconcile`** (`0009_rls_policies.sql:44-45`) | permission | nếu thiếu, màn trả rỗng cho `reconcile` dù route đã cho qua (cùng lớp lỗi D-05 ở `SCR012`) |
| Ghi: tầng 3 trong RPC `record_lot_evaluation`/`adjust_lot_attributes` tự kiểm `current_app_role() = 'receiving'` → `P0005` | role | bài học ADR-004 (bất biến ở tầng thấp nhất, không phải quy ước giao diện) |
| Trigger `guard_closed_day` | permission | `P0009` nếu lọt — `0010_closed_day_guard.sql:45-47` |

Ma trận quyền (xem/ghi): `sys_admin` ✗/✗ · `receiving` ●/● (đánh giá + điều chỉnh; LAB-1 ghi hai chức danh "Tiếp nhận · Đánh giá"; prototype **không có** role đánh giá riêng — đã chốt gộp 目利き vào `receiving`, PREMISE C-10) · `trading` ●/✗ (xem hàng trước khi sang `SCR010`) · `reconcile` ●/✗ (cần migration mở quyền) · `party_admin`, `rule_admin`, `ops_admin`, `market_manager`, `internal_audit` *(đề xuất)* ✗/✗. `can_approve` không liên quan màn này.

## Source References

Chưa có code — màn chỉ đặc tả ở LAB-4. Điểm chạm trong code hiện có (đã kiểm tồn tại):

1. Danh sách lô vào màn này: `app/(app)/lots/page.tsx:35-141` (`SCR008`)
2. Số đã bán/còn lại: `supabase/migrations/0006_derived_views.sql:6-22`; `lib/domain/lots.ts:57-114`
3. Bảng `lots`: `supabase/migrations/0004_business_days_and_lots.sql:29-45`; `audit_log`: `0002_core_identity.sql:28-39`
4. RLS/guard: `0009_rls_policies.sql:44-51`; `0010_closed_day_guard.sql:45-47`
5. Mẫu chống mất dữ liệu khi mất mạng: `app/(app)/lots/new/use-lot-entry-form.ts:53-129`

## Source Walkthrough

Chưa có code — màn chỉ đặc tả ở LAB-4. Thứ tự đọc để hiểu bối cảnh khi xây:

1. **File:** `supabase/migrations/0006_derived_views.sql:6-22` — số còn lại là view dẫn xuất.
2. **File:** `app/(app)/lots/page.tsx:35-141` — màn danh sách dẫn vào đây.
3. **File:** `app/(app)/lots/new/use-lot-entry-form.ts:53-129` — mẫu form giữ dữ liệu khi lỗi.

### Call Hierarchy

```text
(đề xuất) LotDetailPage -> lots + v_lot_availability + lot_evaluations + lot_adjustments (select) -> EvaluationForm -> rpc record_lot_evaluation ; AdjustDialog -> rpc adjust_lot_attributes
```

### Read Calls *(mục bổ sung)*

| Thao tác | Cách gọi | Tham số | Trả về |
|---|---|---|---|
| Đọc lô | `select` từ `lots` theo `id` | `id` | `LotRow` hoặc `null` → `notFound()` |
| Đọc đã bán / còn lại | `select` từ `v_lot_availability` | `lot_id` | `sold_qty`, `remaining`, `sold_to[]` — **không bao giờ đọc từ cột lưu sẵn** |
| Đọc trạng thái ngày | `select` từ `business_days` | `business_date` | `status`, `closed_at` |
| Đọc lịch sử đánh giá | `select` từ `lot_evaluations` + truy vấn **thứ hai** vào `app_users` để tra tên người ghi | `lot_id` | mảng, mới nhất trước |
| Đọc lịch sử điều chỉnh | `select` từ `lot_adjustments` + truy vấn thứ hai vào `app_users` | `lot_id` | mảng, mới nhất trước |
| Ghi kết quả đánh giá | `rpc('record_lot_evaluation', {...})` *(mới, đề xuất)* | `p_lot_id, p_result_note, p_evaluated_by_name, p_evaluated_at, p_mark_previewed` | `jsonb {evaluation_id, lot_status}` hoặc raise `P0001`/`P0002`/`P0005`/`P0060` |
| Điều chỉnh thuộc tính | `rpc('adjust_lot_attributes', {...})` *(mới, đề xuất)* | `p_lot_id, p_field, p_new_value, p_reason` | `jsonb {old_value, new_value}` hoặc raise `P0001`/`P0002`/`P0005`/`P0061` |

### Dependencies & Risks *(mục bổ sung)*

- Cần `SCR007` chạy trước để có lô; cần `v_lot_availability` bật `security_invoker`.
- **Mâu thuẫn phải giải trước khi code:** (tài liệu gốc trong repo, không nộp) `db-schema.md` và ADR-003 ghi `lots.quantity` là **bất biến sau khi tạo** (`0004_business_days_and_lots.sql:41-42`), nhưng `FR-LOT-04`/`F03-05` yêu cầu điều chỉnh thuộc tính lô có kiểm soát. Tài liệu này đọc "bất biến" theo nghĩa *không bị trừ tự động bởi giao dịch* (số còn lại luôn tính ra từ view), **không** theo nghĩa *không bao giờ sửa được*. Nếu chốt theo nghĩa thứ hai thì bỏ `targetField = quantity` khỏi hộp thoại và `P0061` không cần nữa.
- **Rủi ro hiểu sai:** `evaluatedBy` (tên người đánh giá, nhập tay) và `actor_id` (tài khoản đang đăng nhập) là **hai thứ khác nhau**, cố ý. Terminal dùng chung theo ca ⇒ người quyết định kết quả có thể không phải người gõ máy. Gộp hai cột làm mất đúng dấu vết mà `FR-LOT-02` đòi.
- `reconcile` hiện **không đọc được** `lots` (RLS §6 `db-schema.md`) — cùng lớp lỗi với sai lệch D-05 ở `SCR012`.
- Ghi đánh giá **không** có đường offline: LAB-1 chỉ cho `SCR007` và `SCR014` chạy khi mất mạng (`architecture.md` §3.5). Không mở rộng sang màn này nếu chưa có câu trả lời cho câu 1 gửi khách (degraded operation).
- Link chéo: [F003](../../features/F003_LotsAndInspection/screens.md); flow FL-01; ADR-003, ADR-004.

### Unresolved Questions *(mục bổ sung)*

- **Khách có chấp nhận gộp 目利き vào `receiving` không?** LAB-1 tách "Tiếp nhận · Đánh giá"; hướng đã chốt ở PREMISE C-10 là gộp (ghi là nợ: phân tách nhiệm vụ nhận/giao/đánh giá hàng là kiểm soát nội bộ). Nếu khách muốn tách thì phát sinh role thêm và ma trận quyền phải làm lại.
- **`quantity` có được điều chỉnh không** (xem `### Dependencies & Risks`) — câu hỏi chặn, quyết định cả `P0061` lẫn một nửa hộp thoại điều chỉnh.
- Kết quả đánh giá có cần **cấu trúc** (hạng, độ tươi, cỡ) hay chỉ một ô văn bản tự do? Tài liệu này chọn văn bản tự do vì `CAP-HUMAN-01` cấm hệ thống phân loại; nếu khách muốn ô có cấu trúc thì cần danh mục hạng do khách cung cấp.
- Một lô ghi được **nhiều lần** đánh giá hay chỉ một? Tài liệu này chọn append-only nhiều lần (giữ lịch sử, đúng tinh thần `BR-CLOSE-01`); cần khách xác nhận lần nào là lần "chính thức" khi có nhiều lần.
