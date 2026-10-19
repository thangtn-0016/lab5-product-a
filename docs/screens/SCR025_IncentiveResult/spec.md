# SCR025_IncentiveResult — Screen Spec

> Alias LAB-1: **SC-25** · 完納奨励金 計算結果 · Kết quả tính tiền khuyến khích

**Screen**: SCR025: IncentiveResult (Kết quả tính tiền khuyến khích)
**Feature**: [F008_IncentiveAndRateVersion](../../features/F008_IncentiveAndRateVersion/screens.md) — tính năng LAB-1 `F08-01` `F08-02` `F08-06` `F11-03`
**Type**: composite (bảng theo kỳ + bung chi tiết một dòng + nút chạy tính)
**Route**: `/incentives` (bảng theo kỳ) · `/incentives/[period_id]/[participant_id]` (bung chi tiết một dòng) *(đề xuất)*
**Generated**: 2026-10-06
**Trạng thái**: Thiết kế mới, chưa xây (LAB-4)
**Role**: `reconcile` (xem + chạy tính kỳ đang mở) · `rule_admin` *(mới, đề xuất)* (chỉ xem)
**User story**: `US-13`

## Purpose

Bộ phận đối chiếu (`reconcile`, 照合事務; giờ hành chính sau khi ngày nghiệp vụ đã chốt; không phải màn hiện trường — không đeo găng tay, không 2 giờ sáng) xem kết quả tính 完納奨励金 **theo kỳ và theo người tham gia**, kèm **phiên bản 料率 đã áp dụng** và `発効日` của bản đó — để khi có thắc mắc về một con số cũ, mở đúng một dòng là giải thích được, không phải đi tìm bảng tính nào đã dùng (`ASIS-06`).

**Vì sao màn này không phải báo cáo tĩnh.** `F08-06` đòi điều chỉnh sau khi chốt kỳ sinh **chênh lệch ở kỳ sau** thành **dòng riêng có nguyên nhân**, chứ không sửa đè dòng cũ. Một bảng chỉ hiện tổng thì không diễn đạt được "số kỳ này gồm phần của kỳ này cộng phần bù của kỳ trước".

**Công thức bị khách pin cứng, không đàm phán** (`BR-INC-01`): hệ số thuế `110/100`, **làm tròn xuống (切り捨て) theo đơn vị JPY**, **không có 猶予期間**, quá 支払期日 thì 完納奨励金 **= 0** (không giảm dần), mốc thời gian theo **JST + 業務日**.

## Screen Layout

Trang trong khung ứng dụng chung: tiêu đề kèm nút "Tính kỳ này" (chỉ `reconcile`, kỳ đang mở), thanh bộ lọc, bảng một dòng = một (kỳ × người tham gia × loại dòng); bấm một dòng bung chi tiết (route con). Nút "Tính kỳ này" mở hộp thoại xác nhận chặn luồng. Chưa có code nên chưa có breakpoint chốt.

### Layout Sketch

```
┌──────────────────────────────────────────────────────┐
│ R1: Header ứng dụng + nav (static)                │
├──────────────────────────────────────────────────────┤
│ R2: Tiêu đề · [Tính kỳ này] · [Xuất CSV]             │
│     banner kết quả/lỗi (role=status | alert)         │
├──────────────────────────────────────────────────────┤
│ R3: Bộ lọc: kỳ · người tham gia · loại dòng · ☐ =0   │
├──────────────────────────────────────────────────────┤
│ R4: Bảng kết quả (main / delta), cột v{n} → SCR027   │
│     - - dòng bung chi tiết (route con) - -           │
└──────────────────────────────────────────────────────┘
   ┌─ R5: Hộp thoại "Tính kỳ này" (kỳ · bản 料率 · ghi đè tạm) ─┐
```

### Layout Regions

| Region ID | Name | Position | Scrollable | Key Components | Responsive Behavior |
|-----------|------|----------|------------|----------------|---------------------|
| R1 | Header + nav | static | no | khung `(app)` chung | always |
| R2 | Tiêu đề + hành động + banner | static | no | `PageHeader`, nút tính, nút CSV, banner | fluid |
| R3 | Bộ lọc | static | no | select kỳ/người/loại, checkbox | fluid, wrap |
| R4 | Bảng kết quả | static | yes | `DataTable`, `StatusBadge`, link `v{n}` | fluid |
| R5 | Hộp thoại xác nhận tính | floating (modal) | no | nêu kỳ, bản 料率, hệ quả ghi đè | conditional |

