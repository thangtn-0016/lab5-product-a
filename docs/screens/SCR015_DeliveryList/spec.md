# SCR015_DeliveryList — Screen Spec

> Alias LAB-1: **SC-15** · 引渡し一覧 · Danh sách giao hàng

**Screen**: SCR015: DeliveryList (Danh sách giao hàng)
**Feature**: [F005_DeliveryTracking](../../features/F005_DeliveryTracking/screens.md) — tính năng LAB-1 `F05-02` `F05-03`
**Type**: atomic (danh sách + bộ lọc)
**Route**: `/deliveries` *(đề xuất — xem lưu ý về route hiện có ở Source References)*
**Generated**: 2026-10-06
**Trạng thái**: Thiết kế mới, chưa xây (LAB-4)
**Role**: `receiving` (xem) · `reconcile` (xem)
**User story**: không có US riêng — nguồn tra cứu chung cho `US-07` (ghi từng lần giao và biết còn lại bao nhiêu) và `US-08` (chốt hoàn tất).

## Purpose

Bộ phận giao hàng mở đầu ca để biết hôm nay còn phải giao gì; bộ phận đối chiếu mở trước giờ chốt sổ (08:00–10:00 JST) để biết giao dịch nào sẽ rơi vào `pending_conditions` — nhìn một màn biết **giao dịch nào giao tới đâu**: lũy kế đã giao, số còn lại, số lần đã giao, và giao dịch nào đang **ứ đọng**. Thay cho 14 cuộc gọi điện mỗi ngày cao điểm mà nhật ký nội bộ ghi nhận (F05 của LAB-1).

Bộ lọc "ứ đọng" **dùng chung định nghĩa dữ liệu với báo cáo `RPT-04`** (ràng buộc bắt buộc của LAB-1) — một công thức duy nhất, không phải hai chỗ tự định nghĩa.

## Screen Layout

Trang danh sách trong khung ứng dụng chung: tiêu đề (kèm nút "Ghi lần giao" chỉ cho `receiving`), thanh bộ lọc GET, rồi bảng một dòng = một giao dịch. Server Component, đồng bộ bộ lọc qua URL. Chưa có code nên chưa có breakpoint chốt — (đề xuất) fluid như SCR012.

### Layout Sketch

```
┌──────────────────────────────────────────────────────┐
│ R1: Header ứng dụng + nav (static)                │
├──────────────────────────────────────────────────────┤
│ R2: Tiêu đề            [Ghi lần giao → SCR014]       │
│                         (chỉ receiving)              │
├──────────────────────────────────────────────────────┤
│ R3: Bộ lọc: ngày · mã GD · người mua · tình trạng    │
│              · ☐ chỉ ứ đọng · [Áp dụng]              │
├──────────────────────────────────────────────────────┤
│ R4: Bảng giao dịch (1 dòng = 1 giao dịch)            │
│     - - - khối lỗi + [Tải lại] khi truy vấn throw - -│
└──────────────────────────────────────────────────────┘
```

### Layout Regions

| Region ID | Name | Position | Scrollable | Key Components | Responsive Behavior |
|-----------|------|----------|------------|----------------|---------------------|
| R1 | Header + nav | static | no | khung `(app)` chung | always |
| R2 | Tiêu đề + hành động | static | no | `PageHeader`, nút "Ghi lần giao" | fluid |
| R3 | Bộ lọc | static | no | form GET (tương tự `lot-filter`) | fluid, wrap |
| R4 | Bảng giao dịch | static | yes | `DataTable`, `StatusBadge` | fluid |

## Child Routes

Không áp dụng — màn này là một route đơn, không phải khung chứa route con.

## User Flow

> Phạm vi: tương tác trong màn. Điều hướng liên màn xem [screen-flow](../../generated/screen-flow.md).

### Happy Path

