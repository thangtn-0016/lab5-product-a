# SCR023_PendingRequestList — Screen Spec

> Alias LAB-1: **SC-23** · 承認待ち申請票一覧 · Danh sách phiếu chờ duyệt

**Screen**: SCR023: PendingRequestList (Danh sách phiếu chờ duyệt)
**Feature**: [F007_PostCloseCorrection](../../features/F007_PostCloseCorrection/screens.md) — tính năng LAB-1 `F07-05`
**Type**: atomic (danh sách + bộ lọc, đọc thuần)
**Route**: `/corrections` *(đề xuất)*
**Generated**: 2026-10-06
**Trạng thái**: Thiết kế mới, chưa xây (LAB-4)
**Role**: `reconcile` (xem) · `reconcile` có cờ `can_approve` *(đề xuất)* (checker — mở phiếu để duyệt)
**User story**: `US-17` (nhánh phụ — `F07-05` được LAB-1 xếp là *"biến thể hoặc phần đuôi của story đã viết"*)

> **Ký hiệu nguồn.** `[KH]` = bắt buộc theo tài liệu khách / LAB-1, có dẫn nguồn. `[ĐX]` = đề xuất của tài liệu này. Bảng `correction_requests`, view `v_correction_requests`, cờ `app_users.can_approve` và mọi mã `P0020`+ đều là `[ĐX]` — chưa có trong `db-schema.md`.

## Purpose

Hàng đợi việc của **người duyệt** (承認者), dùng vào giờ hành chính, thường một lần đầu buổi sáng và một lần cuối buổi chiều: mở ra là thấy ngay phiếu nào đang chờ mình, chờ **bao lâu rồi** `[KH — F07-05: "kèm thời gian chờ"]`, và phiếu nào mình **không được phép đụng vào** vì chính mình đã 起票 nó. Đây là màn đọc thuần — không có ô nhập số liệu nào; hành động duy nhất là mở phiếu sang SCR024.

Màn này là nơi quy tắc `起票・承認分離` trở nên **nhìn thấy được**: nút duyệt biến mất, không phải mờ đi.

## Screen Layout

Trang danh sách trong khung ứng dụng chung: tiêu đề, bộ lọc GET, bảng một dòng = một phiếu; cột hành động cho ba nhánh nút khác nhau theo người xem. Chưa có code nên chưa có breakpoint chốt.

### Layout Sketch

```
┌──────────────────────────────────────────────────────┐
│ R1: Header ứng dụng + nav (static)                │
├──────────────────────────────────────────────────────┤
│ R2: Tiêu đề · banner thông tin (nếu không can_approve)│
├──────────────────────────────────────────────────────┤
│ R3: Bộ lọc: trạng thái · loại bản ghi · ngày NV      │
│              · ☐ ẩn phiếu do tôi lập · [Áp dụng]     │
├──────────────────────────────────────────────────────┤
│ R4: Bảng phiếu (chờ lâu nhất lên đầu) + cột hành động│
│     - - - rỗng / rỗng (chỉ còn phiếu của tôi) - - -  │
└──────────────────────────────────────────────────────┘
```

### Layout Regions

| Region ID | Name | Position | Scrollable | Key Components | Responsive Behavior |
|-----------|------|----------|------------|----------------|---------------------|
| R1 | Header + nav | static | no | khung `(app)` chung | always |
| R2 | Tiêu đề + banner | static | no | `PageHeader`, banner thông tin | fluid |
| R3 | Bộ lọc | static | no | `<form method="GET">` | fluid, wrap |
| R4 | Bảng phiếu | static | yes | `DataTable`, `StatusBadge`, nút/link hành động | fluid |

## Child Routes

Không áp dụng — màn này là một route đơn, không phải khung chứa route con.

## User Flow

> Phạm vi: tương tác trong màn. Điều hướng liên màn xem [screen-flow](../../generated/screen-flow.md).