## Child Routes

Không áp dụng — màn này là một route đơn, không phải khung chứa route con.

## User Flow

> Phạm vi: tương tác trong màn. Điều hướng liên màn xem [screen-flow](../../generated/screen-flow.md).

### Happy Path

1. `reconcile` mở `/incentives`, màn mặc định chọn **kỳ gần nhất có kết quả**.
2. Lọc theo người tham gia / loại dòng nếu cần.
3. Bấm một dòng → bung chi tiết: số tiền đủ điều kiện, 交付率, số trước làm tròn, phần bị cắt khi 切り捨て, 支払期日, 完納日.
4. Bấm `v{n}` ở cột **適用料率版** → sang SCR027 chỉ đọc, thấy ai lập, ai duyệt, `発効日`. Đây là `F11-03`.
5. Kỳ **đang mở**: bấm "Tính kỳ này" → xác nhận → RPC chạy, bảng nạp lại.
6. Có điều chỉnh của kỳ đã chốt (từ `F07`): lần chạy tính **kỳ kế tiếp** sinh thêm dòng `row_type = delta`, có `delta_origin_period` và `delta_reason`. **Kỳ cũ không đổi một con số nào** — in lại báo cáo kỳ cũ vẫn ra đúng số cũ (`FR-INC-03`).
7. Xuất CSV (`RPT-07`) — xuất đúng tập dòng đang lọc, kèm cột 適用料率版 và 発効日.

### Branches

| Decision point | Condition | Outcome on this screen | Source |
|----------------|-----------|------------------------|--------|
| Bước 5 | kỳ còn ngày nghiệp vụ chưa chốt | `P0038` cảnh báo, **vẫn cho chạy**, kết quả đánh dấu là tạm | thiết kế LAB-4 |
| Bước 5 | kỳ đã chốt | `P0035`, không chạy | thiết kế LAB-4 |
| Bước 5 | `P0034`/`P0005` | banner tự nhận focus, bộ lọc giữ nguyên | thiết kế LAB-4 |
| Bước 1 | kỳ chưa chạy tính lần nào | "Kỳ này chưa có kết quả tính." + nút "Tính kỳ này" (chỉ `reconcile`) | thiết kế LAB-4 |
| Bước 6 | có phiếu điều chỉnh kỳ đã chốt | dòng `delta` kèm `delta_origin_period`/`delta_reason` | thiết kế LAB-4 |

## Data Inventory

> Cột bảng — mỗi dòng là **một (kỳ × người tham gia × loại dòng)**. Bộ lọc nằm ở `## Validation & Error Feedback`.

