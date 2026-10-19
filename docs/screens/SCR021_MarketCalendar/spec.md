# SCR021_MarketCalendar — Screen Spec

> Alias LAB-1: **SC-21** · 休市日・業務日カレンダー · Lịch nghỉ chợ & ngày nghiệp vụ

**Screen**: SCR021: MarketCalendar (Lịch nghỉ chợ & ngày nghiệp vụ)
**Feature**: [F006_ReconciliationAndClosing](../../features/F006_ReconciliationAndClosing/screens.md) — tính năng LAB-1 `F06-05`
**Type**: composite (lưới lịch 12 tháng + khối chuỗi 業務日 + điều khiển chế độ)
**Route**: `/markets/[id]/calendar` *(đề xuất — nằm dưới SCR037 vì lịch nghỉ là quy tắc theo chợ, ADR-007)*
**Generated**: 2026-10-06
**Trạng thái**: Thiết kế mới, chưa xây (LAB-4)
**Role**: `rule_admin` *(đề xuất, SCR037)* (xem + ghi ở chế độ nhập tay) · `reconcile` · `sys_admin` · `ops_admin` *(đề xuất)* (chỉ xem)
**User story**: không có US riêng — `product-a-user-stories.md` xếp `F06-05` vào nhóm *"Cấu hình — thiết lập một lần"*. `US-19` là US cùng khuôn (đổi quy tắc mà không sửa phần mềm).

## Purpose

**Quản trị quy chế** (`rule_admin`, chức danh của SCR037; giờ hành chính, tần suất rất thấp — thường mỗi năm một lần khi chợ công bố lịch năm sau, cộng vài lần sửa đột xuất) khai báo **休市日** của một chợ và xem **chuỗi 業務日** mà hệ thống sinh ra từ lịch đó. `F06-05` nói đúng một câu: *"nhập lịch nghỉ; hệ thống **bỏ qua ngày nghỉ khi sinh chuỗi ngày nghiệp vụ**"*. Câu trả lời khách đã cho ở câu 16 (tài liệu gốc trong repo, không nộp: `product-a-rfp-qa-part-a-client.md`) chốt lại: **`ROLE-RULE-ADMIN` quản trị trong hệ thống; 休市日 không sinh `業務日`**.

**Vì sao đây là màn có sức lan rộng nhất trong nhóm.** `業務日` không phải một nhãn hiển thị; nó là **khoá nghiệp vụ** của gần như mọi bảng: `lots.business_date`, `trades.business_date`, `business_days` (bảng chốt sổ), `RPT-01`…`RPT-09`, và mốc tính 支払期日 của `FR-INC-01`. Lịch nghỉ sai một ngày thì chuỗi `業務日` lệch, và mọi thứ đếm theo `業務日` lệch theo — kể cả `days_overdue` của SCR019/SCR020.

**Hai hướng, phải nêu cả hai.** LAB-1 để ngỏ: *"đồng bộ từ nguồn ngoài → màn hình thành chỉ xem, không nhập"* (`product-a-screen-list.md` dòng SC-21). Các mục Security Surface, Validation & Error Feedback, UI States và User Flow mô tả **cả hai chế độ**, chọn bằng một cờ cấu hình chứ không bằng hai màn khác nhau — xem Dependencies and Risks.

## Screen Layout

Trang trong khung ứng dụng chung, nằm dưới trang quản lý chợ (SCR037): dải thông tin chế độ lịch, bộ chọn năm, lưới 12 tháng mỗi ô là một ngày (tô theo `day_status`), khối điều khiển (`horizon_months`, "đã sinh tới ngày") và hộp thoại xác nhận khi bỏ một ngày nghỉ. Chưa có code nên chưa có breakpoint chốt.

### Layout Sketch