### Happy Path

1. Người duyệt mở `/corrections`, mặc định thấy toàn bộ phiếu `pending` sắp theo `waiting` giảm dần (chờ lâu nhất lên đầu).
2. Quét cột `waiting` để biết việc nào trễ, cột `delta_amount` để biết việc nào nặng tiền.
3. Nhánh A — phiếu do người khác lập, có `can_approve`: bấm "Mở phiếu & duyệt" → SCR024.
4. Nhánh B — phiếu do chính mình lập: không có nút; mở xem được, nhưng SCR024 sẽ không hiện khối duyệt.
5. Nhánh C — không có `can_approve`: banner thông tin, chỉ xem.
6. Sau khi xử lý ở SCR024, quay lại danh sách đã cập nhật.
7. Lọc `decision_state='approved'` để tra lịch sử — truy vết bản ghi điều chỉnh đi qua SCR024 (`F07-04`).

### Branches

Ba nhánh của cột `action` — điểm quan trọng nhất của màn này:

| Decision point | Condition | Outcome on this screen | Source |
|----------------|-----------|------------------------|--------|
| Cột `action` | `decision_state='pending'` **và** `requested_by <> auth.uid()` **và** người xem có `can_approve` | nút **"Mở phiếu & duyệt"** → `/corrections/{id}` | thiết kế LAB-4 |
| Cột `action` | `decision_state='pending'` **và** `requested_by = auth.uid()` | **không có nút duyệt** `[KH — screen-list SC-23, US-17]`; thay bằng liên kết chữ "Xem phiếu của tôi" + chữ nhỏ *"Bạn không duyệt được phiếu do chính mình lập"* | thiết kế LAB-4 |
| Cột `action` | `decision_state ≠ 'pending'` | liên kết chữ "Xem" → `/corrections/{id}` ở chế độ chỉ đọc | thiết kế LAB-4 |

Dòng thứ hai là **ẩn nút, không phải disable nút**. Một nút xám vẫn gợi ý "có cách bật lên được"; ở đây không có cách nào, và giao diện phải nói đúng điều đó.

## Data Inventory

> Cột của bảng (mỗi dòng = một phiếu). Bộ lọc nằm ở `## Validation & Error Feedback`.

| Field | Display Label | Source | Format | Empty Behavior | Cross-ref |
|-------|---------------|--------|--------|----------------|-----------|
| `request_code` | Mã phiếu / 申請番号 *(đề xuất)* | `correction_requests.request_code` | dạng `COR-YYYYMMDD-0001` | luôn có | correction_requests *(mới, đề xuất)* |
| `target` | Bản ghi đích / 対象レコード *(đề xuất)* | `v_correction_requests.target_code` | `{target_table nhãn} · {target_code}` | luôn có | v_correction_requests *(mới, đề xuất)* |
| `target_business_date` | Ngày nghiệp vụ / 対象業務日 | `correction_requests.target_business_date` | date; luôn là ngày đã `closed` | luôn có | business_days |
| `correction_type` | Kiểu / 訂正区分 *(đề xuất)* | `correction_requests.correction_type` | badge; `reverse` → tông cảnh báo; `delta` → tông trung tính | luôn có | correction_requests |
| `delta_amount` | Chênh lệch tiền / 金額差分 *(đề xuất)* | `v_correction_requests.delta_amount` (server tính) | âm hiện dấu `−` và tông đỏ; định dạng tiền JPY không phần thập phân | luôn có | v_correction_requests |
| `requested_by_name` | Người 起票 / 起票者 | `app_users.full_name` | text | "—" nếu RLS `app_users` chặn | app_users |
| `requested_at` | Lúc 起票 / 起票日時 | `correction_requests.requested_at` | JST | luôn có | correction_requests |
| `waiting` | Thời gian chờ / 滞留時間 *(đề xuất)* | `v_correction_requests.waiting_hours` | `now() − requested_at`, **tính ở view**; `{n} giờ` khi `< 24h`, `{n} ngày` khi `≥ 24h`; `≥ 3 ngày làm việc` → tông cảnh báo `[KH — F07-05]` | luôn có | v_correction_requests |
| `decision_state` | Trạng thái / 申請状態 *(đề xuất)* | `correction_requests.decision_state` | badge: `pending` xanh dương · `approved` xanh lá · `rejected` đỏ · `cancelled` xám | luôn có | correction_requests |
| `action` | Hành động / 操作 | — | nút/link theo ba nhánh ở User Flow → Branches | — | — |

