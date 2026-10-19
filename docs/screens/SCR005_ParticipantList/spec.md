# SCR005_ParticipantList — Screen Spec

> Alias LAB-1: **SC-05** · 参加者一覧 *(tên JA đề xuất — bảng màn hình LAB-1 không có cột tên JA)* · Danh sách người tham gia

**Screen**: SCR005_ParticipantList: Danh sách người tham gia
**Feature**: F002_ParticipantsAndQualification — [screens.md](../../features/F002_ParticipantsAndQualification/screens.md)
**Type**: atomic
**Route**: `/participants` *(đề xuất)*
**Generated**: 2026-10-06
**Trạng thái**: Thiết kế mới, chưa xây (đặc tả theo bản đầy đủ của LAB-1; LAB-3 không dựng màn này)
**Tính năng LAB-1**: `F02-05` `F02-07`
**User story**: không có US riêng. Phục vụ `US-04` từ phía trước — cho tra tư cách **trước** lúc chốt, thay vì để `SCR010` trả `P0003` lúc chốt.
**Wireframe**: [sc-05-danh-sach-nguoi-tham-gia.html](../../design/wireframes/sc-05-danh-sach-nguoi-tham-gia.html)

## Purpose

Một chỗ duy nhất trả lời *"người này còn được giao dịch không, và theo căn cứ nào"*. Hiện nay hồ sơ 許可/承認 nằm rời điểm giao dịch (một phần giấy, một phần điện tử), nên bộ phận đối chiếu phải tra tay rồi gọi lại — rủi ro khách tự ghi nhận: **người hết hạn vẫn giao dịch được**.

Người dùng chính: quản lý người tham gia (làm trong giờ hành chính) và bộ phận đối chiếu. Nhân viên giao dịch mở ở chế độ chỉ xem, thường là trước phiên — không phải giữa luồng chốt, nên màn này **không** nằm trong nhóm bàn phím bắt buộc.

Hai thứ màn này phải lọc được mà prototype hiện không làm được: **lọc theo loại căn cứ tham gia** và **lọc theo trạng thái hiệu lực thật của từng căn cứ** (`条件付き` và `再審査` là hai trạng thái khác nhau, không phải một — xem D-01 trong (tài liệu gốc trong repo, không nộp) `database-diagram.md`; ADR-006).

## Screen Layout

Khung `AppLayout` chung (header + nav + `<main>`). Vùng nội dung: `PageHeader` + nút "Thêm người tham gia" (chỉ `party_admin`), dải banner (V-02 / V-03 / P0017), thanh lọc 7 ô, bộ đếm (tổng số dòng và số dòng đang `条件付き`/`再審査` tách riêng), chú giải màu badge, rồi bảng 10 cột (theo wireframe `sc-05-...html`: bảng 10 cột + 7 ô lọc, badge năm màu). Bộ lọc nằm trên URL (`searchParams`) nên giữ nguyên khi tải lại. *Bố cục chi tiết là đề xuất — chưa có code/ảnh deploy.*

### Layout Sketch

```
┌────────────────────────────────────────────────────────────────┐
│ R1: AppHeader                                                  │
├────────────┬───────────────────────────────────────────────────┤
│ R2: AppNav │ R3: PageHeader            [+ Thêm người tham gia]*│
│            │ R4: Banner V-02 / V-03 / P0017  - - - - - - - -   │
│            │ R5: Bộ lọc: q | party_type | basis_type | status  │
│            │             | expiring | warning_days | market_id**│
│            │ R6: Bộ đếm (tổng · số 条件付き/再審査) + chú giải màu│
│            │ R7: Bảng 10 cột: Mã|Tên|Loại|Căn cứ|Số hiệu|Hiệu  │
│            │     lực|Hết hạn|Trạng thái|Điều kiện|Còn lại      │
└────────────┴───────────────────────────────────────────────────┘
 * chỉ party_admin   ** chỉ hiện khi có >1 chợ
```

### Layout Regions