```
┌──────────────────────────────────────────────────────┐
│ R1: Header ứng dụng + nav (static)                │
├──────────────────────────────────────────────────────┤
│ R2: Chợ · chế độ lịch (manual | external_sync)       │
│     - - dải "Đồng bộ từ {nguồn}, lần gần nhất…" - -  │
│     [Năm ▾] · Đã sinh tới ngày · cảnh báo <30 ngày   │
├──────────────────────────────────────────────────────┤
│ R3: Lưới 12 tháng (業務日 / 休市日 / đã chốt / chưa sinh)│
│     [Đặt là 休市日] [Bỏ ngày nghỉ]  (chỉ rule_admin) │
├──────────────────────────────────────────────────────┤
│ R4: Danh sách ngày nghỉ (market_holidays)            │
└──────────────────────────────────────────────────────┘
   ┌─ R5: Hộp thoại "Bỏ ngày nghỉ" (ngày nào sinh thêm) ─┐
```

### Layout Regions

| Region ID | Name | Position | Scrollable | Key Components | Responsive Behavior |
|-----------|------|----------|------------|----------------|---------------------|
| R1 | Header + nav | static | no | khung `(app)` chung | always |
| R2 | Dải chế độ + chọn năm | static | no | badge `calendar_mode`, select năm, `generated_through` | fluid |
| R3 | Lưới lịch 12 tháng | static | yes | ô ngày, nút đặt/bỏ ngày nghỉ | fluid |
| R4 | Danh sách ngày nghỉ | static | yes | bảng `market_holidays` | fluid |
| R5 | Hộp thoại xác nhận | floating (modal) | no | nêu ngày được sinh thêm | conditional |

## Child Routes

Không áp dụng — màn này là một route đơn, không phải khung chứa route con.

## User Flow

> Phạm vi: tương tác trong màn. Điều hướng liên màn xem [screen-flow](../../generated/screen-flow.md).

### Happy Path

Chế độ nhập tay (`calendar_mode = manual` — giả định báo giá hiện tại):

1. `rule_admin` mở `/markets/[id]/calendar`, chọn năm → lưới 12 tháng, mỗi ô là một ngày, tô theo `day_status` (業務日 / 休市日 / đã chốt / chưa sinh).
2. Chọn một hoặc nhiều ngày → "Đặt là 休市日" → nhập `holiday_type` + `label` cho cả nhóm.
3. **Nhánh chặn:** ngày đã chốt sổ hoặc đã có dữ liệu → `P0058`, **cả lô không ghi** (nguyên tử — ghi một phần sẽ để lại lịch nửa vời).
4. Hệ thống **sinh lại chuỗi 業務日** cho khoảng từ ngày sớm nhất bị đổi tới hết `horizon_months`; banner nêu rõ số ngày thay đổi.
5. Xoá một ngày khỏi lịch nghỉ → hộp thoại xác nhận (xem Interaction Patterns) → ngày đó sinh `業務日` mới ở trạng thái `open`.
6. Nhập lịch năm sau: chọn năm kế → lặp bước 2. `generated_through` phải luôn đi trước ngày hiện tại ít nhất 30 ngày, màn cảnh báo khi không còn.

Chế độ nguồn ngoài (`calendar_mode = external_sync`):

7. Màn mở ở **chỉ xem**. Lưới hiển thị y hệt, mọi thao tác ghi trả `P0059`.
8. Dải thông tin trên đầu: nguồn, `synced_at`, số ngày nghỉ của năm đang xem.
9. Đồng bộ chạy nền. **Thất bại đồng bộ phải sinh cảnh báo ở SCR033** (`source = report_batch` hoặc một nguồn thứ tư) — lịch nghỉ lỗi thời mà im lặng là tình huống tệ nhất của màn này: hệ thống vẫn chạy, vẫn sinh 業務日, chỉ là sai.
10. Ngày nghỉ đến từ nguồn ngoài **vẫn chịu ràng buộc `P0058`**: nguồn ngoài báo nghỉ một ngày đã có giao dịch thì **không áp dụng**, ghi vào hàng cảnh báo để người quyết định, không tự ghi đè dữ liệu thật.

### Branches

| Decision point | Condition | Outcome on this screen | Source |
|----------------|-----------|------------------------|--------|
| Bước 3 | ngày đã chốt / đã có dữ liệu | `P0058`, cả lô không ghi | thiết kế LAB-4 |
| Bước 7 | `calendar_mode = external_sync` | toàn lưới readonly, ghi trả `P0059` | thiết kế LAB-4 |
| Bước 1 | role ≠ `rule_admin` | chỉ xem | thiết kế LAB-4 |
| Bước 6 | `generated_through` còn < 30 ngày phía trước | cảnh báo | thiết kế LAB-4 |