### Read Calls *(mục bổ sung)*

| Thao tác | Cách gọi | Tham số | Trả về |
|---|---|---|---|
| Đọc danh sách | `select` từ `v_correction_requests` *(view mới, `[ĐX]`)*, Server Component trước render | `decision_state`, `target_table`, `target_business_date`, `mine` | mảng dòng kèm `delta_amount`, `waiting_hours`, `requested_by_name` |
| Đọc cờ quyền của người xem | `select can_approve from app_users where id = auth.uid()` | | boolean |

Không có thao tác ghi nào ở màn này.

**`delta_amount` và `waiting_hours` phải tính trong view, không tính ở client.** `waiting_hours` tính ở client thì mỗi máy ra một số theo đồng hồ máy đó; `delta_amount` tính ở client thì số tiền hiển thị không còn là số server sẽ ghi. View `security_invoker = true` để RLS bảng gốc vẫn áp dụng — đúng khuôn 3 view hiện có (`db-schema.md` §3).

**Sắp xếp mặc định theo `waiting_hours` giảm dần**, không theo `requested_at`. Hai cái cho cùng thứ tự hôm nay, nhưng khi thêm phân cấp duyệt (xem Unresolved Questions) thì `waiting` mới là thứ người duyệt cần.

## UI States

| State | Trigger | Visual Behavior | User Action Available | Source |
|-------|---------|----------------|-----------------------|--------|
| empty | không có phiếu khớp bộ lọc | "Không có phiếu nào chờ duyệt." + gợi ý xoá lọc | Xoá lọc | thiết kế LAB-4 |
| empty (biến thể) | có phiếu `pending` nhưng **tất cả** do chính người xem lập | "Chỉ còn phiếu do bạn lập — cần người khác duyệt." — **phải phân biệt với rỗng thật**, nếu không người dùng tưởng hệ thống hỏng | xem phiếu của tôi | thiết kế LAB-4 |
| loading | mở trang / đổi bộ lọc | khung xương bảng | none | thiết kế LAB-4 |
| success | sau khi duyệt ở SCR024 rồi quay lại | `revalidatePath("/corrections")` → dòng vừa duyệt đổi badge và mất nút | — | thiết kế LAB-4 |
| error (nghiệp vụ) | không phát sinh ở màn này | — | — | — |
| error (mạng) | `select` throw | thông báo ở Error Catalogue + nút "Tải lại"; bộ lọc trên query string nên **không mất** | Tải lại | thiết kế LAB-4 |
| read-only | người xem không có `can_approve` | toàn bộ bảng hiển thị bình thường, cột `action` chỉ có liên kết "Xem" | xem | thiết kế LAB-4 |

## Validation & Error Feedback

### A) Client-side

Bộ lọc là `<form method="GET">`, đọc từ `searchParams` — không gửi Server Action, giữ đúng khuôn SCR018.

| Field | Type | Required | Constraints | Async Check | Error Message |
|-------|------|----------|-------------|-------------|---------------|
| `decision_state` / 申請状態 *(đề xuất)* | select | no | ∈ `pending` `approved` `rejected` `cancelled` `all`; mặc định `pending` | không | xem Error Catalogue |
| `target_table` / 対象レコード種別 *(đề xuất)* | select | no | ∈ `trades` `deliveries` `incentive_results`, rỗng = tất cả | không | xem Error Catalogue |
| `target_business_date` / 対象業務日 | date | no | không ràng buộc; để trống = tất cả | không | không có |
| `mine` / 自分の起票を除く *(đề xuất)* | checkbox | no | boolean; mặc định `false`; lọc `requested_by <> auth.uid()` | không | không có |