| Field | Display Label | Source | Format | Empty Behavior | Cross-ref |
|-------|---------------|--------|--------|----------------|-----------|
| `participant` | Người tham gia / 参加者 | `participants.code`, `.name` | mã + tên | luôn có | participants |
| `row_type` | Loại dòng / 行種別 | `incentive_results.row_type` | badge: `main` tông trung tính · `delta` tông cảnh báo | luôn có | incentive_results *(mới, đề xuất)* |
| `eligible_amount` | Số tiền đủ điều kiện / 対象金額 | `incentive_results.eligible_amount` | number `≥ 0`, JPY, không thập phân | — | incentive_results |
| `grant_rate` | 交付率 áp dụng / 適用交付率 | `incentive_results.grant_rate_applied` | number; sao **tại lúc tính**, không join lại | — | incentive_results |
| `tax_factor` | Hệ số thuế / 税係数 | hằng số `BR-INC-01` | **luôn `110/100`** — hằng số quốc gia, không phải cấu hình | `110/100` | — |
| `raw_amount` | Số trước làm tròn / 丸め前金額 | `incentive_results.raw_amount` | number 2 chữ số thập phân; hiện để giải thích chênh lệch | — | incentive_results |
| `incentive_amount` | 完納奨励金 | `incentive_results.incentive_amount` | **`floor(raw_amount)`** theo JPY; `≥ 0` | — | incentive_results |
| `rate_version` | Phiên bản 料率 / 適用料率版 | `incentive_results.rate_version_id` | link `v{n}` → mở SCR027 ở chế độ chỉ đọc. **`F11-03`** | — | incentive_rate_versions |
| `rate_effective_from` | Ngày hiệu lực của bản / 発効日 | `incentive_rate_versions.effective_from` | date; ngày của **bản đã áp dụng**, không phải bản đang hiệu lực hôm nay | — | incentive_rate_versions |
| `payment_due_date` | 支払期日 | `incentive_results.payment_due_date` | date, ngày nghiệp vụ JST | — | incentive_results |
| `paid_on` | Ngày thanh toán xong / 完納日 | `incentive_results.paid_on` | date; rỗng khi chưa hoàn tất | — | incentive_results |
| `zero_reason` | Nguyên nhân bằng 0 / ゼロ理由 | `incentive_results.zero_reason` | ∈ `overdue` (`paid_on > payment_due_date`) · `unpaid` (`paid_on` rỗng tại thời điểm tính) · `below_threshold`; rỗng khi `incentive_amount > 0` | rỗng | incentive_results |
| `delta_origin_period` | Bù cho kỳ / 対象元期 | `incentive_results.origin_period_id` | **chỉ có với `row_type = delta`**; trỏ kỳ đã chốt bị điều chỉnh | rỗng với `main` | incentive_results |
| `delta_reason` | Nguyên nhân chênh lệch / 差分理由 | `incentive_results.delta_reason` | **chỉ có với `row_type = delta`**; 10–1000 ký tự, chép từ phiếu điều chỉnh `F07` | rỗng với `main` | incentive_results |
| `calculated_at` | Lúc tính / 計算日時 | `incentive_results.calculated_at` | datetime JST, server đặt | — | incentive_results |

**Cột `grant_rate`, `rate_version`, `rate_effective_from` là cột lưu, không phải cột join động.** `F08-02` đòi mỗi kết quả **gắn với phiên bản cụ thể**; đọc 料率 bằng cách tra bản *đang hiệu lực hôm nay* sẽ làm số cũ đổi theo thời gian — đúng thứ `FR-INC-03` cấm.

### Read Calls *(mục bổ sung)*

| Thao tác | Cách gọi | Tham số | Trả về |
|---|---|---|---|
| Đọc danh sách kỳ | `select` từ `settlement_periods` | — | `id`, `label`, `status`, `open_business_days` |
| Đọc bảng kết quả | `select` từ `v_incentive_results` | `period_id`, `participant_id?`, `row_type?` | mảng dòng, đã join sẵn `rate_version` + `effective_from` |

Đọc dùng `select` thẳng vì chỉ hiển thị.

**`v_incentive_results` join `incentive_rate_versions` theo `rate_version_id` đã lưu trên dòng** — không theo `effective_from ≤ current_date`. Join theo ngày hôm nay là cách phổ biến nhất để vô tình viết lại quá khứ.

## UI States

| State | Trigger | Visual Behavior | User Action Available | Source |
|-------|---------|----------------|-----------------------|--------|
| empty | kỳ đã chọn chưa chạy tính lần nào | "Kỳ này chưa có kết quả tính." + nút "Tính kỳ này" (chỉ `reconcile`) | Tính kỳ này | thiết kế LAB-4 |
| loading | đổi bộ lọc | khung xương bảng, giữ nguyên bộ lọc đã chọn | none | thiết kế LAB-4 |
| saving | đang chạy `calculate_incentive` | nút `disabled` | none | thiết kế LAB-4 |
| success | sau khi chạy tính | banner `role="status"`: "Đã tính kỳ {period}: {n} dòng, tổng {total} JPY, dùng bản 料率 v{v}"; bảng tự nạp lại | dismiss | thiết kế LAB-4 |
| error (nghiệp vụ) | RPC từ chối (`P0035`, `P0034`, `P0005`) | banner **tự nhận focus**; bộ lọc giữ nguyên | sửa bộ lọc | thiết kế LAB-4 |
| error (mạng) | mất kết nối | banner + nút thử lại; **bộ lọc và vị trí cuộn giữ nguyên** — màn này không có ô nhập nên không mất dữ liệu gõ | thử lại | thiết kế LAB-4 |
| read-only | kỳ `closed`, hoặc chức danh `rule_admin` | ẩn hẳn nút "Tính kỳ này"; bảng vẫn xem và xuất CSV được | xem, xuất CSV | thiết kế LAB-4 |