## Data Inventory

> Chỉ field chỉ đọc. Ô nhập nằm ở `## Validation & Error Feedback`.

### Khối lịch nghỉ — `market_holidays`, mỗi dòng là một ngày

| Field | Display Label | Source | Format | Empty Behavior | Cross-ref |
|-------|---------------|--------|--------|----------------|-----------|
| `source` | Nguồn / 取得元 | `market_holidays.source` | badge ∈ `manual` `external_sync`; dòng `external_sync` **không sửa, không xoá bằng tay**; mặc định `manual` | luôn có | market_holidays *(mới, đề xuất)* |
| `synced_at` | Đồng bộ lúc / 同期日時 | `market_holidays.synced_at` | datetime; chỉ có với `source = external_sync` | "—" | market_holidays |
| `created_by` / `created_at` | Người nhập / Lúc nhập / 登録者 / 登録日時 | `market_holidays.created_*` | server đặt | luôn có | market_holidays |

### Khối chuỗi 業務日 — dẫn xuất, **chỉ đọc tuyệt đối**

| Field | Display Label | Source | Format | Empty Behavior | Cross-ref |
|-------|---------------|--------|--------|----------------|-----------|
| `business_date` | 業務日 | `business_days.business_date` | date; **chỉ sinh cho ngày KHÔNG có dòng trong `market_holidays`** — toàn bộ nội dung của `F06-05` | — | business_days |
| `day_status` | Trạng thái ngày / 状態 | `business_days.status` hoặc suy ra | badge ∈ `open` `closed`, thêm nhãn hiển thị `holiday` cho ngày nghỉ và `not_generated` cho ngày tương lai chưa sinh | — | business_days |
| `generated_through` | Đã sinh tới ngày / 生成済み最終日 | `max(business_days.business_date)` | date; ngày cuối cùng đã có trong `business_days`; **cảnh báo khi còn < 30 ngày phía trước** — hết chuỗi là không mở được ngày mới | luôn có | business_days |
| `lock_state` | Đã chốt sổ / 締め状態 | `business_days.status = 'closed'` | badge; `closed` ⇒ ngày đã lock (SCR018), **không đổi được thành ngày nghỉ** → `P0058` | — | business_days |
| `has_data` | Có dữ liệu / データ有無 | đếm từ `lots`, `trades` theo `business_date` | cờ; `true` khi ngày đó đã có `lots`/`trades`/`deliveries`; **không đổi được thành ngày nghỉ** → `P0058` | `false` | lots, trades |

### Khối điều khiển (chỉ đọc)

| Field | Display Label | Source | Format | Empty Behavior | Cross-ref |
|-------|---------------|--------|--------|----------------|-----------|
| `calendar_mode` | Chế độ lịch / カレンダー運用方式 | `markets.calendar_mode` *(đề xuất)* | badge ∈ `manual` `external_sync`; **quyết định màn này nhập được hay chỉ xem**; mặc định `manual` (theo giả định báo giá câu 16) | luôn có | markets |

### Read Calls *(mục bổ sung)*

| Thao tác | Cách gọi | Tham số | Trả về |
|---|---|---|---|
| Đọc lịch nghỉ một năm | `select` từ `market_holidays` | `market_id`, `year` | mảng dòng |
| Đọc chuỗi 業務日 + trạng thái | `select` từ `v_business_calendar` (`security_invoker`) — gộp `business_days` với `has_data` và `lock_state` | `market_id`, `year` | mảng ô lịch |
| Tra một ngày có phải 業務日 không | `rpc('is_business_day', { p_market_id, p_date })` | | `boolean` — **hàm dùng chung** cho SCR020 (`due_date`), SCR032 (`digest_at`), SCR019 (`days_overdue`) |

## UI States