1. `receiving`/`reconcile` mở `/deliveries`.
2. Mặc định lọc theo **ngày hôm nay**, sắp xếp: dòng "Có ngoại lệ" lên đầu, rồi `backlogDays` giảm dần, rồi `business_date` giảm dần.
3. Đổi bộ lọc → "Áp dụng" (GET, giữ trạng thái trên URL để chia sẻ được đường dẫn).
4. Bật "Chỉ hiện ứ đọng" → danh sách thu lại còn các giao dịch còn hàng chưa giao quá ngưỡng ngày.
5. Bấm mã giao dịch → SCR016 (chi tiết từng lần giao của giao dịch đó).
6. `receiving` còn thấy nút "Ghi lần giao" ở góc trên → SCR014; `reconcile` **không** thấy nút này.

### Branches

| Decision point | Condition | Outcome on this screen | Source |
|----------------|-----------|------------------------|--------|
| Bước 3 | không giao dịch nào khớp | bảng rỗng "Không có giao dịch nào khớp bộ lọc." | thiết kế LAB-4 |
| Bước 3 | `buyerCode` không tồn tại | bảng rỗng, không báo lỗi (như SCR012) | thiết kế LAB-4 |
| Bước 6 | role `reconcile` | không có nút "Ghi lần giao" | thiết kế LAB-4 |
| Bước 2 | truy vấn throw | khối lỗi + nút "Tải lại" | thiết kế LAB-4 |

## Data Inventory

> Cột của bảng (mỗi dòng = một giao dịch, không phải một lần giao). Bộ lọc nằm ở `## Validation & Error Feedback`.

| Field | Display Label | Source | Format | Empty Behavior | Cross-ref |
|-------|---------------|--------|--------|----------------|-----------|
| `tradeCode` | Mã giao dịch / 取引番号 | `trades.trade_code` | link → SCR016 | luôn có | trades |
| `businessDate` | Ngày nghiệp vụ / 業務日 | `trades.business_date` | `.toLocaleDateString()`; ngày đã chốt sổ **vẫn hiện bình thường**, thêm badge "Đã chốt sổ" | luôn có | trades |
| `buyer` | Người mua / 買受人 | `participants` theo `trades.buyer_participant_id` | `{name} ({code})` | "" nếu không tra được | participants |
| `commodity` | Mặt hàng / 品目 | `commodities` theo `lots.commodity_id` | tên theo cookie `locale` | "" | commodities |
| `quantity` | Tổng số lượng / 取引総数量 | `v_trade_delivery_status.quantity` | number + đơn vị | luôn có | v_trade_delivery_status |
| `deliveredQty` | Lũy kế đã giao / 引渡し済み（累積） | `v_trade_delivery_status.delivered_qty` | `= Σ deliveries.quantity WHERE status IN ('completed','exception')` — **KHÔNG phải cột lưu sẵn**. Đây là công thức của view hiện có (có lỗi B-01: chưa trừ `returned`/`partial_cancel`); hành vi đã chốt ở PREMISE G3 là view phải *trừ* hai loại này | 0 | v_trade_delivery_status |
| `remaining` | Còn lại / 残数量 | `v_trade_delivery_status.remaining` | `= quantity − delivered_qty`, tính mỗi lần đọc (ADR-003) | `= quantity` | v_trade_delivery_status |
| `deliveryCount` | Số lần đã giao / 引渡し回数 | `v_trade_delivery_status.delivery_count` | number; `≥ 3` ⇒ nền hổ phách (ngưỡng cảnh báo giả định, như SCR014) | 0 | v_trade_delivery_status |
| `progress` | Tình trạng / 状況 | `v_delivery_overview.progress` *(view đề xuất)* | badge: `delivery_count = 0` → "Chưa giao" (neutral) · `remaining > 0 ∧ count > 0` → "Đang giao" (warning) · `remaining = 0` → "Đã giao đủ" (positive) · có ít nhất 1 lần giao `status='exception'` → "Có ngoại lệ" (danger, **ưu tiên cao nhất, đè 3 nhãn kia**) | — | v_delivery_overview |
| `lastDeliveredAt` | Lần giao gần nhất / 直近の引渡し | `v_delivery_overview.last_delivered_at` | `max(deliveries.delivered_at)` | "—" | v_delivery_overview |
| `backlogDays` | Số ngày ứ đọng / 滞留日数 | tính ở tầng view | `= current_date − business_date` khi `remaining > 0`, ngược lại để trống; `≥ 2` ⇒ chữ đỏ | "—" | v_delivery_overview |

### Read Calls *(mục bổ sung)*