### B) Server-side

Không áp dụng — màn chỉ đọc, không có submit-style action.

#### Error Catalogue *(mục bổ sung)*

Màn đọc thuần — không có ô nhập số liệu, nên không có validation phía form. Những gì có thể hiện ra:

| Mã | Điều kiện kích hoạt | Thông báo VI | Thông báo JA | Hiện ở đâu | Chặn gửi? |
|---|---|---|---|---|:---:|
| `V-01` | người xem là `requested_by` của dòng `pending` | Bạn không duyệt được phiếu do chính mình lập. | 自身が起票した申請は決裁できません。 | chữ nhỏ thay chỗ nút ở cột `action` | ● — ẩn hẳn nút |
| `V-02` | người xem không có `can_approve`, lọc `pending` | Bạn xem được danh sách nhưng không có quyền duyệt. | 一覧の閲覧は可能ですが、決裁権限がありません。 | banner thông tin trên đầu bảng | ✗ |
| `P0005` | role ≠ `reconcile` gọi thẳng route | *(không hiện ở màn này)* — `proxy.ts` đã đẩy về `/403` trước khi render | — | `/403` | ● |
| `P0027` *(mới, `[ĐX]`)* | gọi thẳng RPC duyệt mà không có `can_approve` (bỏ qua giao diện) | Bạn không có thẩm quyền duyệt phiếu điều chỉnh. | 修正申請の決裁権限がありません。 | `SCR024`, banner đầu phiếu | ● |
| `P0023` *(mới, `[ĐX]`)* | gọi thẳng RPC duyệt phiếu của chính mình | Người duyệt phải khác người 起票. | 決裁者は起票者と異なる必要があります。 | `SCR024`, banner đầu phiếu | ● |
| (network) | `select` thất bại | Không tải được danh sách phiếu. Kiểm tra kết nối rồi thử lại. | 申請票一覧を取得できませんでした。接続を確認して再度お試しください。 | thay chỗ bảng | — |

`P0023` và `P0027` **được liệt kê ở đây nhưng không bao giờ bắn từ màn này** — chúng là lưới an toàn ở tầng RPC, bắn khi ai đó đi vòng qua giao diện. Ghi ra để người triển khai `SCR024` biết hai mã này tồn tại và vì sao giao diện ẩn nút vẫn chưa đủ.

## Interaction Patterns

- **Tab qua các ô lọc → nút "Áp dụng" → liên kết "Xoá lọc" → lần lượt các nút/liên kết trong cột `action` theo thứ tự dòng** (chuẩn trình duyệt) — thiết kế LAB-4.
- **Không có phím tắt duyệt nhanh, có chủ ý** — duyệt là `CAP-HUMAN-01`, phải đi qua SCR024 và đọc bảng so sánh trước/sau — thiết kế LAB-4.
- Không có yêu cầu bàn phím riêng — không thuộc ba màn keyboard-first (SCR007, SCR010, SCR014).

## Accessibility

| Aspect | Status | Notes |
|--------|--------|-------|
| ARIA roles/labels | unknown | chưa có bản dựng; badge trạng thái cần chữ, không chỉ màu (đề xuất) |
| Keyboard navigation | supported (đặc tả) | input gốc + link/nút theo thứ tự dòng |
| Focus management | unmanaged | không có hộp thoại |
| Screen reader compatibility | unknown | chưa kiểm |

## Conditional Rendering