| Region ID | Name | Position | Scrollable | Key Components | Responsive Behavior |
|-----------|------|----------|------------|----------------|---------------------|
| R1 | App header | static | no | `AppHeader` | always |
| R2 | Nav dọc | static | no | `AppNav` | `w-64` |
| R3 | Tiêu đề + nút thêm | static | no | `PageHeader` | nút ẩn với `trading`/`reconcile` |
| R4 | Dải banner | static (có điều kiện) | no | banner V-02/V-03/P0017 | chỉ khi có điều kiện |
| R5 | Thanh lọc | static | no | 7 ô lọc, GET form | `flex-wrap` |
| R6 | Bộ đếm + chú giải | static | no | tổng số dòng, số `条件付き`/`再審査`, chú giải màu | always |
| R7 | Bảng người tham gia | static | ngang | bảng 10 cột, badge 5 màu | khung xương giữ đúng số cột khi tải |

## Child Routes

Không áp dụng — màn này là một route đơn, không phải khung chứa route con.

## User Flow

> **Scope:** within-screen interactions only.

### Happy Path

1. Người dùng mở `/participants`. Hệ thống tải danh sách với bộ lọc mặc định (tất cả, chợ của người đăng nhập).
2. Đọc ngưỡng cảnh báo từ `resolve_market_rule`; thất bại thì rơi về 30 ngày kèm banner `P0017`.
3. Chọn bộ lọc → ghi vào `searchParams` → tải lại phía server.
4. Bật "Sắp hết hạn" → lọc `0 ≤ days_to_expiry ≤ ngưỡng`, sắp xếp tăng dần theo `days_to_expiry`.
5. Mở một dòng → sang `SCR006_ParticipantProfile` đúng biến thể theo `basis_type`.
6. `party_admin` bấm "Thêm người tham gia" → sang `SCR006` ở chế độ tạo mới.
7. Dòng đang ở `条件付き` hoặc `再審査` có đường tắt sang `SCR036_QualificationReview` để xem phiếu xét duyệt đang mở.

### Branches

| Decision point | Condition | Outcome on this screen | Source |
|----------------|-----------|------------------------|--------|
| Bước 3 | chọn trạng thái `条件付き` hoặc `再審査` mà căn cứ đang để "tất cả" | hệ thống tự đặt căn cứ tương ứng và hiện `V-02` | thiết kế (chưa có code) |
| Bước 2 | `resolve_market_rule` không có bản hiệu lực | rơi về 30 ngày + banner `P0017` | thiết kế |

## Data Inventory

> Prefix `col.` là cột bảng. Tất cả đọc từ view đề xuất `v_participant_qualification` (bật `security_invoker = true` như ba view hiện có). Ô lọc `filter.*` ghi ở `## Validation & Error Feedback`.

| Field | Display Label | Source | Format | Empty Behavior | Cross-ref |
|-------|---------------|--------|--------|----------------|-----------|
| `col.code` | Mã người tham gia / 参加者コード | `participants.code` | readonly | — | `participants` |
| `col.name` | Tên / 名称 | `participants.name` | readonly | — | `participants` |
| `col.party_type` | Loại đối tác / 参加者区分 | `participants.party_type` | nhãn tra từ từ điển, không dựng chuỗi tại chỗ | — | `participants` |
| `col.basis` | Căn cứ tham gia / 参加資格の根拠 | `qualification_basis_rules.label_ja` / `label_vi` | nhãn JA gốc: `市場登録` `許可` `承認` `卸売場登録` | — | `qualification_basis_rules` |
| `col.basis_no` | Số hiệu / 番号 | `participant_qualifications.basis_no` | text | `—` khi rỗng | `participant_qualifications` |
| `col.valid_from` | Ngày hiệu lực / 有効日 | `participant_qualifications.valid_from` | `YYYY-MM-DD` | — | `participant_qualifications` |
| `col.valid_to` | Ngày hết hạn / 失効日 | `participant_qualifications.valid_to` | `YYYY-MM-DD`; rỗng ⇒ hiện `無期限` (chỉ xảy ra với căn cứ `市場登録` và `卸売場登録` — G9 của PREMISE, `valid_to` nullable là thay đổi đề xuất; `許可`, `承認` luôn có hạn) | `—` | `participant_qualifications` |
| `col.status` | Trạng thái / 資格状態 | `v_participant_qualification.effective_status` | huy hiệu màu; `条件付き` và `再審査` **không** được dùng chung màu | — | view |
| `col.restriction` | Điều kiện / lý do xét lại / 条件・再審査理由 | `qualification_license.condition_note` hoặc `qualification_approval.review_reason` | chỉ có giá trị khi `条件付き`/`再審査`; cắt còn 40 ký tự kèm tooltip đầy đủ | `—` | bảng riêng B/C |
| `col.days_to_expiry` | Còn lại (ngày) / 残日数 | tính trong view: `valid_to − current_date` | âm ⇒ hiện `失効`; `valid_to` NULL ⇒ rỗng, hiện `無期限`, không vào bộ lọc `filter.expiring` (G9) | — | view |