## Validation & Error Feedback

### A) Client-side

Bộ lọc; màn này không có ô nhập số liệu.

| Field | Type | Required | Constraints | Async Check | Error Message |
|-------|------|----------|-------------|-------------|---------------|
| `filter.period_id` / 対象期 | select | yes | phải tồn tại trong `settlement_periods.id`; mặc định kỳ gần nhất có kết quả | đọc `settlement_periods` | `V-01` (xem Error Catalogue) |
| `filter.participant_id` / 参加者 | select (tìm được) | no | phải tồn tại trong `participants.id`; rỗng = tất cả | không | không có |
| `filter.row_type` / 行種別 | select | no | ∈ `main` (本計算) `delta` (差分); rỗng = cả hai | không | không có |
| `filter.zero_only` / ゼロのみ | checkbox | no | lọc `incentive_amount = 0`; mặc định tắt; tính trên view | không | không có |

### B) Server-side

#### Chạy tính kỳ
- **Endpoint:** `rpc('calculate_incentive', { p_period_id })` *(đề xuất)*
- **Request:** `p_period_id`
- **Success:** `jsonb { row_count, total_amount, rate_version, warnings[] }` → banner xanh, bảng nạp lại
- **Errors:** `P0035` · `P0034` · `P0005` (và cảnh báo `P0038` không chặn)
- **Trigger:** nút "Tính kỳ này" → hộp thoại xác nhận
- **Source:** chưa có code — màn chỉ đặc tả ở LAB-4

#### Xuất CSV
- **Endpoint:** Server Action đọc **cùng `v_incentive_results`** rồi ghi file (`RPT-07`)
- **Request:** cùng bộ lọc
- **Success:** CSV đúng tập dòng đang lọc
- **Errors:** theo Error Catalogue
- **Trigger:** nút "Xuất CSV"
- **Source:** chưa có code — màn chỉ đặc tả ở LAB-4

Tính bắt buộc qua RPC `SECURITY DEFINER`: công thức `BR-INC-01` phải nằm một chỗ ở phía server, và `incentive_results` không có policy ghi.

#### Error Catalogue *(mục bổ sung)*

| Mã | Điều kiện kích hoạt | Thông báo VI | Thông báo JA | Hiện ở đâu | Chặn gửi? |
|---|---|---|---|---|:---:|
| `V-01` | chưa chọn kỳ ở bộ lọc | Chọn kỳ trước khi xem kết quả. | 対象期を選択してください。 | dưới ô lọc | ● |
| `P0035` *(mới, đề xuất)* | bấm "Tính kỳ này" trên kỳ có `status = closed` | Kỳ {period} đã chốt — không tính lại. Điều chỉnh sẽ sinh chênh lệch ở kỳ {next_period}. | 期{period}は締め済みです。調整は次期{next_period}で差分として計上されます。 | banner đầu bảng | ● |
| `P0034` *(mới, đề xuất)* | không có bản 料率 nào `approved` có `effective_from ≤` ngày nghiệp vụ đang tính | Không có phiên bản 料率 hiệu lực cho ngày {business_date}. | 業務日{business_date}に有効な料率版がありません。 | banner đầu bảng | ● |
| `P0038` *(mới, đề xuất)* | kỳ còn ngày nghiệp vụ `open` | Kỳ {period} còn {n} ngày nghiệp vụ chưa chốt — tính ra số tạm. | 期{period}に未締めの業務日が{n}日あります。暫定値になります。 | banner đầu bảng | ✗ — **cảnh báo**, vẫn cho tính |
| `P0005` | chức danh không phải `reconcile` mà gọi RPC tính | Bạn không có quyền chạy tính cho kỳ này. | この期の計算を実行する権限がありません。 | banner đầu bảng | ● |