| Thao tác | Cách gọi | Tham số | Trả về |
|---|---|---|---|
| Tra mã người mua (khi lọc `buyerCode`) | `select` từ `participants` qua `findParticipantByCode()` | `code` | `participant.id` hoặc `null` |
| Đọc danh sách | `select` từ `v_delivery_overview` *(view mới, đề xuất)* kèm embed `participants` + `commodities` | `businessDate?`, `tradeCode?`, `buyer_participant_id?`, `progress?`, `backlog?` | mảng dòng theo bảng Data Inventory |

**Vì sao cần view mới thay vì ghép ở tầng ứng dụng:** hai cột `progress` và `last_delivered_at` cần phép gộp "tồn tại ít nhất một lần giao `exception`" và `max(delivered_at)` — PostgREST không diễn đạt được gọn, và nếu tính ở Next.js thì công thức "ứ đọng" sẽ nằm ở hai nơi (màn này và `RPT-04`), trái ràng buộc dùng chung dữ liệu của LAB-1.

`v_delivery_overview` *(đề xuất)* dựng trên `trades` ⨝ `v_trade_delivery_status` ⨝ `lots` ⨝ `deliveries` (gộp), **bắt buộc `security_invoker = true`** — bài học ADR-003: thiếu cờ này thì RLS bảng gốc không áp dụng qua view.

## UI States

| State | Trigger | Visual Behavior | User Action Available | Source |
|-------|---------|----------------|-----------------------|--------|
| loading | Server Component — không có trạng thái tải riêng | nếu thêm `loading.tsx` thì dùng skeleton 10 dòng | none | thiết kế LAB-4 |
| empty | không có giao dịch khớp bộ lọc | "Không có giao dịch nào khớp bộ lọc." / 条件に一致する取引がありません。 | đổi bộ lọc | thiết kế LAB-4 |
| error | truy vấn `throw` | khối lỗi kèm nút "Tải lại" (khác SCR012, màn đó rơi xuống trang lỗi mặc định — cải thiện có chủ ý) | Tải lại | thiết kế LAB-4 |
| saving | không áp dụng (không ghi trên màn này) | — | — | — |
| success | không áp dụng | — | — | — |
| read-only | luôn chỉ đọc | ngày đã chốt sổ **vẫn hiện bình thường**, thêm badge "Đã chốt sổ" ở cột ngày nghiệp vụ | xem | thiết kế LAB-4 |

Lỗi nghiệp vụ: không áp dụng.

## Validation & Error Feedback

### A) Client-side

Bộ lọc, đồng bộ qua `searchParams`; không trường nào bắt buộc.

| Field | Type | Required | Constraints | Async Check | Error Message |
|-------|------|----------|-------------|-------------|---------------|
| `businessDate` / 業務日 | `input type="date"` | no | rỗng = không lọc; mặc định hôm nay; `searchParams.businessDate` → `trades.business_date` | không | không có — không báo lỗi |
| `tradeCode` / 取引番号 | text | no | khớp **chứa** (`ilike %…%`); không khớp dòng nào ⇒ bảng rỗng, **không phải lỗi**; `searchParams.tradeCode` | không | không có |
| `buyerCode` / 買受人コード | text, placeholder "VD: PTA-01" | no | mã không tồn tại trong `participants.code` ⇒ bảng rỗng, không báo lỗi (giữ đúng hành vi SCR012); `searchParams.buyerCode` → `findParticipantByCode()` | `findParticipantByCode()` | không có |
| `progress` / 引渡し状況 | select | no | ∈ `chua_giao` `dang_giao` `da_giao_du` `co_ngoai_le` + "Tất cả"; giá trị lạ bị bỏ qua lặng lẽ; mặc định "Tất cả" | không | không có |
| `backlogOnly` / 滞留のみ | checkbox | no | bật ⇒ thêm `remaining > 0 AND business_date ≤ current_date − 2` *(ngưỡng 2 ngày nghiệp vụ là **đề xuất**, xem Unresolved Questions)*; mặc định tắt; `searchParams.backlog` | không | không có |

### B) Server-side

Không áp dụng — màn chỉ đọc, không có submit-style action.