**Ghi chú về `filter.warning_days`.** `F02-07` ghi ngưỡng cảnh báo **cấu hình được**. Theo ADR-007 thứ cấu hình được và đổi theo quyết định hành chính là **dữ liệu có phiên bản**, nên ngưỡng mặc định đọc từ `market_rules`, người dùng chỉ ghi đè tạm trong phiên xem. Đây là **đề xuất** — tài liệu khách không nói ngưỡng thuộc chợ hay toàn hệ thống.

**Ghi chú về `col.status`.** `effective_status` là hợp của ba enum (xem `SCR006` `## Validation & Error Feedback`): `qualification_license_status`, `qualification_approval_status`, và `qualification_status` cũ. View chịu trách nhiệm hợp nhất để màn danh sách không phải biết căn cứ nào dùng enum nào.

## UI States

| State | Trigger | Visual Behavior | User Action Available | Source |
|-------|---------|----------------|-----------------------|--------|
| loading | lần đầu và mỗi lần đổi bộ lọc | khung xương bảng giữ đúng số cột, tránh nhảy bố cục | none | thiết kế |
| empty | bộ lọc không khớp dòng nào | "Không có người tham gia nào khớp điều kiện lọc." + nút xoá bộ lọc | Xoá bộ lọc | thiết kế |
| success | có dữ liệu | bảng + tổng số dòng + số dòng đang `条件付き`/`再審査` tách riêng | mở dòng | thiết kế |
| error (nghiệp vụ) | không áp dụng — màn chỉ đọc | — | — | — |
| error (mạng) | `select` thất bại | banner + nút thử lại. **Bộ lọc nằm trên URL (`searchParams`) nên không mất khi tải lại** | Thử lại | thiết kế |
| read-only | `trading` và `reconcile` | không hiện nút "Thêm người tham gia", dòng vẫn mở sang `SCR006` ở chế độ xem | — | thiết kế |
| saving | không áp dụng — không có ghi | — | — | — |

## Validation & Error Feedback

### A) Client-side

Màn chỉ đọc nên không có lỗi ghi. Các mục dưới là tham số lọc (prefix `filter.`).

| Field | Type | Required | Constraints | Async Check | Error Message |
|-------|------|----------|-------------|-------------|---------------|
| `filter.q` | text | no | ≤ 60 ký tự; khớp `code` hoặc `name`, không phân biệt hoa thường | không | — |
| `filter.party_type` | select | no | ∈ `wholesaler` `intermediary` `approved_trader` `market_buyer` | không | — |
| `filter.basis_type` | select | no | ∈ `registration` `license` `approval` `market_registration` | không | — |
| `filter.status` | select | no | ∈ `有効` `条件付き` `再審査` `失効` `取消`. `条件付き` chỉ có ở căn cứ `license`; `再審査` chỉ có ở `approval` | không | `V-02` |
| `filter.expiring` | select | no | ∈ rỗng / `warning` / `expired`. `warning` = `0 ≤ days_to_expiry ≤ ngưỡng`; dòng `valid_to` NULL không thuộc `warning`/`expired` (G9) | không | — |
| `filter.warning_days` | number | no | số nguyên, `1 ≤ n ≤ 180` | `resolve_market_rule(market_id, 'qualification_expiry_warning_days', current_date)` | `V-01` |
| `filter.market_id` | select | no | phải tồn tại trong `markets.id`. **Chỉ hiện khi có >1 chợ** | không | `V-03` |