| State | Trigger | Visual Behavior | User Action Available | Source |
|-------|---------|----------------|-----------------------|--------|
| empty | chợ chưa khai báo ngày nghỉ nào | "Chưa khai báo 休市日 nào — **mọi ngày đều sinh 業務日**." — câu thứ hai bắt buộc, vì rỗng ở màn này **có hậu quả**, không trung tính | Đặt ngày nghỉ | thiết kế LAB-4 |
| loading | mở trang / đổi năm | khung xương lưới 12 tháng | none | thiết kế LAB-4 |
| success | sau khi thêm/xoá ngày nghỉ | banner `role="status"`: "Đã cập nhật. **Chuỗi 業務日 từ {date} trở đi đã sinh lại; {n} ngày thay đổi.**" | dismiss | thiết kế LAB-4 |
| error (nghiệp vụ) | `P0058`/`P0059`/`P0005` | banner đỏ **tự nhận focus**; lịch không đổi một dòng nào | sửa | thiết kế LAB-4 |
| error (mạng) | mất kết nối lúc lưu | banner + nút thử lại; **ngày đã chọn trên lưới giữ nguyên** | thử lại | thiết kế LAB-4 |
| read-only (do chế độ) | `calendar_mode = external_sync` | toàn lưới readonly, ẩn hẳn nút thêm/xoá, dải "Đồng bộ từ {nguồn}, lần gần nhất {synced_at}" + nút "Đồng bộ lại" nếu khách cho phép kích hoạt tay | Đồng bộ lại | thiết kế LAB-4 |
| read-only (do chức danh) | role ≠ `rule_admin` | như trên nhưng không có nút đồng bộ | xem | thiết kế LAB-4 |

## Validation & Error Feedback

### A) Client-side

| Field | Type | Required | Constraints | Async Check | Error Message |
|-------|------|----------|-------------|-------------|---------------|
| `market_id` / 市場 | select (hoặc readonly khi một chợ) | yes | phải tồn tại trong `markets`; **ADR-007: hai chợ nghỉ khác ngày là hợp lệ và phải biểu diễn được**; mặc định chợ đang chọn ở SCR037 | không | xem Error Catalogue |
| `calendar_date` / 日付 | date | yes | duy nhất theo `(market_id, calendar_date)`; `≥ 業務日` đang mở — xem `P0058` cho ngày quá khứ | không (`P0058` ở server) | xem Error Catalogue |
| `holiday_type` / 休市区分 | select | yes | ∈ `weekly` (nghỉ định kỳ trong tuần) · `national` (lễ) · `market_specific` (chợ tự nghỉ) · `temporary` (nghỉ đột xuất) *(đề xuất)*; mặc định `market_specific` | không | xem Error Catalogue |
| `label` / 名称 | text | yes | 1–60 ký tự; ví dụ `年末年始`, `棚卸` | không | xem Error Catalogue |
| `note` / 備考 | text | no | 0–500 ký tự; mặc định rỗng | không | xem Error Catalogue |
| `horizon_months` / 生成先行期間 | number | yes | nguyên, `3`–`24`; chuỗi `業務日` được sinh trước ngần ấy tháng; mặc định `12` *(đề xuất)*; lưu `markets.calendar_horizon_months` | không | xem Error Catalogue |
| bộ lọc `year` / 年 | select | no | mặc định năm hiện tại theo múi giờ chợ; `searchParams.year` | không | không có |

### B) Server-side

Mọi đường ghi qua RPC `SECURITY DEFINER`; `market_holidays` và `business_days` **không có policy ghi**. Ba bất biến nằm trong thân RPC: **ngày đã chốt/có dữ liệu không thành 休市日 được**, **chế độ nguồn ngoài khoá mọi ghi**, và **sinh lại chuỗi 業務日 nằm cùng transaction với thay đổi lịch** — tách ra là có cửa sổ mà lịch đã đổi còn chuỗi ngày thì chưa.

#### Đặt ngày nghỉ (một lô)
- **Endpoint:** `rpc('set_market_holidays')` *(đề xuất)*
- **Request:** `p_market_id, p_dates[], p_holiday_type, p_label, p_note`
- **Success:** `jsonb { added, skipped, regenerated_days }` → banner xanh nêu số ngày đổi
- **Errors:** `P0058` · `P0059` · `P0005`
- **Trigger:** nút "Đặt là 休市日"
- **Source:** chưa có code — màn chỉ đặc tả ở LAB-4