| Condition | Type | Renders | Hidden | Notes |
|-----------|------|---------|--------|-------|
| phiếu `pending`, do người khác lập, người xem có `can_approve` | auth | "Mở phiếu & duyệt" | — | bypass: RPC duyệt vẫn chặn (`P0023`) |
| phiếu do chính người xem lập | auth | link "Xem phiếu của tôi" + chữ nhỏ | nút duyệt | **ẩn, không disable**; ẩn nút là tiện lợi giao diện, không phải phép chặn |
| người xem không có `can_approve` | auth | banner thông tin, link "Xem" | nút duyệt mọi dòng | |
| mọi phiếu `pending` đều do người xem lập | hardcoded-id | thông báo "Chỉ còn phiếu do bạn lập…" | thông báo rỗng thật | |

## Component Variants

Không áp dụng — màn không có component đổi hình dạng theo biến thể; khác biệt theo chức danh/trạng thái đã mô tả ở Conditional Rendering.

## Security Surface

| Guard | Type | Consequence if bypassed |
|-------|------|------------------------|
| `lib/auth/route-policy.ts` → `{ prefix: "/corrections", roles: ["reconcile"] }`, `proxy.ts` đẩy về `/403` *(đề xuất)* | role | role sai mở được màn |
| RLS SELECT trên `correction_requests` cho `reconcile` | permission | đọc ngoài chức danh |
| maker ≠ checker **bên trong RPC duyệt** (SCR024, `P0023`) | permission | tự duyệt phiếu của mình. Việc ẩn nút ở màn này là **tiện lợi giao diện, không phải phép chặn** `[KH — US-17: "cưỡng chế ở tầng dữ liệu"]`; bài học ADR-004: bất biến nghiệp vụ nằm ở tầng giao diện thì chỉ là quy ước code, và RPC viết sau sẽ không có nó |

Quyền theo chức danh: `sys_admin` ✗ · `reconcile` xem — thấy phiếu mình lập, **không thấy nút duyệt ở bất kỳ phiếu nào** · `reconcile` có cờ `can_approve` *(đề xuất)* xem + ghi — là **checker**; nút duyệt hiện ở mọi phiếu **trừ** phiếu do chính mình lập `[KH — screen-list SC-23]` · `trading` · `receiving` ✗.

> **Vì sao dùng cờ `can_approve` chứ không thêm chức danh riêng cho người duyệt (đã chốt, PREMISE C-09).** `GOV-RULE-01` nói tách **người**, không nói tách **chức danh**. Dùng lại đúng lựa chọn của SCR036 (một cột boolean trên `app_users`) để cả hệ thống chỉ có **một** cơ chế phân người duyệt — thêm vai thứ hai cho F07 là nhân đôi ma trận quyền mà không đổi được gì. `[ĐX]` ở mức role mới; lựa chọn cờ đã chốt ở C-09.

## Source References

Chưa có code — màn chỉ đặc tả ở LAB-4. Tài liệu nguồn đã đọc:

1. (tài liệu gốc trong repo, không nộp) `sc-23-danh-sach-phieu-cho-duyet.md` — spec 10 mục gốc
2. `lib/auth/route-policy.ts:17-27` — `ROUTE_POLICY` hiện có (chưa có `/corrections`)
3. (tài liệu gốc trong repo, không nộp) `db-schema.md` §3 — khuôn 3 view `security_invoker`

## Source Walkthrough

Chưa có code — màn chỉ đặc tả ở LAB-4.

### Call Hierarchy

```text
(thiết kế) /corrections page -> v_correction_requests (+ app_users.can_approve) (đọc)
```

## Dependencies & Risks *(mục bổ sung)*