#### Field Labels & Defaults *(mục bổ sung)*

| Field | Nhãn VI | Nhãn JA | Mặc định | Nguồn dữ liệu |
|---|---|---|---|---|
| `filter.q` | Mã hoặc tên | コード・名称 | rỗng | `participants.code`, `participants.name` |
| `filter.party_type` | Loại đối tác | 参加者区分 | rỗng = tất cả | `participants.party_type` |
| `filter.basis_type` | Căn cứ tham gia | 参加資格の根拠 | rỗng = tất cả | `participant_qualifications.basis_type` |
| `filter.status` | Trạng thái hiệu lực | 資格状態 | rỗng = tất cả | `v_participant_qualification.effective_status` |
| `filter.expiring` | Sắp hết hạn | 期限切れ間近 | rỗng | tính từ `days_to_expiry` |
| `filter.warning_days` | Ngưỡng cảnh báo (ngày) | 警告日数 | lấy từ quy tắc chợ (xem ghi chú ở `## Data Inventory`) | `resolve_market_rule(...)` |
| `filter.market_id` | Chợ | 市場 | chợ của người đăng nhập | `markets.id` — xem `SCR037_MarketManagement` |

#### Mã lỗi hiển thị

| Mã | Điều kiện kích hoạt | Thông báo VI | Thông báo JA | Hiện ở đâu | Chặn gửi? |
|---|---|---|---|---|:---:|
| `V-01` | `filter.warning_days` không phải số nguyên hoặc ngoài `1…180` | Ngưỡng cảnh báo phải là số nguyên từ 1 đến 180 ngày. | 警告日数は1〜180の整数で入力してください。 | dưới ô | ● |
| `V-02` | `filter.status` = `条件付き` nhưng `filter.basis_type` ≠ `license` (hoặc `再審査` nhưng ≠ `approval`) | Trạng thái「{status}」chỉ tồn tại ở căn cứ「{basis}」. Đã tự chuyển bộ lọc căn cứ. | 状態「{status}」は根拠「{basis}」のみです。根拠の絞り込みを切り替えました。 | banner nhẹ trên bảng | ✗ — tự sửa bộ lọc rồi tải lại |
| `V-03` | tham số `market_id` không tồn tại trong `markets` | Chợ không tồn tại. Đã quay về chợ mặc định. | 指定の市場がありません。既定の市場に戻しました。 | banner đầu trang | ✗ |
| `P0017` *(mới, đề xuất)* | `resolve_market_rule` không tìm được bản quy tắc hiệu lực tại `current_date` cho `qualification_expiry_warning_days` | Chưa có quy tắc hiệu lực cho ngưỡng cảnh báo. Đang dùng mặc định 30 ngày. | 有効な規程が見つかりません。既定の30日を使用します。 | banner đầu trang | ✗ — suy giảm êm, không chặn xem danh sách |

Thông báo dựng từ `error.details` (`rule_key`, `as_of`), rẽ nhánh theo `error.code` — không so khớp chuỗi `message`, theo đúng lối `lib/domain/rpc-errors.ts:46-66` (`mapRpcError`).