#### Error Catalogue *(mục bổ sung)*

Màn này **không có form ghi** — chỉ bộ lọc GET. Không có mã lỗi mới.

| Mã | Điều kiện kích hoạt | Thông báo VI | Thông báo JA | Hiện ở đâu | Chặn gửi? |
|---|---|---|---|---|:---:|
| — | `tradeCode`/`buyerCode` không khớp bản ghi nào | (dùng chung câu rỗng) "Không có giao dịch nào khớp bộ lọc." | 条件に一致する取引がありません。 | vùng bảng (`EmptyState`) | không áp dụng — GET luôn thành công |
| (network) | truy vấn Supabase `throw` | Không tải được danh sách giao hàng. Kiểm tra kết nối rồi tải lại trang. | 引渡し一覧を読み込めませんでした。接続を確認して再読み込みしてください。 | thay chỗ bảng, kèm nút "Tải lại" | không áp dụng |

## Interaction Patterns

- **Toàn bộ bộ lọc là input gốc, Tab theo thứ tự DOM; mã giao dịch ở cột 1 là link nên tới được bằng Tab mà không cần chuột** — thiết kế LAB-4. Không có yêu cầu bàn phím riêng.
- **"Áp dụng" gửi GET và giữ bộ lọc trên URL để chia sẻ đường dẫn** — thiết kế LAB-4.
- **Bấm mã giao dịch mở SCR016** — thiết kế LAB-4.

## Accessibility

| Aspect | Status | Notes |
|--------|--------|-------|
| ARIA roles/labels | unknown | chưa có bản dựng; badge tình trạng cần chữ, không chỉ màu (đề xuất) |
| Keyboard navigation | supported (đặc tả) | input gốc + link; không phím tắt riêng |
| Focus management | unmanaged | không có hộp thoại |
| Screen reader compatibility | unknown | chưa kiểm |

## Conditional Rendering

| Condition | Type | Renders | Hidden | Notes |
|-----------|------|---------|--------|-------|
| role = `receiving` | auth | nút "Ghi lần giao" → SCR014 | — | `reconcile` không thấy; ghi thật vẫn bị `record_delivery` chặn (`P0005`) |
| ngày nghiệp vụ đã `closed` | hardcoded-id | badge "Đã chốt sổ" | — | không khoá đọc |
| `deliveryCount ≥ 3` | hardcoded-id | nền hổ phách | — | ngưỡng giả định [NEEDS_DOMAIN_CONFIRMATION] |
| `backlogDays ≥ 2` | hardcoded-id | chữ đỏ | — | ngưỡng đề xuất [NEEDS_DOMAIN_CONFIRMATION] |
| tập progress có `exception` | hardcoded-id | badge "Có ngoại lệ" đè 3 nhãn còn lại | — | |

## Component Variants

Không áp dụng — màn không có component đổi hình dạng theo biến thể; khác biệt theo chức danh/trạng thái đã mô tả ở Conditional Rendering.

## Security Surface

| Guard | Type | Consequence if bypassed |
|-------|------|------------------------|
| `ROUTE_POLICY` — mục khớp chính xác `/deliveries/new` → `["receiving"]` xét **trước** mục prefix `/deliveries` → `["receiving","reconcile"]` | role | nới prefix mà không thêm mục chính xác sẽ **vô tình mở màn ghi SCR014** cho `reconcile`; tầng ghi thật vẫn là RPC `record_delivery` (`P0005`) nhưng không được dựa vào đó để biện minh cho route sai |
| RLS `deliveries` SELECT đã có `receiving` + `trading` + `reconcile` | permission | không cần migration RLS cho màn này |
| `v_delivery_overview` bắt buộc `security_invoker = true` | permission | thiếu cờ ⇒ RLS bảng gốc không áp dụng qua view |

Quyền theo chức danh: `sys_admin` ✗ · `receiving` xem · `trading` ✗ (LAB-1 không liệt kê, dù RLS cho SELECT) · `reconcile` xem. Chức danh "Giao hàng" (配送部門) của LAB-1 đã chốt gộp vào `receiving` (PREMISE C-10, như SCR014). 5 role đề xuất (`party_admin`, `rule_admin`, `ops_admin`, `market_manager`, `internal_audit`) ✗; `can_approve` không liên quan (màn chỉ đọc).