#### Bỏ ngày nghỉ (một lô)
- **Endpoint:** `rpc('clear_market_holidays')` *(đề xuất)*
- **Request:** `p_market_id, p_dates[]`
- **Success:** `jsonb { removed, regenerated_days }`
- **Errors:** `P0059` · `P0005`
- **Trigger:** xác nhận trong hộp thoại bỏ ngày nghỉ
- **Source:** chưa có code — màn chỉ đặc tả ở LAB-4

#### Sinh chuỗi 業務日 tới hạn mức
- **Endpoint:** `rpc('generate_business_days')` *(đề xuất)*
- **Request:** `p_market_id, p_through`
- **Success:** `jsonb { generated_count, generated_through }`
- **Errors:** theo Error Catalogue
- **Trigger:** thay đổi lịch / `horizon_months` / đồng bộ nền
- **Source:** chưa có code — màn chỉ đặc tả ở LAB-4

**`is_business_day` là một hàm, không phải một bản sao logic ở mỗi màn.** Đây là điểm DRY quan trọng nhất của nhóm: SCR019 đếm `days_overdue`, SCR020 kiểm `due_date`, SCR032 đặt `digest_at`, `FR-INC-01` tính 支払期日 — bốn chỗ cùng hỏi một câu. Bốn bản sao công thức sẽ lệch nhau ngay lần đầu lịch nghỉ thay đổi, và không chỗ nào báo lỗi.

#### Error Catalogue *(mục bổ sung)*

| Mã | Điều kiện kích hoạt | Thông báo VI | Thông báo JA | Hiện ở đâu | Chặn gửi? |
|---|---|---|---|---|:---:|
| `V-01` | ô bắt buộc để trống | Mục này bắt buộc. | 必須項目です。 | dưới ô | ● |
| `V-02` | `label` > 60 ký tự | Tên ngày nghỉ tối đa 60 ký tự. | 名称は60文字以内で入力してください。 | dưới ô | ● |
| `V-03` | `horizon_months` ngoài 3–24 | Thời gian sinh trước phải từ 3 đến 24 tháng. | 生成先行期間は3〜24か月で指定してください。 | dưới ô | ● |
| `P0057` *(mới, đề xuất — định nghĩa chuẩn ở màn này)* | một màn khác chọn một ngày là 休市日 làm `業務日` (ví dụ `due_date` ở `SCR020`) | Ngày {date} là 休市日 của chợ「{market}」, không phải 業務日. | {date}は市場「{market}」の休市日のため業務日ではありません。 | dưới ô ngày của màn gọi | ● |
| `P0058` *(mới, đề xuất)* | đánh dấu nghỉ một ngày **đã sinh 業務日** mà ngày đó `status = 'closed'` hoặc `has_data = true` | Ngày {date} đã có dữ liệu nghiệp vụ (hoặc đã chốt sổ), không đặt thành 休市日 được. | {date}は業務データが存在する（または締め済み）ため休市日にできません。 | banner đầu trang | ● |
| `P0059` *(mới, đề xuất)* | mọi lời gọi ghi khi `calendar_mode = external_sync` | Lịch nghỉ của chợ này đồng bộ từ nguồn ngoài — màn hình chỉ xem. | この市場の休市日は外部ソースから同期されるため、画面上での編集はできません。 | banner đầu trang | ● |
| `P0005` | role ≠ `rule_admin` | Chức danh của bạn không được phép sửa lịch nghỉ. | あなたの職掌では休市日を変更できません。 | banner đầu trang | ● |
| — | thêm một ngày nghỉ đã tồn tại | (không phải lỗi) "Ngày {date} đã là 休市日." — thao tác **không làm gì**, không ghi thêm dòng | {date}はすでに休市日です。 | banner trung tính | không — **idempotent có chủ ý** |

Rẽ nhánh theo `error.code`; thông báo dựng từ `error.details` (`date`, `market`, `reason`) — không so khớp chuỗi `message`.