**Cấp phát mã lỗi mới của bốn màn LAB-4** (`P0001`–`P0009` đã dùng hết trong prototype): `P0010` vượt điều kiện `条件付き` · `P0011` đang `再審査` — cả hai do (tài liệu gốc trong repo, không nộp) `architecture.md` §2.3 ấn định, bắn ở `SCR010`/`SCR011`. Từ `P0012` trở đi là **đề xuất của tài liệu này**: `P0012` chuyển trạng thái sai máy trạng thái · `P0013` người duyệt trùng người đề nghị · `P0014` `条件付き` thiếu nội dung/hạn điều kiện · `P0015` vượt 4 lần đổi quy chế trong năm · `P0016` ngày hiệu lực không hợp lệ · `P0017` không có quy tắc hiệu lực tại `as_of` · `P0018` căn cứ không khớp loại đối tác · `P0019` thiếu chợ cho căn cứ bắt buộc gắn chợ. Các mã còn lại `P0001`–`P0009` (đã dùng — xem (tài liệu gốc trong repo, không nộp) `db-schema.md` §4) không phát sinh trực tiếp trên màn chỉ đọc này; `P0003` chỉ là nơi người dùng sẽ gặp nếu **không** tra ở màn này trước khi chốt, và `P0009` (ngày đã chốt, trigger `guard_closed_day`) / `P0001` (không tìm thấy bản ghi) không áp dụng cho `select`.

### B) Server-side

N/A — không có hành động submit/ghi. Đường đọc là `select` thẳng (xem `### Data Calls`).

## Interaction Patterns

- **Chọn trạng thái `条件付き`/`再審査` tự đặt bộ lọc căn cứ tương ứng và hiện `V-02`** — source: thiết kế (chưa có code).
- **Mở một dòng sang `SCR006` đúng biến thể theo `basis_type`; Enter trên dòng mở `SCR006`** — source: thiết kế.
- **Không có yêu cầu bàn phím riêng.** Màn dùng trong giờ hành chính, không đeo găng tay, không thuộc ba màn ⌨️ (`SCR007`, `SCR010`, `SCR014`). Mức nền vẫn giữ: mọi ô lọc và mọi dòng tới được bằng Tab, Enter trên dòng mở `SCR006`, không có affordance chỉ hiện khi hover.

## Accessibility

| Aspect | Status | Notes |
|--------|--------|-------|
| ARIA roles/labels | unknown | chưa có code; yêu cầu thiết kế: nhãn badge là chữ (không chỉ màu), `<th scope="col">` |
| Keyboard navigation | unknown | yêu cầu: Tab tới mọi ô lọc và mọi dòng, Enter mở dòng |
| Focus management | unknown | không có modal |
| Screen reader compatibility | unknown | `条件付き` vs `再審査` phải phân biệt được bằng chữ, không chỉ màu (chú giải màu ở wireframe) |

`[NO_A11Y_DETECTED]` — màn chưa có code; cần audit khi xây.

## Conditional Rendering

| Condition | Type | Renders | Hidden | Notes |
|-----------|------|---------|--------|-------|
| role = `party_admin` | auth | nút "Thêm người tham gia" | — | `trading`/`reconcile` chỉ xem |
| số chợ > 1 | feature-flag | ô lọc `filter.market_id` + cột chợ | — | một chợ thì bỏ hẳn (xem câu hỏi 4) |
| trạng thái `条件付き`/`再審査` | ui-state | cột "Điều kiện/lý do" có giá trị + đường tắt sang `SCR036` | — | nhuộm `条件付き` cùng màu `失効` dạy người dùng luật sai (xem Risks) |
| role ∉ {`party_admin`,`reconcile`,`trading`} | auth | — | cả route | `proxy.ts` → `/403` |

## Component Variants

Không áp dụng — màn không có component đổi hình dạng theo biến thể; khác biệt theo chức danh/trạng thái đã mô tả ở Conditional Rendering.

## Security Surface