## Source References

Chưa có code — màn chỉ đặc tả ở LAB-4. Lưu ý đã kiểm trong code LAB-3: route `/deliveries` **có** nhưng chỉ chuyển hướng sang `/deliveries/new` (SCR014), không phải danh sách.

1. `app/(app)/deliveries/page.tsx:11-15` — `/deliveries` hiện redirect sang `/deliveries/new`; màn này sẽ thay thế trang đó
2. `lib/auth/route-policy.ts:17-27` — `ROUTE_POLICY` hiện chỉ có `{ prefix: "/deliveries", roles: ["receiving"] }` (dòng 23)
3. `supabase/migrations/0006_derived_views.sql:27` — `v_trade_delivery_status`
4. (tài liệu gốc trong repo, không nộp) `sc-15-danh-sach-giao-hang.md` — spec 10 mục gốc

## Source Walkthrough

Chưa có code — màn chỉ đặc tả ở LAB-4.

### Call Hierarchy

```text
(thiết kế) /deliveries page -> v_delivery_overview (đề xuất) + participants (lọc buyerCode)
```

## Dependencies & Risks *(mục bổ sung)*

- Phụ thuộc SCR010/SCR011 (có giao dịch) và SCR014 (có lần giao) chạy trước.
- Phụ thuộc `v_delivery_overview` được tạo **trước** khi làm màn; nếu LAB-5 xếp màn này trước migration view thì cả màn không có dữ liệu.
- **Rủi ro route (Security Surface)** — nới prefix `/deliveries` cho `reconcile` mà quên tách `/deliveries/new` là lỗi bảo mật thật, không phải lỗi hiển thị.
- **Rủi ro hiểu sai:** mỗi dòng là **một giao dịch**, không phải một lần giao. 4 trạng thái của `delivery_status` (`pending`/`in_transit`/`completed`/`exception`) thuộc **từng lần giao** và chỉ hiện ở SCR016; cột "Tình trạng" ở đây là nhãn **suy ra**, không phải enum trong database. Đặt tên cột trùng với enum sẽ khiến LAB-5 tách task sai.
- Lần giao `exception` vẫn tính vào `delivered_qty` (số đã thực giao thật, (tài liệu gốc trong repo, không nộp) `db-schema.md` §3) — nên một giao dịch có thể vừa "Có ngoại lệ" vừa `remaining = 0`. Badge ngoại lệ ưu tiên hơn.
- Không phân trang ở bản đầu: RFP nêu 700–1.200 giao dịch/ngày, bộ lọc mặc định theo ngày giữ kích thước trong tầm; bỏ lọc ngày thì phải có giới hạn — xem Unresolved Questions.

### Unresolved Questions *(mục bổ sung)*

- **Ngưỡng "ứ đọng" là bao nhiêu ngày?** Tài liệu này đề xuất `≥ 2 ngày nghiệp vụ`, không có nguồn trong RFP. Phải chốt cùng `RPT-04`, vì hai nơi dùng chung một công thức — hai ngưỡng khác nhau là lỗi nghiệm thu.
- Ngày ứ đọng đếm theo **ngày nghiệp vụ** (bỏ qua ngày chợ nghỉ, SCR021) hay ngày lịch? Tài liệu này tạm đếm ngày lịch; nếu khách yêu cầu ngày nghiệp vụ thì công thức phải tra `business_days`.
- `trading` có cần xem màn này không? LAB-1 nói không, nhưng RLS đã cho phép đọc — giữ nguyên thế lệch có chủ ý như SCR012, hay thống nhất lại cả hai tầng?
- Ngưỡng phân trang và giới hạn mặc định khi bỏ lọc ngày.
- Nhãn "Đã giao đủ" (`remaining = 0`) khác với "đã hoàn tất giao hàng" ở cấp giao dịch (`trades.delivery_completed_at`, thao tác ở `SCR016`, PREMISE G3): một giao dịch giao đủ số lượng vẫn có thể chưa được chốt hoàn tất. Cần chốt cột/nhãn riêng cho "Đã hoàn tất" để người dùng không nhầm hai khái niệm.