Dòng cuối là quyết định có chủ ý: nhập trùng ngày nghỉ là **chuyện bình thường** khi dán lịch năm từ bảng tính. Biến nó thành lỗi chỉ làm người dùng phải dò thủ công xem dòng nào đã có. Ràng buộc `UNIQUE (market_id, calendar_date)` vẫn giữ ở tầng bảng; RPC nuốt xung đột và trả nguyên trạng.

## Interaction Patterns

- **Không có yêu cầu bàn phím riêng** — không thuộc ba màn keyboard-first (SCR007, SCR010, SCR014) — thiết kế LAB-4.
- **Xoá một ngày khỏi lịch nghỉ (biến 休市日 thành 業務日) có hộp thoại xác nhận chặn luồng, nêu rõ ngày nào sẽ được sinh thêm và chuỗi 業務日 phía sau có dịch không** — vì đây là thao tác duy nhất trên màn này **thêm** một ngày vào chuỗi, và nó chạm vào mốc tính 支払期日 của `FR-INC-01`; thiết kế LAB-4.
- **Chọn nhiều ngày trên lưới rồi đặt `holiday_type` + `label` cho cả nhóm; ghi nguyên tử cả lô** — thiết kế LAB-4.

## Accessibility

| Aspect | Status | Notes |
|--------|--------|-------|
| ARIA roles/labels | partial (đặc tả) | banner `role="status"`; lưới lịch cần nhãn ngày + trạng thái bằng chữ, không chỉ màu (đề xuất) |
| Keyboard navigation | unknown | chưa đặc tả điều hướng ô lưới bằng phím mũi tên — [NEEDS_DOMAIN_CONFIRMATION] |
| Focus management | managed (đặc tả) | banner lỗi tự nhận focus; hộp thoại xác nhận chặn luồng |
| Screen reader compatibility | unknown | chưa có bản dựng để kiểm |

## Conditional Rendering

| Condition | Type | Renders | Hidden | Notes |
|-----------|------|---------|--------|-------|
| `calendar_mode = external_sync` | feature-flag | dải "Đồng bộ từ {nguồn}…", lưới readonly | nút thêm/xoá | ghi vẫn bị RPC chặn `P0059` bất kể chức danh |
| role = `rule_admin` và `calendar_mode = manual` | auth | nút đặt/bỏ ngày nghỉ | — | bypass: RPC `P0005` |
| role ≠ `rule_admin` | auth | chỉ xem | nút ghi, nút đồng bộ | |
| `generated_through` còn < 30 ngày | hardcoded-id | cảnh báo | — | hết chuỗi là không mở được ngày mới |
| `lock_state = closed` hoặc `has_data = true` | hardcoded-id | ô không chọn được làm 休市日 | — | `P0058` |

## Component Variants

Không áp dụng — màn không có component đổi hình dạng theo biến thể; khác biệt theo chức danh/trạng thái đã mô tả ở Conditional Rendering.

## Security Surface

| Guard | Type | Consequence if bypassed |
|-------|------|------------------------|
| `route-policy.ts` — prefix `/markets` của SCR037 đã phủ route này, chỉ cần thêm `reconcile`/`ops_admin` vào danh sách role *(đề xuất)* | role | role ngoài danh sách mở được màn |
| RLS `market_holidays` SELECT theo bốn chức danh; **không có policy ghi** (ADR-002) | permission | đọc/ghi ngoài chức danh |
| RPC: chế độ nguồn ngoài từ chối mọi lời gọi ghi bằng `P0059` bất kể chức danh | permission | khoá ở dữ liệu, không khoá bằng cách ẩn nút; `[UNVERIFIED]` — RPC chưa tồn tại |

Quyền theo chức danh: `rule_admin` *(SCR037)* xem + ghi (chỉ chế độ nhập tay) — LAB-1 gọi là "Quản trị quy chế"; khách xác nhận `ROLE-RULE-ADMIN` ở câu 16 · `reconcile` xem — cần biết ngày nào là 業務日 để giải thích bảng đối chiếu và SLA tranh chấp · `sys_admin` xem — hỗ trợ, không quyết định nghiệp vụ · `ops_admin` *(đề xuất)* xem — cần lịch để đọc `digest_at` và cửa sổ batch · `receiving` · `trading` · `party_admin` ✗. Chế độ **nguồn ngoài**: không chức danh nào ghi được.