| Guard | Type | Consequence if bypassed |
|-------|------|------------------------|
| `lib/auth/route-policy.ts` thêm `{ prefix: "/participants", roles: ["party_admin", "reconcile", "trading"] }` — đặt **trước** entry `"/"` (hiện bảng ở `route-policy.ts:17-27`) | role | `proxy.ts` đẩy về `/403` |
| RLS: policy `participants_select` hiện cho **mọi** role đã đăng nhập đọc (`supabase/migrations/0009_rls_policies.sql:34-35`). **Phải siết lại còn ba role** trên | permission | nếu không, `receiving` vẫn đọc được qua PostgREST dù proxy đã chặn route |
| lọc/phân trang **phía server** qua `searchParams` | permission | lọc client sẽ lộ dòng mà RLS lẽ ra không cho role đó thấy khi ma trận quyền siết lại |

Ma trận quyền: `sys_admin` ✗/✗ (quản trị kỹ thuật, không phải nghiệp vụ người tham gia) · `party_admin` *(mới, đề xuất)* ●/● · `reconcile` ●/✗ (LAB-1 giao cho cả đối chiếu) · `trading` ●/✗ (**đề xuất** — tra trước khi chốt, giảm số lần đụng `P0003` ở `SCR010`) · `receiving` ✗/✗ · `rule_admin`, `ops_admin`, `market_manager`, `internal_audit` *(đề xuất)* ✗/✗ (khớp `docs/system/permissions.md` mục D). Cờ `can_approve` không liên quan màn chỉ đọc này.

> `party_admin` là **đề xuất** (cùng 4 role đề xuất còn lại — 9 role đã chốt ở PREMISE C-09). Prototype chỉ có 4 chức danh (tài liệu gốc trong repo, không nộp: `lab3-scope-lock.md` § S1 đã khai là sai lệch có ý thức so với 7 chức danh của LAB-1). Thêm vai này là điều kiện tiên quyết của `SCR006` và `SCR036`. Chặn ở **cả hai tầng**, theo đúng lối đã dùng cho `/accounts` (`SCR003`).

## Source References

Chưa có code — màn chỉ đặc tả ở LAB-4. Điểm chạm trong code hiện có (đã kiểm tồn tại):

1. Bảng route cần thêm entry: `lib/auth/route-policy.ts:17-27`
2. RLS cần siết: `supabase/migrations/0009_rls_policies.sql:34-37`
3. Mẫu đọc danh sách người tham gia hiện có: `lib/domain/master-data.ts:119-131` (`listParticipants`)
4. Mẫu ánh xạ lỗi RPC cần dùng lại: `lib/domain/rpc-errors.ts:14-66`
5. Mẫu bộ lọc GET + bảng: `app/(app)/accounts/page.tsx:21-104` (`SCR003`)

## Source Walkthrough

Chưa có code — màn chỉ đặc tả ở LAB-4. Thứ tự đọc để hiểu bối cảnh khi xây:

1. **File:** `supabase/migrations/0003_participants_and_master.sql:12-52` — bảng `qualification_basis_rules`, `participant_qualifications`: dữ liệu nền.
2. **File:** `lib/auth/route-policy.ts:17-37` — cách thêm route theo role.
3. **File:** `app/(app)/accounts/page.tsx:21-104` — mẫu danh sách + lọc đã chạy.

### Call Hierarchy

```text
(đề xuất) ParticipantListPage -> v_participant_qualification (select, searchParams) ; -> rpc resolve_market_rule (ngưỡng)
```

### Read Calls *(mục bổ sung)*

| Thao tác | Cách gọi | Tham số | Trả về |
|---|---|---|---|
| Đọc danh sách | `select` từ `v_participant_qualification` | `q`, `party_type`, `basis_type`, `status`, `expiring`, `warning_days`, `market_id` | mảng dòng, kèm `count` |
| Đọc ngưỡng cảnh báo | `rpc('resolve_market_rule', { p_market_id, p_rule_key: 'qualification_expiry_warning_days', p_as_of: current_date })` | | `jsonb` hoặc raise `P0017` |
| Đọc nhãn căn cứ | `select` từ `qualification_basis_rules` | — | 4 dòng, cache theo phiên |

