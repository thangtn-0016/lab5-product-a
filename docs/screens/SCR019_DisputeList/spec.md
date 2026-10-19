# SCR019_DisputeList — Screen Spec

> Alias LAB-1: **SC-19** · 係争一覧 · Danh sách tranh chấp

**Screen**: SCR019: DisputeList (Danh sách tranh chấp)
**Feature**: [F006_ReconciliationAndClosing](../../features/F006_ReconciliationAndClosing/screens.md) — tính năng LAB-1 `F06-04`
**Type**: atomic (danh sách + bộ lọc + hai số tổng)
**Route**: `/disputes` *(đề xuất)*
**Generated**: 2026-10-06
**Trạng thái**: Thiết kế mới, chưa xây (LAB-4)
**Role**: `reconcile` (xem) · `receiving` (xem; gộp 配送部門, PREMISE C-10) · `trading` (xem, đề xuất) · `internal_audit` (xem, đề xuất — role đề xuất, chưa có trong `app_role`)
**User story**: `US-18` (*"xác nhận giao hàng khi có tranh chấp"*) — LAB-1 gán đích danh `SC-17` `SC-19` `SC-20` cho US này. Requirement: `FR-SETTLE-03`.

## Purpose

Màn `reconcile` mở đầu ca sau 10:00 JST để biết hôm nay phải đuổi việc nào: danh sách **mọi vụ tranh chấp** phát sinh từ đối chiếu và ngoại lệ giao hàng, lọc theo ba tiêu chí LAB-1 nêu đích danh — **trạng thái**, **người xử lý**, **quá hạn xử lý**.

`FR-SETTLE-03` yêu cầu *"danh sách tranh chấp đang mở + ngày dự kiến xử lý + lịch sử cập nhật"* — màn này gánh hai vế đầu; vế lịch sử cập nhật nằm ở SCR020.

**Vì sao "quá hạn" là bộ lọc chứ không phải một cột trang trí.** `RPT-09` của khách là *"Tổng hợp tranh chấp + SLA xử lý"*. SLA chỉ đo được khi "quá hạn" là một **vị từ tính được**, không phải cảm nhận. Mục Data Inventory định nghĩa nó bằng một công thức duy nhất, dùng chung giữa màn này và báo cáo.

## Screen Layout

Trang danh sách trong khung ứng dụng chung: tiêu đề, dải hai số tổng (Đang mở / Quá hạn, không chịu bộ lọc), thanh bộ lọc, rồi bảng. Chưa có code nên chưa có breakpoint chốt — (đề xuất) fluid như SCR012.

### Layout Sketch

```
┌──────────────────────────────────────────────────────┐
│ R1: Header ứng dụng + nav (static)                │
├──────────────────────────────────────────────────────┤
│ R2: Tiêu đề · [Đang mở: N] [Quá hạn: M] (không lọc)  │
├──────────────────────────────────────────────────────┤
│ R3: Bộ lọc: trạng thái · người xử lý · ☐ chỉ quá hạn │
│              · 業務日 · [Áp dụng] [Xoá lọc → mặc định]│
├──────────────────────────────────────────────────────┤
│ R4: Bảng tranh chấp (quá hạn lên đầu)                │
│     - - - rỗng (có lọc) + [Xoá lọc] - - -            │
└──────────────────────────────────────────────────────┘
```

### Layout Regions

| Region ID | Name | Position | Scrollable | Key Components | Responsive Behavior |
|-----------|------|----------|------------|----------------|---------------------|
| R1 | Header + nav | static | no | khung `(app)` chung | always |
| R2 | Tiêu đề + số tổng | static | no | `PageHeader`, hai ô số | fluid |
| R3 | Bộ lọc | static | no | form GET | fluid, wrap |
| R4 | Bảng tranh chấp | static | yes | `DataTable`, `StatusBadge` | fluid |

## Child Routes

