# SCR006_ParticipantProfile — Screen Spec

> Alias LAB-1: **SC-06** · 参加者台帳 *(tên JA đề xuất — bảng màn hình LAB-1 không có cột tên JA)* · Hồ sơ người tham gia

**Screen**: SCR006_ParticipantProfile: Hồ sơ người tham gia
**Feature**: F002_ParticipantsAndQualification — [screens.md](../../features/F002_ParticipantsAndQualification/screens.md)
**Type**: atomic *(form chi tiết, **bốn biến thể** theo căn cứ tham gia — xem `## Component Variants`)*
**Route**: `/participants/[id]` · tạo mới `/participants/new?party_type=…` *(đề xuất)*
**Generated**: 2026-10-06
**Trạng thái**: Thiết kế mới, chưa xây
**Tính năng LAB-1**: `F02-01` `F02-02` `F02-03` `F02-04` `F02-05`
**User story**: không có US riêng. Là nguồn dữ liệu cho `US-04` (`F02-06` kiểm tư cách lúc chốt).
**Wireframe**: [sc-06-ho-so-nguoi-tham-gia.html](../../design/wireframes/sc-06-ho-so-nguoi-tham-gia.html)

## Purpose

Lập và sửa hồ sơ của một người tham gia, kèm **căn cứ tham gia** của họ. Đây là nơi duy nhất sinh ra dữ liệu mà `_assert_buyer_qualified` đọc lúc chốt giao dịch (`supabase/migrations/0007_rpc_functions.sql:12`) — sai ở đây thì `SCR010` chặn nhầm hoặc cho qua nhầm.

Người dùng: quản lý người tham gia, giờ hành chính, không thuộc nhóm màn hiện trường.

**Màn này có bốn biến thể, không phải một form với một ô chọn.** LAB-1 ghi thẳng: *"許可 ≠ 承認 — hai căn cứ khác bản chất, tuyệt đối không gộp thành một rule chung. Đây là điều kiện trực tiếp bị loại (RFP 12-06)"*, và *"là hai mô hình dữ liệu riêng, hai luồng trạng thái riêng"*. Prototype LAB-3 đang gộp — đó là sai lệch **D-01**, hạng *phải sửa* (ADR-006).

## Screen Layout