Đi `select` thẳng chứ không qua RPC vì đây là đường **đọc** thuần, không có bất biến nào phải giữ trong một transaction — đúng ranh giới ADR-002 (RPC dành cho đường ghi). View bật `security_invoker = true` như ba view hiện có, để RLS của bảng gốc vẫn áp dụng. Lọc và phân trang làm **phía server** qua `searchParams`.

### Dependencies & Risks *(mục bổ sung)*

- **Cần trước:** `SCR006` (nguồn dữ liệu), `SCR036` (sinh ra trạng thái `条件付き`/`再審査`), `SCR037` (nguồn ngưỡng cảnh báo và chợ). Không có `SCR036` thì cột `col.status` và `col.restriction` chỉ bao giờ hiện `有効`/`失効`/`取消`.
- **Dữ liệu phải có sẵn:** 4 dòng `qualification_basis_rules` (đã có từ `0003`), ít nhất 1 dòng `markets`.
- **Dễ hiểu sai #1:** một người tham gia về nguyên tắc có **nhiều** dòng `participant_qualifications` (gia hạn tạo dòng mới, không ghi đè). View phải chọn dòng hiệu lực tại `as_of`, không phải dòng mới nhất theo `created_at`. Chọn sai thì danh sách hiện "有効" cho người đã hết hạn.
- **Dễ hiểu sai #2:** `条件付き` **vẫn giao dịch được**. Nhuộm nó cùng màu với `失効` sẽ dạy người dùng một luật sai, rồi họ từ chối bán cho đối tác hợp lệ.
- **Rủi ro hiệu năng:** `days_to_expiry` tính trong view nên không index được trực tiếp. Lọc "sắp hết hạn" nên viết thành điều kiện trên `valid_to` (`valid_to between current_date and current_date + n`) để còn dùng được index.
- **Rủi ro di trú:** siết `participants_select` từ "mọi role" xuống ba role là **thay đổi phá tương thích** với `SCR010` nếu màn đó đang đọc `participants` để hiện tên người mua (`lib/domain/master-data.ts:62-73` — `findParticipantByCode` do `SCR007`/`SCR010`/`SCR011` gọi). Phải kiểm trước khi đổi policy.
- Link chéo: [F002](../../features/F002_ParticipantsAndQualification/screens.md); flow FL-03 (vòng đời tư cách tham gia); ADR-006 (hai máy trạng thái 許可 vs 承認), ADR-007 (quy tắc theo chợ).

### Unresolved Questions *(mục bổ sung)*

1. **Ngưỡng cảnh báo hết hạn thuộc chợ hay toàn hệ thống?** `F02-07` chỉ nói "cấu hình được". Nếu toàn hệ thống thì bỏ `market_id` khỏi lời gọi `resolve_market_rule`.
2. **`条件付き` cụ thể hạn chế cái gì?** Chưa biết thì cột `col.restriction` chỉ hiện được chữ tự do, không hiện được con số trần. Câu hỏi gốc ở ADR-006.
3. **Khách xác nhận cách duyệt `再審査`?** Hướng đã chốt giả định G8 (PREMISE): `party_admin` có cờ `can_approve`, khác người lập — nên không thêm vai nào cần xem `SCR005`. Nếu khách chọn chức danh duyệt riêng thì vai đó phải xem được màn này.
4. **Hệ thống phục vụ một chợ hay nhiều chợ?** Một chợ thì bỏ hẳn ô lọc `filter.market_id` và cột chợ. Câu hỏi gốc ở ADR-007.
5. **Người tham gia có bị xoá bao giờ không, hay chỉ `取消`?** Prototype không có cột `deleted_at`. Đề xuất: không xoá, chỉ `取消` — cần khách xác nhận vì ảnh hưởng tới báo cáo kỳ cũ.
6. **Một người tham gia có thể giữ đồng thời hai căn cứ không** (ví dụ vừa `許可` vừa `承認`)? Schema hiện không cấm. Ma trận §7.2 của khách ngụ ý một nhóm đối tác một căn cứ, nhưng không nói thẳng.