## Source References

Chưa có code — màn chỉ đặc tả ở LAB-4. Tài liệu nguồn đã đọc:

1. (tài liệu gốc trong repo, không nộp) `sc-21-lich-nghi-cho.md` — spec 10 mục gốc
2. (tài liệu gốc trong repo, không nộp) `adr.md` — ADR-007 (quy tắc theo chợ), ADR-002
3. `supabase/migrations/0004_business_days_and_lots.sql` — `business_days` hiện có (chưa có bảng `market_holidays`)

## Source Walkthrough

Chưa có code — màn chỉ đặc tả ở LAB-4.

### Call Hierarchy

```text
(thiết kế) /markets/[id]/calendar page -> market_holidays + v_business_calendar (đọc)
   -> Server Action -> rpc set_market_holidays | clear_market_holidays | generate_business_days
is_business_day() <- SCR019 · SCR020 · SCR032 · FR-INC-01
```

## Dependencies & Risks *(mục bổ sung)*

- **Cần trước:** `SCR037` — không có thực thể chợ thì không có `market_id`, và theo ADR-007 lịch nghỉ **là quy tắc riêng từng chợ**: hai chợ nghỉ khác ngày là tình huống bình thường, không phải ngoại lệ. Màn này nằm dưới route của `SCR037` chính vì thế.
- **Quan hệ với `market_rules` của `SCR037` — một khác biệt có chủ ý.** Lịch nghỉ **không** mượn khuôn phiên bản `draft → approved → superseded` của `market_rules`. Lý do: `market_rules` cần phiên bản vì **in lại báo cáo kỳ cũ phải ra đúng số cũ** (`FR-INC-03`), nghĩa là quá khứ phải tra lại được bằng `as_of`. Lịch nghỉ không có bài toán đó — vì `business_days` **đã vật chất hoá**: ngày quá khứ đã nằm thành dòng thật trong bảng, và `P0058` cấm sửa ngược. Quá khứ đã bất biến rồi, phiên bản hoá thêm một lần nữa là chi phí không mua được gì (**YAGNI**). Cái vẫn giữ là **dấu vết**: mọi lần đổi ghi `created_by`/`created_at` và một dòng audit `F11`.
- **Phạm vi lan rộng:** đây là màn chạm nhiều thứ nhất trong sáu màn. `business_days` là bảng mà `SCR018` chốt sổ, `lots`/`trades` tham chiếu, `RPT-01`…`RPT-09` nhóm theo, và `FR-INC-01` tính 支払期日 từ. Ước lượng sai độ lan rộng của màn này là rủi ro estimate thật.
- **Suy luận có đánh dấu:** LAB-1 cho SCR021 đúng một dòng. Phần **bắt buộc theo khách**: nhập lịch nghỉ, 休市日 không sinh `業務日`, `ROLE-RULE-ADMIN` quản trị trong hệ thống, và khả năng chuyển sang chỉ-xem nếu đồng bộ nguồn ngoài. Phần **đề xuất của tài liệu này**: bốn `holiday_type`, `horizon_months`, cờ `calendar_mode`, hành vi idempotent khi trùng ngày, và hàm dùng chung `is_business_day`.
- **Dễ hiểu sai #1:** 休市日 **không phải** `business_days.status = 'closed'`. `closed` nghĩa là ngày nghiệp vụ **đã diễn ra và đã chốt sổ**; 休市日 nghĩa là **ngày đó không tồn tại trong `business_days`**. Lẫn hai thứ này làm `RPT-01` đếm thừa ngày và `days_overdue` đếm sai.
- **Dễ hiểu sai #2:** đổi lịch nghỉ chỉ ảnh hưởng **tương lai**. Không có đường nào sinh lại `業務日` quá khứ — `P0058` chặn, và đó là tính năng chứ không phải hạn chế.
- **Dễ hiểu sai #3:** 休市日 vẫn có thể có **giao hàng kéo dài** hoặc email gửi đi. Nghỉ chợ nghĩa là **không sinh 業務日**, không nghĩa là hệ thống tắt.
- **Rủi ro vận hành #1:** `generated_through` cạn mà không ai để ý ⇒ một sáng không mở được ngày nghiệp vụ, trong khi ca bắt đầu lúc 02:00 và không có ai trực kỹ thuật. Vì thế `generated_through` có cảnh báo, và nên có cả thông báo `F10` khi còn < 30 ngày — **nhưng loại sự kiện này không nằm trong 4 loại của `F10-02`**, xem Unresolved Questions.
- **Rủi ro vận hành #2 (chế độ nguồn ngoài):** nguồn ngoài im lặng không trả dữ liệu thì hệ thống vẫn chạy với lịch cũ và **không có triệu chứng nào nhìn thấy được**. Bước 9 User Flow bắt buộc có cảnh báo, không phải tuỳ chọn.
- **Rủi ro cutover:** `R-28` của sổ rủi ro LAB-1 chọn **休市日 làm cửa sổ cutover**. Nghĩa là lịch này phải đúng **trước** khi cutover chứ không phải sau — một phụ thuộc về thứ tự triển khai, không phải về kỹ thuật.