- **Cần trước:** `SCR022` (không có phiếu thì danh sách luôn rỗng), bảng `correction_requests`, view `v_correction_requests`, cờ `app_users.can_approve`.
- **Cần đồng bộ:** `SCR024` phải chặn maker=checker ở RPC. Làm màn này mà `SCR024` tin vào việc nút đã bị ẩn là để ngỏ đúng lỗ hổng `US-17` yêu cầu bịt.
- **Dữ liệu phải có sẵn:** **ít nhất hai tài khoản `reconcile`**, một có `can_approve`. Một tài khoản thì không phiếu nào duyệt được và màn này chỉ hiện nhánh B — demo sẽ trông như hỏng. Seed hiện chỉ có **một** `reconcile@demo.local` (`db-schema.md` §8), phải thêm.
- **Dễ hiểu sai #1:** disable nút thay vì ẩn nút. Nút xám gợi ý có đường bật; không có đường nào.
- **Dễ hiểu sai #2:** lọc phiếu của chính mình **ra khỏi** danh sách luôn. Không được — người lập phải theo dõi được phiếu của mình đang chờ ai. Ẩn nút, giữ dòng. Checkbox `mine` là **tuỳ chọn của người xem**, không phải mặc định.
- **Dễ hiểu sai #3:** `requested_by_name` có thể `null` dù `requested_by` có giá trị — RLS `app_users` chỉ cho tự xem mình + `sys_admin` xem toàn bộ (`db-schema.md` §6). Người duyệt sẽ thấy "—" ở cột người 起票, tức là **không biết phiếu của ai mà vẫn được duyệt**. Đây là lỗi nghiệp vụ thật, không phải khiếm khuyết hiển thị: phải nới RLS `app_users` cho `reconcile` đọc `full_name`, hoặc `v_correction_requests` phải là `security_definer` chỉ lộ đúng cột tên. Cùng gốc với vấn đề `closedByName` của `SCR018` nhưng hệ quả nặng hơn.
- **Rủi ro vận hành:** chợ nhỏ có thể chỉ có một người đủ thẩm quyền đối chiếu. Lúc đó maker-checker thành vật cản vận hành chứ không phải kiểm soát — cần biết số người **trước** khi triển khai, không phát hiện lúc chạy thật.

### Unresolved Questions *(mục bổ sung)*

1. **Số cấp duyệt.** Câu 3 gửi khách (tài liệu LAB-1 `product-a-rfp-qa-part-a-client.md`) chưa có trả lời. Một cấp → màn này đúng như mô tả. Phân cấp theo số tiền → phải thêm cột "cấp duyệt", lọc theo cấp của người xem, và một màn định nghĩa ngưỡng `[KH — screen-list SCR024 ghi rõ hệ quả này]`.
2. **Ai được gắn cờ `can_approve`?** Cơ chế đã chốt (PREMISE C-09: cờ trên tài khoản, không thêm chức danh `correction_approver`); còn mở là *danh sách người cụ thể* mang cờ và cách cấp/thu hồi cờ (do `sys_admin` hay `party_admin`?). Quyết định ngoài hệ thống rồi chỉ ghi lại kết quả bị loại vì không cưỡng chế được maker-checker, trái `US-17`.
3. **"Thời gian chờ" tính theo giờ đồng hồ hay giờ làm việc?** `[ĐX]` hiện tính theo giờ đồng hồ. Phiếu lập chiều thứ Sáu sẽ hiện "3 ngày" vào sáng thứ Hai dù chưa qua ngày làm việc nào. Tính theo ngày làm việc thì cần lịch nghỉ chợ của `SCR021`.
4. **Ngưỡng cảnh báo trễ là bao nhiêu?** `[ĐX]` 3 ngày làm việc. Chưa có yêu cầu nào định lượng.
5. **Có cần thông báo (email/đẩy) khi có phiếu mới không?** Tài liệu khách không nhắc. Không có thông báo thì "thời gian chờ" chỉ đo được sau khi ai đó chịu mở màn này ra xem.
6. **Phiếu đã xử lý giữ trong danh sách bao lâu?** Lọc `all` trên một hệ thống chạy vài năm sẽ nặng. Chưa có yêu cầu lưu trữ/đóng băng.