Không áp dụng — màn này là một route đơn, không phải khung chứa route con.

## User Flow

> Phạm vi: tương tác trong màn. Điều hướng liên màn xem [screen-flow](../../generated/screen-flow.md).

### Happy Path

1. `reconcile` mở `/disputes` sau 10:00 JST → mặc định thấy tranh chấp **đang mở**, quá hạn lên đầu.
2. Hai con số tổng (đang mở / quá hạn) hiện trên đầu, **không đổi theo bộ lọc**.
3. Bật "Chỉ quá hạn" → danh sách thu về các vụ vi phạm `due_date`.
4. Chọn "Chưa có người xử lý" ở bộ lọc người xử lý → thấy các vụ chưa ai nhận; đây là nhánh dùng hằng ngày, không phải trường hợp hiếm.
5. Bấm một dòng → SCR020 chi tiết tranh chấp.
6. "Xoá lọc" → về `/disputes` không query string, tức là **quay về mặc định "đang mở"**, không phải "tất cả" — nói rõ trên nhãn nút để tránh hiểu nhầm.
7. Không có đường tạo tranh chấp từ màn này: tranh chấp sinh từ ngoại lệ giao hàng (SCR017) hoặc từ đối chiếu (SCR018). Xem Unresolved Questions.

### Branches

| Decision point | Condition | Outcome on this screen | Source |
|----------------|-----------|------------------------|--------|
| Bước 1 | chưa có tranh chấp nào trong hệ thống | "Chưa có tranh chấp nào." | thiết kế LAB-4 |
| Bước 3–4 | lọc không khớp | thông báo riêng kèm nút "Xoá lọc" | thiết kế LAB-4 |
| Bước 4 | uuid người xử lý không tồn tại | trả rỗng, không báo lỗi | thiết kế LAB-4 |
| Bước 1 | truy vấn danh sách lỗi | banner đỏ + thử lại, bộ lọc trên URL giữ nguyên | thiết kế LAB-4 |

## Data Inventory

> Khối danh sách — mỗi dòng là một `disputes`. Bộ lọc nằm ở `## Validation & Error Feedback`.

| Field | Display Label | Source | Format | Empty Behavior | Cross-ref |
|-------|---------------|--------|--------|----------------|-----------|
| `dispute_code` | Mã tranh chấp / 係争番号 | `disputes.dispute_code` | duy nhất, server cấp; dạng `DSP-{yyyymmdd}-{seq}` *(đề xuất)* | luôn có | disputes *(mới, đề xuất)* |
| `status` | Trạng thái / 状態 | `disputes.status` | badge ∈ `open` `investigating` `waiting_party` `resolved` `closed` *(đề xuất — máy trạng thái ở SCR020)* | luôn có | disputes |
| `cause_category` | Nhóm nguyên nhân / 原因区分 | `disputes.cause_category` | badge ∈ `short_delivery` `over_delivery` `return` `partial_cancel` `price` `other` — bốn giá trị đầu khớp **4 loại ngoại lệ giao hàng** của `US-18` | luôn có | disputes |
| `business_date` | 業務日 | `disputes.business_date` | date; ngày nghiệp vụ của giao dịch gốc, **không phải ngày lập tranh chấp** | luôn có | disputes |
| `trade_code` | Giao dịch / 取引番号 | `disputes.trade_id` → `trades(trade_code)` | link, phải tồn tại trong `trades.trade_code`; bấm sang SCR012/SCR013 | luôn có | trades |
| `counterparty` | Người tham gia / 参加者 | `disputes.participant_id` → `participants(code, name)` | `{tên} ({mã})`; "" cho cả hai phần nếu embed không khớp — cùng quy ước SCR012 | "" | participants |
| `amount_in_dispute` | Số tiền tranh chấp / 係争金額 | `disputes.amount_in_dispute` | `≥ 0`, `.toLocaleString()`; rỗng với tranh chấp số lượng thuần | "—" | disputes |
| `assignee` | Người xử lý / 担当者 | `disputes.assignee_id` → `app_users.full_name` | text; rỗng hợp lệ khi `status = open` (chưa ai nhận) | "—" | app_users |
| `due_date` | Ngày dự kiến xử lý / 処理予定日 | `disputes.due_date` | date; `FR-SETTLE-03` nêu đích danh; `≥ business_date` | luôn có | disputes |
| `days_overdue` | Số ngày quá hạn / 超過日数 | `v_disputes_overdue.days_overdue` | **= số 業務日 từ `due_date` tới 業務日 hiện tại**, bỏ qua 休市日 (SCR021); `≤ 0` ⇒ chưa quá hạn | "—" | v_disputes_overdue *(đề xuất)* |
| `is_overdue` | Quá hạn / 期限超過 | `v_disputes_overdue.is_overdue` | badge đỏ; **= `status ∉ {resolved, closed}` AND `days_overdue > 0`** — một công thức duy nhất, dùng chung với `RPT-09` | `false` | v_disputes_overdue |
| `last_update_at` | Cập nhật gần nhất / 最終更新日時 | `v_disputes_overdue.last_update_at` | datetime JST; = `max(dispute_events.occurred_at)` | luôn có | v_disputes_overdue |
| `open_count` / `overdue_count` | Đang mở / Quá hạn / 未解決 / 期限超過 | `v_disputes_summary` | number; **không chịu bộ lọc** — luôn là tổng, để con số trên màn khớp con số trong `RPT-09` | luôn có | v_disputes_summary *(đề xuất)* |