Khung `AppLayout`; vùng nội dung gồm `PageHeader`, khối trạng thái tư cách (`status`, và nếu `条件付き`/`再審査` thì nội dung điều kiện/lý do xét lại **chỉ đọc** kèm nút "Đề nghị chuyển trạng thái"), **khối chung** (#1–#10) đặt một lần ở trên, rồi **một trong bốn khối riêng** theo biến thể (wireframe vẽ bốn thẻ tab thuần CSS để so sánh; app thật dựng đúng một khối theo `party_type`), và khối lịch sử chuyển trạng thái. Wireframe tự quyết đẩy `basis_no`/`valid_from`/`valid_to` xuống khối riêng để người xem thấy đúng ràng buộc từng biến thể (xem (tài liệu gốc trong repo, không nộp) `README.md` § "Chỗ spec chưa đủ"). *Bố cục là đề xuất, chưa có ảnh deploy.*

### Layout Sketch

```
┌────────────────────────────────────────────────────────────┐
│ R1: AppHeader                                              │
├────────────┬───────────────────────────────────────────────┤
│ R2: AppNav │ R3: PageHeader "Hồ sơ người tham gia"         │
│            │ R4: Trạng thái tư cách [Đề nghị chuyển → SCR036]│
│            │ R5: Khối chung  #1..#10 (code,name,party_type, │
│            │      basis_type↔, contact, market_id, ...)     │
│            │ R6: Khối RIÊNG biến thể  A | B | C | D         │
│            │ R7: Lịch sử chuyển trạng thái (qualification_  │
│            │      transitions) + banner lỗi/đúng - - - -    │
└────────────┴───────────────────────────────────────────────┘
```

### Layout Regions

| Region ID | Name | Position | Scrollable | Key Components | Responsive Behavior |
|-----------|------|----------|------------|----------------|---------------------|
| R1 | App header | static | no | `AppHeader` | always |
| R2 | Nav dọc | static | no | `AppNav` | `w-64` |
| R3 | Tiêu đề | static | no | `PageHeader` | always |
| R4 | Khối trạng thái tư cách | static | no | badge `status`, nút "Đề nghị chuyển trạng thái" (chỉ `party_admin`) | always |
| R5 | Khối chung | static | no | form field #1–#10 | `max-w` cột đơn |
| R6 | Khối riêng biến thể | static | no | 1 trong 4 khối (A/B/C/D) | chỉ dựng đúng 1 khối |
| R7 | Lịch sử + banner | static | no | bảng `qualification_transitions`, banner đầu form | always |

## Child Routes

Không áp dụng — màn này là một route đơn, không phải khung chứa route con.

## User Flow

> **Scope:** within-screen interactions only.

### Happy Path

1. Từ `SCR005` chọn "Thêm người tham gia" → chọn **loại đối tác** trước khi vào form.
2. Hệ thống suy ra `basis_type` và **dựng đúng một trong bốn biến thể**. Không có bước "chọn căn cứ".
3. Nhập khối chung (#1–#9), rồi khối riêng của biến thể.
4. Lưu → `rpc('upsert_participant_profile', …)`. Thành công → ở lại màn, banner xác nhận, trạng thái tư cách hiện `有効`.
5. Mở hồ sơ có sẵn: khối trạng thái tư cách hiện `status`; nếu `条件付き` hoặc `再審査` thì hiện nội dung điều kiện / lý do xét lại ở chế độ **chỉ đọc**, kèm đường sang `SCR036_QualificationReview`.
6. Muốn đổi trạng thái → "Đề nghị chuyển trạng thái" → sang `SCR036`. **Không có đường đổi trạng thái tại màn này.**
7. Gia hạn: tạo **dòng tư cách mới** với `valid_from` kế tiếp, không sửa đè dòng cũ — để báo cáo kỳ cũ vẫn tra được tư cách tại thời điểm đó (`FR-INC-03`, nguyên tắc append-only §3.1).

### Branches

| Decision point | Condition | Outcome on this screen | Source |
|----------------|-----------|------------------------|--------|
| Bước 3 | biến thể B | không hiện ô chợ (`market_id` không áp dụng) | thiết kế |
| Bước 3 | biến thể C, D | ô chợ bắt buộc; D còn tải danh sách điểm giao dịch và regex mã từ quy chế chợ đang hiệu lực | thiết kế |
| Bước 5 | trạng thái `取消` hoặc role `trading`/`reconcile` | mọi ô readonly, không nút lưu | thiết kế |

## Data Inventory

> Chỉ các field chỉ đọc/dẫn xuất. Field nhập liệu ở `## Validation & Error Feedback`.

| Field | Display Label | Source | Format | Empty Behavior | Cross-ref |
|-------|---------------|--------|--------|----------------|-----------|
| `basis_type` (#4) | Căn cứ tham gia / 参加資格の根拠 | **suy ra 1:1 từ `party_type` (#3)**, người dùng không chọn — bảng ánh xạ bên dưới | readonly | — | `participant_qualifications.basis_type` |
| `status` (#10) | Trạng thái tư cách / 資格状態 | enum theo biến thể; **chỉ đổi qua `SCR036`** | badge; `有効` khi tạo | — | enum theo biến thể |
| `condition_note` (B5) | Nội dung điều kiện / 条件の内容 | `qualification_license.condition_note` — chỉ khi `条件付き`; **đặt ở `SCR036`** | readonly | `—` | `qualification_license` |
| `condition_until` (B6) | Hạn điều kiện / 条件期限 | `qualification_license.condition_until` — chỉ khi `条件付き` | readonly | `—` | `qualification_license` |
| `condition_max_qty_per_day` (B7) | Trần số lượng/ngày / 1日あたり数量上限 | `qualification_license.condition_max_qty_per_day`; `> 0` khi có; đơn vị theo mặt hàng. Vượt trần ở `SCR010` → **`P0010`** | readonly | `—` | `qualification_license` |
| `condition_negotiated_only` (B8) | Chỉ cho 相対, cấm せり / 相対のみ・せり不可 | `qualification_license.condition_negotiated_only`; boolean; `true` ⇒ `SCR011` từ chối với `P0010` | readonly | `false` | `qualification_license` |
| `approver_office` (C2) | Bên chấp thuận / 承認者 | `markets.operator_name` — **luôn là chính chợ**, không cho sửa | readonly | theo `market_id` | `markets` |
| `review_reason` (C6) | Lý do xét lại / 再審査理由 | `qualification_approval.review_reason` — chỉ khi `再審査`; **đặt ở `SCR036`** | readonly | `—` | `qualification_approval` |
| `review_due_on` (C7) | Hạn xét lại / 再審査期限 | `qualification_approval.review_due_on` — chỉ khi `再審査` | readonly | `—` | `qualification_approval` |
| `issuer` (D3) | Bên cấp / 発行者 | `markets.operator_name` (Ban quản lý chợ) | readonly | — | `markets` |
| `rule_version` (D4) | Phiên bản quy chế áp dụng / 適用規程版 | `resolve_market_rule(market_id,'buyer_registration',current_date)` — hiện `version` + `発効日` | readonly | — | `market_rules` |

**Ánh xạ `party_type` (#3) → `basis_type` (#4)** (từ ma trận 参加資格 của khách, (tài liệu gốc trong repo, không nộp) `product-a-rfp-business-summary.md` §7.2):

| `party_type` | Nhóm đối tác | `basis_type` | Nhãn JA | Khi vi phạm |
|---|---|---|---|---|
| `wholesaler` | 卸売業者 | `registration` | `市場登録` | thu hồi chấp thuận → `取消` |
| `intermediary` | 仲卸 | `license` | **`許可`** | **`条件付き`** — hạn chế có điều kiện |
| `approved_trader` | 売買参加者 | `approval` | **`承認`** | **`再審査`** — phải xét lại |
| `market_buyer` | 買出人 | `market_registration` | `卸売場登録` | theo quy chế chợ |

Vì ánh xạ là 1:1 nên `basis_type` là **field dẫn xuất, không phải ô nhập**. Phép thử của (tài liệu gốc trong repo, không nộp) `architecture.md` §3.3 — *"xoá `basis_type` đi mà logic vẫn chạy y nguyên thì vẫn đang gộp"* — được thoả vì ba khối riêng có **cột khác nhau ở bảng khác nhau**, không phải cùng một bảng rẽ bằng `CASE WHEN`.

Enum riêng theo biến thể: A và D dùng `qualification_status` hiện có (`有効` `失効` `取消`, **không** có `条件付き`/`再審査`); B dùng `qualification_license_status` (`有効` `条件付き` `失効` `取消`); C dùng `qualification_approval_status` (`有効` `再審査` `失効` `取消`).

## UI States

| State | Trigger | Visual Behavior | User Action Available | Source |
|-------|---------|----------------|-----------------------|--------|
| empty | chế độ tạo mới | form trống, `basis_type` hiện ngay khi chọn `party_type` | nhập | thiết kế |
| loading | mở hồ sơ có sẵn | khung xương theo đúng biến thể đã biết từ `party_type` trên URL, tránh nhảy bố cục khi đổi biến thể | none | thiết kế |
| success | sau khi ghi | banner "Đã lưu hồ sơ {code}"; focus về đầu form; khối trạng thái tư cách tải lại | tiếp tục | thiết kế |
| error (nghiệp vụ) | RPC từ chối | banner đầu form **tự nhận focus**; dữ liệu đã gõ giữ nguyên | sửa & gửi lại | thiết kế |
| error (mạng) | mất kết nối | banner + nút thử lại; **dữ liệu đã gõ giữ nguyên trong state client, không reset** | Thử lại | thiết kế |
| read-only | `trading`, `reconcile`, hoặc trạng thái `取消` | mọi ô readonly, không nút lưu; với `取消` hiện thêm ngày và lý do thu hồi | — | thiết kế |
| saving | đang gọi RPC | nút lưu disabled | none | thiết kế |

## Validation & Error Feedback

### A) Client-side

| Field | Type | Required | Constraints | Async Check | Error Message |
|-------|------|----------|-------------|-------------|---------------|
| `code` (#1) | text | yes | `^[A-Z0-9-]{3,20}$`; duy nhất trong `participants.code`; **khoá sau khi tạo** | không | `V-01`/`V-02`/`P0069` |
| `name` (#2) | text | yes | 1–120 ký tự | không | `V-01` |
| `party_type` (#3) | select | yes | ∈ `wholesaler` `intermediary` `approved_trader` `market_buyer`; **khoá sau khi tạo** | không | `V-01` |
| `contact` (#5) | text | no | ≤ 200 ký tự | không | — |
| `market_id` (#6) | select | tuỳ biến thể | tồn tại trong `markets.id`. **Bắt buộc** với C và D, **không áp dụng** với B, **tuỳ chọn** với A | không | `V-05`/`P0019` |
| `basis_no` (#7) | text | yes | 1–40 ký tự; duy nhất theo cặp `(basis_type, basis_no)` | không | `V-01` |
| `valid_from` (#8) | date | yes | `YYYY-MM-DD`; `≤ valid_to` khi có `valid_to` | không | `V-03` |
| `valid_to` (#9) | date | tuỳ biến thể | `≥ valid_from`. **Bắt buộc** với B (`許可`) và C (`承認`); **không bắt buộc** với A (`市場登録`) và D (`卸売場登録`) — để trống = không thời hạn (G9 của PREMISE; LAB-1 F02-01, F02-04 không ghi ngày hết hạn). Cột `valid_to` hiện `NOT NULL`: cho phép NULL là thay đổi ràng buộc **đề xuất** (migration ở LAB-5) | không | `V-03`/`V-04` |
| `issuer` (A1) | text | yes (A) | 1–120 ký tự; **là 開設者 của chợ**, không phải cơ quan hành chính ngoài | không | `V-01` |
| `registered_on` (A2) | date | yes (A) | `≤ current_date` | không | `V-07` |
| `license_no` (B1) | text | yes (B) | 1–40 ký tự; duy nhất | không | `V-01` |
| `issuing_authority` (B2) | text | yes (B) | 1–120 ký tự. **Cơ quan hành chính NGOÀI chợ** — chợ không cấp và không thu hồi được | không | `V-01` |
| `licensed_on` (B3) | date | yes (B) | `≤ current_date` | không | `V-07` |
| `license_expires_on` (B4) | date | yes (B) | `> licensed_on` | không | `V-04` |
| `approval_no` (C1) | text | yes (C) | 1–40 ký tự; duy nhất | không | `V-01` |
| `approved_on` (C3) | date | yes (C) | `≤ current_date` | không | `V-07` |
| `approval_expires_on` (C4) | date | yes (C) | `> approved_on` | không | `V-04` |
| `participation_limit` (C5) | number | no (C) | `> 0`; JPY, số nguyên | không | `V-06` |
| `site_code` (D1) | text | yes (D) | theo regex đọc từ quy tắc chợ, **không hằng số trong code** | `resolve_market_rule(..., 'buyer_registration', ...)` | `V-08` |
| `site_name` (D2) | select | yes (D) | phải thuộc danh sách điểm của `market_id` | quy tắc chợ `buyer_registration.sites[]` | `V-01` |

#### Field Labels & Defaults *(mục bổ sung)*

| # | Field | Nhãn VI | Nhãn JA | Mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|
| 1 | `code` | Mã người tham gia | 参加者コード | sinh gợi ý theo tiền tố loại đối tác | `participants.code` |
| 2 | `name` | Tên | 名称 | | `participants.name` |
| 3 | `party_type` | Loại đối tác | 参加者区分 | theo tham số URL | `participants.party_type` |
| 4 | `basis_type` | Căn cứ tham gia | 参加資格の根拠 | suy ra từ #3 | `participant_qualifications.basis_type` |
| 5 | `contact` | Liên hệ | 連絡先 | | `participants.contact` |
| 6 | `market_id` | Chợ | 市場 | chợ của người đăng nhập | `markets.id` — xem `SCR037` |
| 7 | `basis_no` | Số hiệu căn cứ | 番号 | | `participant_qualifications.basis_no` |
| 8 | `valid_from` | Ngày hiệu lực | 有効日 | hôm nay | `participant_qualifications.valid_from` |
| 9 | `valid_to` | Ngày hết hạn | 失効日 | | `participant_qualifications.valid_to` |
| 10 | `status` | Trạng thái tư cách | 資格状態 | `有効` khi tạo | enum theo biến thể |
| A1 | `issuer` | Cơ quan đăng ký | 登録機関 | điền sẵn từ `markets.operator_name` | `participant_qualifications.issuer` |
| A2 | `registered_on` | Ngày đăng ký | 登録日 | | `participant_qualifications.valid_from` (dùng chung #8) |
| B1 | `license_no` | Số giấy phép | 許可番号 | | `qualification_license.license_no` (ánh xạ về #7) |
| B2 | `issuing_authority` | Cơ quan cấp phép | 許可行政庁 | | `qualification_license.issuing_authority` |
| B3 | `licensed_on` | Ngày cấp phép | 許可日 | | `qualification_license.licensed_on` |
| B4 | `license_expires_on` | Hạn giấy phép | 有効期限 | | `qualification_license.expires_on` (ánh xạ về #9) |
| C1 | `approval_no` | Số chấp thuận | 承認番号 | | `qualification_approval.approval_no` (ánh xạ về #7) |
| C3 | `approved_on` | Ngày chấp thuận | 承認日 | | `qualification_approval.approved_on` |
| C4 | `approval_expires_on` | Hạn chấp thuận | 有効期限 | | `qualification_approval.expires_on` (ánh xạ về #9) |
| C5 | `participation_limit` | Hạn mức tham gia | 参加限度額 | | `qualification_approval.participation_limit` |
| D1 | `site_code` | Mã đăng ký tại chợ | 卸売場登録コード | | `participant_qualifications.basis_no` |
| D2 | `site_name` | Điểm giao dịch | 登録場所 | | quy tắc chợ `buyer_registration.sites[]` |

#### Mã lỗi hiển thị

| Mã | Điều kiện kích hoạt | Thông báo VI | Thông báo JA | Hiện ở đâu | Chặn gửi? |
|---|---|---|---|---|:---:|
| `V-01` | ô bắt buộc để trống | Mục này bắt buộc. | 必須項目です。 | dưới ô | ● |
| `V-02` | `code` sai regex | Mã chỉ gồm chữ in hoa, số và dấu gạch ngang, 3–20 ký tự. | コードは英大文字・数字・ハイフン3〜20文字です。 | dưới ô | ● |
| `V-03` | `valid_to < valid_from` | Ngày hết hạn phải từ ngày hiệu lực trở đi. | 失効日は有効日以降にしてください。 | dưới ô `valid_to` | ● |
| `V-04` | biến thể B hoặc C mà `valid_to` trống | Căn cứ「{basis}」bắt buộc có ngày hết hạn. | 根拠「{basis}」は有効期限が必須です。 | dưới ô `valid_to` | ● |
| `V-05` | biến thể C hoặc D mà `market_id` trống | Căn cứ này phải gắn với một chợ. | この根拠は市場の指定が必須です。 | dưới ô `market_id` | ● |
| `V-06` | `participation_limit ≤ 0` hoặc không nguyên | Hạn mức tham gia phải là số nguyên dương (JPY). | 参加限度額は正の整数（円）で入力してください。 | dưới ô | ● |
| `V-07` | `licensed_on > current_date` / `approved_on > current_date` | Ngày cấp không được ở tương lai. | 交付日に未来日は指定できません。 | dưới ô | ● |
| `V-08` | biến thể D, `site_code` sai regex của quy chế đang hiệu lực | Mã đăng ký không đúng quy chế「{version}」của chợ. | 登録コードが市場規程「{version}」に適合しません。 | dưới ô | ● |
| `P0069` *(mới, đề xuất; dùng chung với SCR037)* | mã người tham gia đã tồn tại (vi phạm UNIQUE, mã đã có trong prototype) (cấp lại khi nộp lại 2026-10-06: trước đây trùng P0005) | Mã「{code}」đã được dùng. | コード「{code}」は既に使用されています。 | banner đầu form | ● |
| `P0018` *(mới, đề xuất)* | `basis_type` gửi lên không khớp ánh xạ 1:1 với `party_type` | Căn cứ「{basis}」không áp dụng cho loại đối tác「{party_type}」. | 根拠「{basis}」は区分「{party_type}」には適用できません。 | banner đầu form | ● |
| `P0019` *(mới, đề xuất)* | thiếu `market_id` với căn cứ `approval` hoặc `market_registration` | Căn cứ「{basis}」phải gắn với một chợ. | 根拠「{basis}」は市場の指定が必須です。 | banner đầu form | ● |
| `P0017` *(mới, đề xuất)* | biến thể D, không có bản `buyer_registration` hiệu lực tại `current_date` | Chợ chưa có quy chế đăng ký người mua đang hiệu lực. Liên hệ quản trị quy chế. | 有効な買出人登録規程がありません。規程管理者に連絡してください。 | banner đầu form | ● |

Lỗi server rẽ nhánh theo `error.code`, thông báo dựng từ `error.details` (`code`, `basis`, `party_type`, `rule_key`, `version`) qua `mapRpcError` (`lib/domain/rpc-errors.ts:46-66`) — không so khớp chuỗi `message`.

> Lưu ý đối chiếu mã: spec LAB-4 cũ dùng `P0005` cho "mã đã tồn tại", trong khi prototype đã cài `P0005` = *sai chức danh* (`supabase/migrations/0007_rpc_functions.sql:54`). Lượt nộp lại 2026-10-06 cấp lại "mã đã tồn tại" thành `P0069` (đề xuất, chưa cài); `P0005` chỉ còn nghĩa *sai chức danh*.

### B) Server-side

#### Ghi hồ sơ người tham gia
- **Endpoint:** `rpc('upsert_participant_profile', { p_participant, p_qualification, p_variant_payload })` *(RPC mới, đề xuất)*
- **Request:** `jsonb` theo biến thể (khối chung #1–#9 + khối riêng)
- **Success:** `jsonb { participant_id, qualification_id }` → ở lại màn, banner xác nhận, trạng thái `有効`
- **Errors:** `P0069` mã đã dùng | `P0018` căn cứ không khớp loại đối tác | `P0019` thiếu chợ | `P0017` thiếu quy chế (biến thể D)
- **Trigger:** nút lưu (form submit; Enter trong ô text **không** submit)
- **Source:** thiết kế — chưa có code

**Bắt buộc đi qua RPC, không `insert` thẳng.** Ba lý do: (a) ADR-002 — hai bảng này sẽ không có policy ghi, nên PostgREST không ghi được; (b) một lần lưu chạm **hai bảng** (`participants` + bảng riêng của biến thể), phải cùng một transaction; (c) ánh xạ `party_type → basis_type` là bất biến nghiệp vụ, phải giữ ở tầng thấp nhất chứ không phải ở client — bài học ADR-004.

## Interaction Patterns

- **Chọn `party_type` trước, form dựng đúng một biến thể; không có bước "chọn căn cứ"** — source: thiết kế.
- **Enter trong ô text không submit** (form dài, submit nhầm là mất công); nút lưu tới được bằng bàn phím; banner lỗi tự nhận focus — source: thiết kế.
- **Không có yêu cầu bàn phím riêng** — không thuộc ba màn ⌨️ (`SCR007`, `SCR010`, `SCR014`). Mức nền: Tab đi qua đủ các ô theo thứ tự khai báo.

## Accessibility

| Aspect | Status | Notes |
|--------|--------|-------|
| ARIA roles/labels | unknown | chưa có code; yêu cầu: dùng `Field` (`components/ui/field.tsx:23-57`) gắn `aria-describedby`/`aria-invalid` |
| Keyboard navigation | unknown | yêu cầu: Tab theo thứ tự khai báo |
| Focus management | unknown | banner lỗi tự nhận focus (yêu cầu thiết kế) |
| Screen reader compatibility | unknown | — |

`[NO_A11Y_DETECTED]` — màn chưa có code; cần audit khi xây.

## Conditional Rendering

| Condition | Type | Renders | Hidden | Notes |
|-----------|------|---------|--------|-------|
| `party_type` → `basis_type` | ui-state | đúng một khối riêng A/B/C/D | ba khối còn lại | không có dropdown chọn căn cứ |
| biến thể B | ui-state | — | ô `market_id` | giấy phép do cơ quan ngoài cấp, hiệu lực liên chợ (`architecture.md` §2.3) |
| `status ∈ {条件付き, 再審査}` | ui-state | nội dung điều kiện/lý do xét lại (chỉ đọc) + link `SCR036` | — | không đổi được ở màn này |
| role `party_admin` | auth | form ghi + nút lưu | — | `trading`/`reconcile`: mọi ô readonly |
| trạng thái `取消` | ui-state | ngày và lý do thu hồi; không nút lưu | — | |

## Component Variants

| Component | Discriminating field | Variants on this screen | Screen-specific props/slots | Cross-ref |
|-----------|---------------------|------------------------|----------------------------|-----------|
| Form hồ sơ người tham gia | `party_type` ⇒ `basis_type` | A `市場登録` (卸売業者) · B `許可` (仲卸) · C `承認` (売買参加者) · D `卸売場登録` (買出人) | khối riêng theo biến thể | ADR-006; `DISC-` chưa cấp |

**Vì sao bốn biến thể không dùng chung một form:**

| | A `市場登録` | B `許可` | C `承認` | D `卸売場登録` |
|---|---|---|---|---|
| Bên cấp | chợ (開設者) | **cơ quan hành chính ngoài chợ** | **chính chợ** | ban quản lý chợ |
| Hạn hiệu lực | tuỳ chọn, trống = không thời hạn (G9) | **bắt buộc** | **bắt buộc** | tuỳ chọn, trống = không thời hạn (G9) |
| Vi phạm → | `取消` | **`条件付き`, vẫn giao dịch** | **`再審査`, dừng giao dịch** | theo quy chế chợ |
| Gỡ bằng | — | người duyệt nội bộ gỡ điều kiện | xét lại hồ sơ, `承認` lại | theo quy chế chợ |
| Gắn chợ | tuỳ chọn | **không** | **bắt buộc** | **bắt buộc** |
| Enum trạng thái | `qualification_status` | `qualification_license_status` | `qualification_approval_status` | `qualification_status` |
| Mã lỗi khi chặn ở `SCR010` | `P0003` | **`P0010`** | **`P0011`** | `P0003` |

Bốn cột này khác nhau ở **cấu trúc dữ liệu**, không ở giá trị. Một form dùng chung buộc phải để `条件付き` và `再審査` cùng một cột trạng thái — chính là thứ RFP 12-06 xếp vào điều kiện bị loại, và chính là thứ prototype đang làm (D-01). Với D: không có luồng vi phạm cứng — mọi điều kiện (thời hạn, gia hạn, thu hồi) đọc từ `market_rules`, vì `violation_action` của căn cứ này là *"Theo quy chế chợ"*.

## Security Surface

| Guard | Type | Consequence if bypassed |
|-------|------|------------------------|
| `route-policy.ts` prefix `/participants` — `party_admin` ●/●; `reconcile`, `trading` chỉ xem; `sys_admin`, `receiving` ✗ | role | `/403` |
| RLS đọc: siết `participants_select` (hiện mọi role đã đăng nhập, `0009_rls_policies.sql:34-37`) | permission | `receiving` đọc được qua PostgREST |
| **Đường ghi tầng 3:** `participants` / `participant_qualifications` **không mở policy INSERT/UPDATE**, đường ghi duy nhất là RPC `SECURITY DEFINER` (ADR-002) — vì ai cũng có publishable key và gọi thẳng PostgREST được | permission | ghi trực tiếp vượt ánh xạ 1:1 |
| Ba trường **không ghi được từ màn này**, kể cả bởi `party_admin`: trạng thái tư cách, nội dung điều kiện `条件付き`, lý do `再審査` — chỉ đổi qua `SCR036` với người duyệt khác người đề nghị | role | mất maker-checker |

> `party_admin` là **đề xuất** (chưa có trong enum `app_role`; xem `SCR005` `## Security Surface`).

## Source References

Chưa có code — màn chỉ đặc tả ở LAB-4. Điểm chạm trong code hiện có (đã kiểm tồn tại):

1. Hàm kiểm tư cách đọc dữ liệu từ màn này: `supabase/migrations/0007_rpc_functions.sql:12-38` (`_assert_buyer_qualified`, `P0003`)
2. Bảng nền: `supabase/migrations/0003_participants_and_master.sql:12-52`
3. RLS: `supabase/migrations/0009_rls_policies.sql:34-37`
4. Mẫu `Field`/form: `components/ui/field.tsx:23-57`; mẫu chống mất dữ liệu khi mất mạng: `app/(app)/lots/new/use-lot-entry-form.ts:53-129`
5. Ánh xạ lỗi RPC: `lib/domain/rpc-errors.ts:46-66`

## Source Walkthrough

Chưa có code — màn chỉ đặc tả ở LAB-4. Thứ tự đọc để hiểu bối cảnh khi xây:

1. **File:** `supabase/migrations/0003_participants_and_master.sql:12-52` — dữ liệu nền tư cách.
2. **File:** `supabase/migrations/0007_rpc_functions.sql:12-38` — nơi dữ liệu màn này được tiêu thụ.
3. **File:** `app/(app)/lots/new/use-lot-entry-form.ts:53-129` — mẫu form giữ dữ liệu khi lỗi mạng.

### Call Hierarchy

```text
(đề xuất) ParticipantProfilePage -> v_participant_qualification + bảng riêng (select) -> form biến thể -> rpc upsert_participant_profile
```

### Read Calls *(mục bổ sung)*

| Thao tác | Cách gọi | Tham số | Trả về |
|---|---|---|---|
| Đọc hồ sơ | `select` từ `v_participant_qualification` + bảng riêng theo `basis_type` | `participant_id` | 1 dòng chung + 0–1 dòng riêng |
| Đọc quy chế biến thể D | `rpc('resolve_market_rule', { p_market_id, p_rule_key: 'buyer_registration', p_as_of })` | | `jsonb` (regex, `sites[]`, thời hạn) hoặc raise `P0017` |
| Ghi hồ sơ | `rpc('upsert_participant_profile', {...})` | `jsonb` theo biến thể | `jsonb { participant_id, qualification_id }` hoặc raise `P0069`/`P0018`/`P0019` |
| Đọc lịch sử tư cách | `select` từ `qualification_transitions` | `participant_id` | mảng, mới nhất trước |

`p_as_of` cho `resolve_market_rule` là **ngày nghiệp vụ**, không phải `now()` — xem `architecture.md` C6.

### Dependencies & Risks *(mục bổ sung)*

- **Cần trước:** `SCR037_MarketManagement` (bảng `markets`, quy chế `buyer_registration`). Không có thì biến thể C và D không lưu được. **Cần sau:** `SCR036` là nơi duy nhất đổi trạng thái; `SCR005` là đường vào.
- **Dữ liệu phải có sẵn:** 4 dòng `qualification_basis_rules`, ≥1 dòng `markets`, ≥1 bản `market_rules` cho `buyer_registration`.
- **Di trú — chỗ khó nhất (ADR-006):** dữ liệu `participant_qualifications` hiện có đang ở `active`/`expired`/`revoked` và **không suy ra được** trạng thái mới, vì `expired` do hết hạn khác hẳn `取消` do vi phạm, mà cột hiện tại không phân biệt. Đề xuất: giữ nguyên `qualification_status` cho A và D; với B và C, backfill `active → 有効`, `expired → 失効`, `revoked → 取消`, và **không** tự sinh `条件付き`/`再審査` cho dòng nào — hai trạng thái đó chỉ phát sinh từ `SCR036` về sau.
- **Dễ hiểu sai #1:** gia hạn **không** phải sửa `valid_to` của dòng cũ. Sửa đè sẽ làm báo cáo kỳ cũ in ra tư cách sai.
- **Dễ hiểu sai #2:** `条件付き` không phải "tạm dừng". Người tham gia ở trạng thái đó **vẫn giao dịch được** trong giới hạn. Dựng giao diện như một trạng thái khoá là dựng sai nghiệp vụ.
- **Dễ hiểu sai #3:** cột `qualification_basis_rules.violation_action` hiện có trong schema nhưng **không dòng logic nào đọc** (D-01). Đừng dùng nó làm nguồn hành vi; nó là nhãn hiển thị.
- **Rủi ro phạm vi:** "hồ sơ người tham gia 4 biến thể" nằm trong 9 hạng mục ngoài phạm vi prototype (D-06). Đây là thiết kế, chưa có code nào đối chiếu được.
- Ma trận quyền (ghi/xem, 9 role — PREMISE C-09): `sys_admin` ✗/✗ · `party_admin` *(đề xuất)* ●/● (vai duy nhất được ghi) · `reconcile` ●/✗ · `trading` ●/✗ (**đề xuất**) · `receiving` ✗/✗ · `rule_admin`, `ops_admin`, `market_manager`, `internal_audit` *(đề xuất)* ✗/✗. `can_approve` không liên quan màn này (đổi trạng thái duyệt ở `SCR036`).
- Link chéo: [F002](../../features/F002_ParticipantsAndQualification/screens.md); flow FL-03; ADR-002, ADR-004, ADR-006, ADR-007.

### Unresolved Questions *(mục bổ sung)*

1. **`条件付き` cụ thể hạn chế cái gì?** B7 (trần số lượng/ngày) và B8 (chỉ `相対`) là **suy luận của tài liệu này** từ cụm "hạn chế có điều kiện" — tài liệu khách không liệt kê. Sai ở đây là sai thẳng vào `P0010`.
2. **Khách xác nhận cách duyệt `再審査`?** Hướng đã chốt giả định G8 (PREMISE): `party_admin` có cờ `can_approve`, khác người lập. Không đổi form; chỉ đổi ma trận quyền của `SCR036` nếu khách chọn hướng khác.
3. **Hệ thống phục vụ một chợ hay nhiều chợ?** Một chợ thì `market_id` ẩn hẳn và C2/D3 thành hằng.
4. **Hạn hiệu lực của A và D — ĐÃ CHỐT (G9 của PREMISE, người làm bài chốt; khách chưa xác nhận).** `valid_to` không bắt buộc với A (`市場登録`) và D (`卸売場登録`) vì LAB-1 F02-01 và F02-04 không ghi ngày hết hạn; B và C bắt buộc (F02-02, F02-03). Hệ quả: cột `valid_to` phải nới thành nullable (**đề xuất**, migration ở LAB-5); `有効 → 失効` tự động chỉ áp cho dòng có `valid_to`. Hướng loại: bắt buộc có hạn cho mọi căn cứ. Với D, quy chế chợ (`buyer_registration`) vẫn có thể áp hạn mặc định — không còn là điều kiện để quyết cột.
5. **Một người tham gia giữ được đồng thời hai căn cứ không?** Ảnh hưởng trực tiếp: nếu có thì `_assert_buyer_qualified` phải quyết định lấy căn cứ nào, và màn này phải cho nhiều khối biến thể.
6. **`許可` của 仲卸 có bị ràng buộc theo chợ trong thực tế không?** Tài liệu này theo `architecture.md` §2.3 cho là **không** (hiệu lực liên chợ). Nếu khách nói ngược lại thì `market_id` thành bắt buộc cho cả bốn biến thể.
7. **Đổi `party_type` của một người tham gia đang hoạt động** — cho hay không? Đề xuất: không, phải `取消` hồ sơ cũ và lập hồ sơ mới, vì đổi loại là đổi cả căn cứ lẫn bảng dữ liệu.
8. *(đã xử lý khi nộp lại 2026-10-06)* Mã `P0005` từng mang hai nghĩa (sai chức danh ở prototype; "mã đã tồn tại" ở bảng lỗi trên). "Mã đã tồn tại" nay là `P0069`; không còn việc treo cho mục này.