Rẽ nhánh theo `error.code`; thông báo dựng từ `error.details` (`period`, `next_period`, `business_date`, `n`) qua `mapRpcError` — **không so khớp chuỗi `message`**.

## Interaction Patterns

- **"Tính kỳ này" mở hộp thoại xác nhận chặn luồng, nêu rõ kỳ nào, bản 料率 nào sẽ áp, và việc chạy lại sẽ ghi đè kết quả tạm của kỳ đang mở** — cùng lý do với SCR018: thao tác ghi khó hoàn tác thì phải có một nhịp dừng; thiết kế LAB-4.
- **Bấm một dòng bung chi tiết; bấm `v{n}` mở SCR027 chỉ đọc** — thiết kế LAB-4.
- Không có yêu cầu bàn phím riêng — không thuộc ba màn keyboard-first (SCR007, SCR010, SCR014).

## Accessibility

| Aspect | Status | Notes |
|--------|--------|-------|
| ARIA roles/labels | partial (đặc tả) | banner `role="status"`; badge `main`/`delta` cần chữ, không chỉ màu |
| Keyboard navigation | supported (đặc tả) | input gốc, link `v{n}`, hộp thoại xác nhận |
| Focus management | managed (đặc tả) | banner lỗi tự nhận focus |
| Screen reader compatibility | unknown | chưa có bản dựng để kiểm |

## Conditional Rendering

| Condition | Type | Renders | Hidden | Notes |
|-----------|------|---------|--------|-------|
| role = `reconcile` và kỳ `open` | auth | nút "Tính kỳ này" | — | bypass: RPC `P0005` |
| kỳ `closed` hoặc role = `rule_admin` | auth | bảng + xuất CSV | nút "Tính kỳ này" | ẩn hẳn |
| `row_type = delta` | hardcoded-id | `delta_origin_period`, `delta_reason`, tông cảnh báo | — | |
| `incentive_amount = 0` | hardcoded-id | `zero_reason` | — | |
| kỳ chưa có kết quả | hardcoded-id | thông báo rỗng + nút tính | bảng | |

## Component Variants

Không áp dụng — màn không có component đổi hình dạng theo biến thể; khác biệt theo chức danh/trạng thái đã mô tả ở Conditional Rendering.

## Security Surface

| Guard | Type | Consequence if bypassed |
|-------|------|------------------------|
| `proxy.ts` / `route-policy.ts` thêm `{ prefix: "/incentives", roles: ["reconcile", "rule_admin"] }` *(đề xuất)* | role | role ngoài danh sách mở được màn |
| RLS SELECT trên `incentive_results`; **không có policy ghi** (ADR-002) | permission | đọc/ghi ngoài chức danh; đường ghi duy nhất là RPC `calculate_incentive` |
| RPC `calculate_incentive` | permission | `[UNVERIFIED]` — chưa tồn tại, là yêu cầu thiết kế |

Quyền theo chức danh: `sys_admin` ✗ — quản trị kỹ thuật không đọc số tiền nghiệp vụ · `reconcile` xem + ghi ("Ghi" = chạy tính cho kỳ **đang mở**; không sửa được từng ô) · `rule_admin` *(mới, đề xuất — xem SCR037)* xem — để đối chiếu hệ quả của bản 料率 mình vừa duyệt · `trading` · `receiving` ✗ · `party_admin` *(mới)* ✗.

## Source References

Chưa có code — màn chỉ đặc tả ở LAB-4. Tài liệu nguồn đã đọc:

1. (tài liệu gốc trong repo, không nộp) `sc-25-tien-khuyen-khich.md` — spec 10 mục gốc
2. `lib/auth/route-policy.ts:17-27` — `ROUTE_POLICY` hiện có (chưa có `/incentives`)
3. (tài liệu gốc trong repo, không nộp) `adr.md` — ADR-002

## Source Walkthrough

Chưa có code — màn chỉ đặc tả ở LAB-4.

### Call Hierarchy