Sắp xếp mặc định: `is_overdue = true` lên đầu, rồi `due_date` **tăng dần** — sắp hết hạn trước.

### Read Calls *(mục bổ sung)*

| Thao tác | Cách gọi | Tham số | Trả về |
|---|---|---|---|
| Đọc danh sách | `select` từ `v_disputes_overdue` (`security_invoker`) kèm embed PostgREST `trades(trade_code)` + `participants(code, name)` | `status[]?`, `assignee?`, `overdue?`, `businessDate?` | mảng `DisputeRow` |
| Đọc hai con số tổng | `select` từ `v_disputes_summary` | — | `{ open_count, overdue_count }` |
| Đọc danh sách người xử lý cho bộ lọc | `select` từ `app_users` lọc `role = 'reconcile'` **hợp** mọi user đang là `assignee_id` của ≥ 1 dispute; người `status = 'suspended'` vẫn có mặt, gắn nhãn "đã nghỉ" | — | mảng `{id, full_name, suspended}` |

Select thẳng, không RPC — màn chỉ hiển thị. Embed PostgREST dùng được vì `disputes.trade_id` và `disputes.participant_id` trỏ **hai bảng khác nhau**, không nhập nhằng tên ràng buộc (cùng lý do với SCR012, khác SCR004).

**`is_overdue` và `days_overdue` tính trong view, không tính ở client.** Lý do: công thức phải bỏ qua 休市日 (SCR021), và một bản sao công thức ở TypeScript sẽ lệch khỏi `RPT-09` ngay lần đầu lịch nghỉ thay đổi.

## UI States

| State | Trigger | Visual Behavior | User Action Available | Source |
|-------|---------|----------------|-----------------------|--------|
| empty (thật) | chưa có tranh chấp nào trong hệ thống | "Chưa có tranh chấp nào." / 係争はまだありません。 | — | thiết kế LAB-4 |
| empty (có lọc) | lọc không khớp | thông báo riêng (Error Catalogue) **kèm nút "Xoá lọc"** — phân biệt với rỗng thật | Xoá lọc | thiết kế LAB-4 |
| loading | mở trang / đổi bộ lọc | khung xương 5 dòng | none | thiết kế LAB-4 |
| error (mạng) | truy vấn danh sách lỗi | banner đỏ + nút thử lại; **bộ lọc trên URL giữ nguyên** nên thử lại không mất điều kiện đã chọn | thử lại | thiết kế LAB-4 |
| saving / success / error nghiệp vụ | không áp dụng — không ghi trên màn này | — | — | — |
| read-only | luôn luôn | — | xem | thiết kế LAB-4 |