### Unresolved Questions *(mục bổ sung)*

1. **Nếu khách đổi sang đồng bộ nguồn ngoài thì sao?** Đã chốt `calendar_mode = manual` làm mặc định (PREMISE C-08, giả định báo giá, câu 16 của bộ hỏi-đáp LAB-1 `product-a-rfp-qa-part-a-client.md`); màn thiết kế theo hướng này và chỉ giữ cờ `calendar_mode`. Chỉ còn lại rủi ro: nếu khách đổi hướng thì toàn bộ khối ghi ở Data Inventory biến mất, `P0058`/`P0059` đổi vai, và phát sinh một hạng mục **tích hợp** chưa có trong ước lượng (giao thức, lịch chạy, xử lý lệch dữ liệu, cảnh báo đồng bộ lỗi).
2. **Hệ thống một chợ hay nhiều chợ?** Cùng câu hỏi gốc với ADR-007 và `SCR037` Unresolved Questions. Một chợ ⇒ bỏ bước chọn chợ, `market_id` thành hằng. Nhiều chợ ⇒ mỗi chợ một lịch, và mọi truy vấn đếm theo `業務日` phải mang `market_id`.
3. **休市日 có bao giờ cần sinh `業務日` không?** Khách trả lời *"休市日 không sinh 業務日"*, nhưng chưa nói về **ngày làm bù** hoặc **chợ mở ngoại lệ**. Nếu có thì cần một `holiday_type` đảo chiều (`special_open`), không chỉ là xoá dòng.
4. **Cảnh báo "sắp hết chuỗi 業務日" gửi qua kênh nào?** `F10-02` khoá đúng **4 loại sự kiện**; đây là loại thứ 5. Hoặc mở rộng `F10-02` (đổi phạm vi), hoặc đẩy vào `SCR033` như một cảnh báo vận hành (rẻ hơn, và đúng tinh thần `F09-06`). Tài liệu này nghiêng về phương án hai.
5. **`horizon_months` là cấu hình hay hằng số?** Để thành `horizon_months` cho linh hoạt, nhưng nếu khách không bao giờ đổi thì đây là một ô thừa (**YAGNI**) — cắt được mà không ảnh hưởng gì.
6. **Lịch nghỉ có ảnh hưởng 支払期日 không, và ảnh hưởng thế nào?** Câu 10 (tài liệu LAB-1 `product-a-rfp-qa-part-a-client.md`) (định nghĩa 支払期日) và câu 16 liên quan trực tiếp với nhau — khách tự nêu điều này trong văn bản hỏi. "N ngày từ `業務日`" đếm theo **ngày lịch** hay theo **業務日**? Hai cách cho ra hai ngày khác nhau mỗi khi có 休市日 xen vào, và nó chạm vào tiền thật của `F08`.