```text
(thiết kế) /incentives page -> settlement_periods + v_incentive_results (đọc)
   -> Server Action -> rpc calculate_incentive ; Server Action CSV -> v_incentive_results
```

## Dependencies & Risks *(mục bổ sung)*

- **Cần trước:** `SCR018` (ngày nghiệp vụ đã chốt), `SCR026`/`SCR027` (phải có ít nhất một bản 料率 `approved`), và luồng điều chỉnh sau chốt của `F07` để sinh dòng `delta`.
- **Dữ liệu phải có sẵn:** `settlement_periods` — **bảng này chưa tồn tại trong prototype**, và độ dài một "kỳ" chưa được khách định nghĩa. Xem Unresolved Questions.
- **Dễ hiểu sai #1:** tra 料率 phải theo **`as_of` = ngày nghiệp vụ**, không phải `now()`. Một `now()` lọt vào hàm tính là đủ làm số của kỳ cũ đổi sau mỗi lần đổi 料率, mà không test nào đỏ.
- **Dễ hiểu sai #2:** làm tròn là **`floor`**, không phải `round`. Chênh nhau 1 JPY mỗi dòng; test vector `INC-EDGE-03` của Appendix C bắt đúng chỗ này.
- **Dễ hiểu sai #3:** quá hạn → **`= 0`**, không phải giảm theo số ngày trễ. Không có 猶予期間 ở bất kỳ đâu trong luồng.
- **Dễ hiểu sai #4:** điều chỉnh kỳ đã chốt **không `UPDATE` dòng cũ**. Sinh dòng `delta` ở kỳ sau. Sửa đè là phá `FR-INC-03` và `BR-CLOSE-01` cùng lúc.
- **Dễ hiểu sai #5:** `grant_rate_applied` là **cột lưu**. Ai đó "tối ưu" bằng cách bỏ cột và join động sẽ làm hỏng `F08-02` mà màn hình vẫn trông đúng.
- **Rủi ro nghiệm thu:** định nghĩa **支払期日** chưa có trong tài liệu khách (câu 10 gửi khách). Công thức đúng mà mốc sai thì số vẫn sai, và sai một cách khó phát hiện.

### Unresolved Questions *(mục bổ sung)*

1. **Test vector Appendix C của khách** — `FR-INC-01` ghi *"kết quả phải khớp test vector Appendix C"*, và LAB-1 đã nêu lại ở `US-13`. Hiện **chưa có bộ số thật**, chỉ có mô tả ba case (`INC-EDGE-01` ngưỡng tối thiểu · `INC-EDGE-02` điều chỉnh sau chốt · `INC-EDGE-03` thập phân sau hệ số thuế). **Không verify được công thức nếu không có bộ số đầu vào/đầu ra của khách.** Đây là mục chặn nghiệm thu, cần lấy trước LAB-5.
2. **"Kỳ" dài bao nhiêu?** Tháng, nửa tháng, hay theo lịch chợ? Quyết định này đổi cả `settlement_periods` lẫn nghĩa của "kỳ kế tiếp" ở `F08-06`.
3. **支払期日 tính từ mốc nào?** Câu 10 gửi khách. Hiện thiết kế là **tham số của phiên bản quy chế** (theo `product-a-feature-list.md`), nằm trong payload của bản 料率 — xem `SCR027` field `payment_due_days`.
4. **Có ngưỡng tối thiểu để được hưởng không?** `INC-EDGE-01` nhắc *"đúng ngưỡng tối thiểu"* nhưng không nói ngưỡng là gì. `zero_reason = below_threshold` ở Data Inventory là **đề xuất** dựa trên câu đó.
5. **Chạy tính là thủ công hay theo lịch?** Tài liệu này chọn thủ công cho kỳ đang mở (KISS, và để người chịu trách nhiệm bấm nút). Nếu khách muốn chạy đêm thì thêm job, và cần quyết số liệu tạm có hiển thị khác màu không.
6. **"完納" đo bằng gì?** Thanh toán đủ toàn phần, hay đủ theo từng lần giao? Giao nhiều lần (`F05`) làm câu này không hiển nhiên — RFP cũng để ngỏ *thời điểm phản ánh vào 精算 / 完納奨励金*.