## Validation & Error Feedback

### A) Client-side

Bộ lọc, đồng bộ qua `searchParams`; không trường nào bắt buộc.

| Field | Type | Required | Constraints | Async Check | Error Message |
|-------|------|----------|-------------|-------------|---------------|
| bộ lọc `status` / 状態 | multi-select, 5 lựa chọn | no | giá trị ngoài enum bị bỏ qua (cùng kiểu `parseStatus()` ở SCR012); mặc định **`open` + `investigating` + `waiting_party`** *(đề xuất — "đang mở" là mặc định, theo `FR-SETTLE-03`)*; `searchParams.status` | không | không có |
| bộ lọc `assignee` / 担当者 | select người dùng + "Chưa có người xử lý" + "Tất cả" | no | `unassigned` ⇒ `assignee_id IS NULL`; uuid không tồn tại ⇒ **trả rỗng, không báo lỗi** (cùng quy ước `buyerCode` ở SCR012); `searchParams.assignee` | đọc `app_users` | không có |
| bộ lọc `overdueOnly` / 期限超過のみ | checkbox | no | bật ⇒ thêm `is_overdue = true`; mặc định tắt; `searchParams.overdue` | không | không có |
| bộ lọc `businessDate` / 業務日 | `input type="date"` | no | rỗng = không lọc; không ở tương lai so với 業務日 đang mở (`V-01`); kiểm ở client khi bấm "Áp dụng" **và** khi parse `searchParams.businessDate` từ URL (giá trị tương lai trên URL bị bỏ qua kèm thông báo `V-01`) | không | `V-01` — Ngày nghiệp vụ không được ở tương lai. |

### B) Server-side

Không áp dụng — màn chỉ đọc, không có submit-style action.

#### Error Catalogue *(mục bổ sung)*

Không có form ghi trên màn này. Hai hành vi bộ lọc cần nói rõ vì chúng **không phải lỗi**:

| Mã | Điều kiện kích hoạt | Thông báo VI | Thông báo JA | Hiện ở đâu | Chặn gửi? |
|---|---|---|---|---|:---:|
| — | bộ lọc không khớp dòng nào | Không có tranh chấp nào khớp bộ lọc. | 条件に一致する係争がありません。 | vùng bảng (`EmptyState`) + nút "Xoá lọc" | không áp dụng — GET luôn "thành công" |
| — | `assignee` trỏ tới người đã nghỉ (`app_users.status = 'suspended'`) | (không có thông báo riêng — vẫn lọc bình thường và vẫn trả dòng) | | | không áp dụng |
| `V-01` | `businessDate` ở tương lai so với 業務日 đang mở | Ngày nghiệp vụ không được ở tương lai. | 業務日に未来日は指定できません。 | dưới ô | ● |

Dòng thứ hai là quyết định có chủ ý: tranh chấp **không được biến mất** khỏi danh sách chỉ vì người xử lý rời tổ chức. Nếu lọc theo người đã nghỉ mà trả rỗng thì một vụ đang mở trở nên vô hình — đúng thứ `F10` gọi là *mất im lặng*.

## Interaction Patterns

- **Không có yêu cầu bàn phím riêng** — không thuộc ba màn keyboard-first (SCR007, SCR010, SCR014); bộ lọc dùng input gốc, Tab theo thứ tự DOM — thiết kế LAB-4.
- **"Xoá lọc" về `/disputes` không query string = mặc định "đang mở"** (không phải "tất cả"); nhãn nút nói rõ — thiết kế LAB-4.
- **Bấm một dòng mở SCR020** — thiết kế LAB-4.

## Accessibility

| Aspect | Status | Notes |
|--------|--------|-------|
| ARIA roles/labels | unknown | chưa có bản dựng; badge quá hạn cần chữ, không chỉ màu đỏ (đề xuất) |
| Keyboard navigation | supported (đặc tả) | input gốc + dòng bấm được bằng link |
| Focus management | unmanaged | không có hộp thoại |
| Screen reader compatibility | unknown | chưa kiểm |

## Conditional Rendering

| Condition | Type | Renders | Hidden | Notes |
|-----------|------|---------|--------|-------|
| `is_overdue = true` | hardcoded-id | badge đỏ + dòng lên đầu | — | công thức dùng chung `RPT-09` |
| `assignee_id IS NULL` | hardcoded-id | "—" | — | hợp lệ khi `status = open` |
| lọc không khớp | hardcoded-id | thông báo + nút "Xoá lọc" | bảng | khác rỗng thật |
| role ∈ `reconcile` `receiving` `trading` `internal_audit` | auth | màn | — | role khác bị `/403` (đề xuất `ROUTE_POLICY`) |

## Component Variants

Không áp dụng — màn không có component đổi hình dạng theo biến thể; khác biệt theo chức danh/trạng thái đã mô tả ở Conditional Rendering.

## Security Surface

| Guard | Type | Consequence if bypassed |
|-------|------|------------------------|
| `route-policy.ts` thêm `{ prefix: "/disputes", roles: ["reconcile","receiving","trading","internal_audit"] }` *(đề xuất)* | role | role ngoài danh sách mở được màn |
| RLS `disputes` SELECT theo đúng bốn chức danh đó *(đề xuất)* | permission | đọc ngoài chức danh. Hai tầng **cố ý trùng nhau** ở màn này, khác SCR012 nơi chúng lệch có chủ ý |
| `v_disputes_overdue`, `v_disputes_summary` phải `security_invoker = true` | permission | `[UNVERIFIED]` — view chưa tồn tại; thiếu cờ thì RLS bảng gốc không áp dụng |

Quyền theo chức danh (đủ 9 role, PREMISE C-09):

| Role | Quyền | Ghi chú |
|---|:---:|---|
| `reconcile` | xem | tạo/cập nhật ở SCR020/SCR017, không có form ghi ở đây |
| `receiving` | xem | gánh cả 配送部門 (C-10); `US-18` xuất phát từ bộ phận giao hàng |
| `trading` | xem | **đề xuất**: giao dịch của mình bị tranh chấp thì cần thấy; nếu khách không đồng ý thì bỏ dòng này, không ảnh hưởng gì khác |
| `internal_audit` | xem | **đề xuất**: chỉ đọc, phục vụ kiểm toán SLA |
| `sys_admin` | ✗ | |
| `party_admin` | ✗ | |
| `rule_admin` | ✗ | |
| `ops_admin` | ✗ | |
| `market_manager` | ✗ | **đề xuất**: Ban quản lý đọc tổng hợp qua `RPT-09` (SCR029), không cần danh sách vụ |

Cờ `can_approve` không dùng ở màn này (không có thao tác duyệt).

## Source References

Chưa có code — màn chỉ đặc tả ở LAB-4. Tài liệu nguồn đã đọc:

1. (tài liệu gốc trong repo, không nộp) `sc-19-danh-sach-tranh-chap.md` — spec 10 mục gốc
2. `lib/auth/route-policy.ts:17-27` — `ROUTE_POLICY` hiện có (chưa có `/disputes`)
3. (tài liệu gốc trong repo, không nộp) `database-diagram.md` — mô hình dữ liệu (bảng `disputes` là đề xuất)

## Source Walkthrough

Chưa có code — màn chỉ đặc tả ở LAB-4.

### Call Hierarchy

```text
(thiết kế) /disputes page -> v_disputes_overdue (+ trades, participants) + v_disputes_summary + app_users
```

## Dependencies & Risks *(mục bổ sung)*

- **Cần trước:** `SCR021` — `days_overdue` đếm theo **業務日**, nên không có lịch nghỉ chợ thì con số quá hạn sai ở mọi tuần có 休市日. Đây là phụ thuộc thật, không phải phụ thuộc hình thức.
- **Cần trước:** `SCR017` (ngoại lệ giao hàng) và `SCR018` (đối chiếu) — hai nguồn sinh tranh chấp. Không có chúng thì màn này luôn rỗng.
- **Suy luận có đánh dấu:** LAB-1 chỉ cho **một dòng** mô tả SCR019 (*"lọc theo trạng thái, người xử lý, quá hạn xử lý"*). Năm giá trị `status`, sáu `cause_category`, quy tắc đếm quá hạn theo 業務日 và hai con số tổng đều là **đề xuất của tài liệu này**, suy từ `FR-SETTLE-03` và `RPT-09`. Phần bắt buộc theo khách chỉ gồm: ba bộ lọc, `due_date`, `assignee`, `status`, nguyên nhân.
- **Dễ hiểu sai #1:** `business_date` ở cột `business_date` là ngày nghiệp vụ **của giao dịch gốc**, không phải ngày lập tranh chấp. Lọc nhầm hai thứ này làm mất các vụ lập muộn — mà vụ lập muộn chính là vụ dễ quá hạn nhất.
- **Dễ hiểu sai #2:** `is_overdue` **loại trừ** `resolved`/`closed`. Một vụ đóng muộn không còn là "đang quá hạn"; nó là dữ liệu cho `RPT-09`, không phải việc tồn trên màn.
- **Dễ hiểu sai #3:** bộ lọc mặc định **không** là "tất cả". Người đọc code thấy `searchParams` rỗng sẽ tưởng không lọc gì, trong khi màn đang lọc ba trạng thái.
- **Rủi ro khối lượng:** không phân trang. RFP nêu 700–1.200 giao dịch/ngày; nếu tỷ lệ tranh chấp cao thì danh sách "tất cả" theo nhiều năm sẽ tải toàn bộ — cùng nợ kỹ thuật đã ghi ở `SCR012` Unresolved Questions.

### Unresolved Questions *(mục bổ sung)*

1. **Ai tạo tranh chấp, và từ màn nào?** `US-18` mô tả ngoại lệ giao hàng ở `SCR017`, nhưng không nói rõ ngoại lệ **tự động** thành tranh chấp hay cần người lập. Hai hướng cho ra hai thiết kế rất khác: tự động thì cần quy tắc sinh; thủ công thì `SCR019` cần nút "Lập tranh chấp".
2. **`due_date` do ai đặt, theo SLA nào?** `RPT-09` nhắc "SLA xử lý" nhưng khách chưa đưa số ngày. Chưa chốt thì `due_date` là ô nhập tay và "quá hạn" không so được giữa các vụ.
3. **Tranh chấp có chặn chốt sổ `業務日` không?** `SCR018` hiện coi "điều kiện chưa đạt" là **cảnh báo, không chặn** (ADR-005 chưa chốt). Nếu khách muốn tranh chấp đang mở chặn chốt sổ thì đó là thay đổi ở `SCR018`, không ở màn này — nhưng phải quyết cùng lúc.
4. **`internal_audit` và `market_manager` có xem màn này không?** Security Surface để `internal_audit` xem (chỉ đọc) và `market_manager` ✗ — cả hai là **đề xuất** của bài này, cần khách xác nhận. (Câu "`delivery` có thấy không" đã đóng: 配送部門 gộp vào `receiving`, PREMISE C-10, nên `receiving` xem.)
5. **Tranh chấp theo chợ?** Nếu ADR-007 thành hiện thực thì `disputes` cần `market_id` và bộ lọc mọc thêm một chiều.
