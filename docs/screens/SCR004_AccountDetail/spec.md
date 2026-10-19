# SCR004_AccountDetail — Screen Spec

> Alias LAB-1: **SC-04** · アカウント詳細・権限 · Chi tiết tài khoản & quyền

**Screen**: SCR004_AccountDetail: Chi tiết tài khoản & quyền
**Feature**: F001_AdminAndPermissions — [screens.md](../../features/F001_AdminAndPermissions/screens.md)
**Type**: composite *(REG001 form đổi chức danh · REG002 lịch sử đổi quyền — xem [screen-list.md](../../generated/screen-list.md))*
**Route**: `/accounts/[id]`
**Generated**: 2026-10-06
**Trạng thái**: Đã xây ở LAB-3
**Tính năng LAB-1**: `F01-03` `F01-04`
**User story**: không có US riêng — màn quản trị hỗ trợ (`F01`)
**Ảnh**: [sc04-account-detail-vi.png](../../design/screenshots/sc04-account-detail-vi.png)

## Purpose

Xem chi tiết một tài khoản, đổi chức danh (ghi thật qua RPC), và xem lịch sử đổi quyền (quyền cũ, quyền mới, ai làm, lúc nào). Dùng bởi `sys_admin`, không gắn với ca làm việc cụ thể (hành chính, không phải hiện trường). Đây là **màn duy nhất** trong toàn hệ thống mà `sys_admin` ghi được dữ liệu.

## Screen Layout

Khung `AppLayout`; vùng nội dung xếp dọc `gap-8`: `PageHeader` (tiêu đề "Chi tiết tài khoản" + mô tả `{họ tên} · {email}`), khối "Đổi chức danh" (form `max-w-md`), khối "Lịch sử thay đổi quyền" (`DataTable` mới nhất trước) (`app/(app)/accounts/[id]/page.tsx:31-54`). *Lịch sử là một khối dưới form, không phải tab thật dù LAB-1 gọi là "tab".*

### Layout Sketch

```
┌──────────────────────────────────────────────────────┐
│ R1: AppHeader                                        │
├────────────┬─────────────────────────────────────────┤
│ R2: AppNav │ R3: PageHeader  "{Họ tên} · {email}"    │
│            │ R4: Đổi chức danh (REG001)              │
│            │   Chức danh hiện tại: X                 │
│            │   [Chức danh mới ▾] [Ghi chú ...]       │
│            │   (banner lỗi/đúng) [Xác nhận đổi quyền]│
│            │ R5: Lịch sử thay đổi quyền (REG002)     │
│            │   Quyền cũ|Quyền mới|Người thao tác|... │
└────────────┴─────────────────────────────────────────┘
```

### Layout Regions

| Region ID | Name | Position | Scrollable | Key Components | Responsive Behavior |
|-----------|------|----------|------------|----------------|---------------------|
| R1 | App header | static | no | `AppHeader` | always |
| R2 | Nav dọc | static | no | `AppNav` | `w-64` |
| R3 | Tiêu đề | static | no | `PageHeader` | always |
| R4 | Khối đổi chức danh (REG001) | static | no | `RoleChangeForm`, `Field` | `max-w-md` |
| R5 | Khối lịch sử (REG002) | static | ngang | `RoleHistoryTable`, `DataTable` | tràn ngang khi hẹp |

## Child Routes

Không áp dụng — màn này là một route đơn, không phải khung chứa route con.

## User Flow

> **Scope:** within-screen interactions only.

### Happy Path

1. Từ `SCR003_AccountList`, bấm vào tên tài khoản → `/accounts/[id]`.
2. Trang hiện: tiêu đề (tên + email), khối "Đổi chức danh" (R4), khối "Lịch sử thay đổi quyền" (R5, mới nhất trước).
3. Chọn chức danh mới (mặc định = hiện tại), gõ ghi chú (tuỳ chọn), bấm "Xác nhận đổi quyền".
4. **Thành công** → banner xanh, select hiện role MỚI (`revalidatePath` nạp lại `account.role`), bảng lịch sử có thêm 1 dòng trên cùng.

### Branches

| Decision point | Condition | Outcome on this screen | Source |
|----------------|-----------|------------------------|--------|
| Bước 3 | thất bại (kể cả tự đổi chính mình — `P0007`) | banner đỏ nêu lý do cụ thể, không có gì thay đổi dưới DB | `actions.ts:56-58` |
| Bước 1 | `id` trong URL không tồn tại trong `app_users` | `notFound()` → trang 404 mặc định Next.js (không có `not-found.tsx` tuỳ biến) | `page.tsx:26-27` |

## Data Inventory

| Field | Display Label | Source | Format | Empty Behavior | Cross-ref |
|-------|---------------|--------|--------|----------------|-----------|
| tiêu đề | `{họ tên} · {email}` | `app_users.full_name`, `app_users.email` của `id` trong URL | text | — | `app_users` |
| `currentRole` | Chức danh hiện tại / 現在の職掌 | `app_users.role` | dịch qua `common.roles` | — | `app_users.role` |
| `oldRole` (lịch sử) | Quyền cũ / 変更前 | `role_change_log.old_role` | dịch qua `common.roles` | "—" (`null` hợp lệ cho dòng đầu — tài khoản chưa từng đổi quyền) | `role_change_log` |
| `newRole` (lịch sử) | Quyền mới / 変更後 | `role_change_log.new_role` | dịch qua `common.roles` | — | `role_change_log` |
| `changedByName` | Người thao tác / 実行者 | tra `app_users.full_name` theo `role_change_log.changed_by` bằng **truy vấn THỨ HAI**, không embed PostgREST (`role_change_log` có 2 FK tới cùng `app_users` — `target_user_id` và `changed_by` — nên tên ràng buộc auto-sinh không chắc khớp hint embed) | text | fallback = `changed_by` (uuid) nếu không tra được tên | `app_users` |
| `changedAt` | Thời điểm / 日時 | `role_change_log.changed_at` | `.toLocaleString()` | — | `role_change_log` |
| `note` (lịch sử) | Ghi chú / メモ | `role_change_log.note` | text | "—" | `role_change_log` |

Cột: `role-history-table.tsx:14-32`; truy vấn: `lib/domain/accounts.ts:87-121`.

## UI States

| State | Trigger | Visual Behavior | User Action Available | Source |
|-------|---------|----------------|-----------------------|--------|
| loading | — | không có (Server Component) | none | `page.tsx:18` |
| empty | tài khoản chưa từng đổi quyền | khối lịch sử hiện "Chưa có lần đổi quyền nào." / 権限変更履歴はまだありません。 | — | `role-history-table.tsx:39` |
| saving | sau khi bấm "Xác nhận đổi quyền", trước khi Server Action trả kết quả | nút `disabled`, **không đổi nhãn** (khác `SCR002`) | none | `role-change-form.tsx:82-88` |
| success | `success: true` | banner xanh lục `role="status"` "Đã đổi quyền thành công."; `revalidatePath` làm mới `currentRole` và lịch sử; trang không chuyển hướng | — | `role-change-form.tsx:76-80`, `actions.ts:60-61` |
| error (nghiệp vụ) | 1 trong 4 mã ở `## Validation & Error Feedback` | banner đỏ dưới select; `newRole` đã chọn và `note` đã gõ **giữ nguyên** (không reset), sửa và gửi lại ngay | sửa & gửi lại | `role-change-form.tsx:71-75` |
| error (mạng) | request tới Server Action không tới được server | **không xử lý riêng** — không `try/catch` quanh transport; hành vi không xác định được chỉ từ đọc code | — | `role-change-form.tsx:30` |
| read-only | không áp dụng — đổi quyền không gắn ngày nghiệp vụ, luôn ghi được bất kể `business_days.status` | — | — | — |

## Validation & Error Feedback

### A) Client-side

| Field | Type | Required | Constraints | Async Check | Error Message |
|-------|------|----------|-------------|-------------|---------------|
| `userId` (hidden) | hidden input | yes | phải là uuid tồn tại trong `app_users`; lấy từ `params.id`, không cho người dùng sửa | không | `P0001` nếu thiếu |
| `newRole` | select, 4 lựa chọn | yes | thuộc 4 giá trị `app_role`; RPC chặn nếu `target_user_id` trùng chính người gọi, bất kể giá trị chọn (`P0007`) | không | `generic` nếu ngoài enum |
| `note` | textarea, 2 dòng | no | không ràng buộc độ dài ở cả client lẫn server; rỗng → lưu `null` | không | — |

#### Field Labels & Defaults *(mục bổ sung)*

| Field | Nhãn VI | Nhãn JA | Mặc định | Nguồn dữ liệu |
|---|---|---|---|---|
| `newRole` | Chức danh mới | 新しい職掌 | = chức danh hiện tại (`defaultValue={currentRole}`) | option list = `ROLE_OPTIONS` (hằng số 4 role, `page.tsx:11`) |
| `note` | Ghi chú | メモ | rỗng | input người dùng |
| `userId` | — | — | = `account.id` | `params.id` của route |

#### Mã lỗi hiển thị

| Mã | Điều kiện kích hoạt | Thông báo VI | Thông báo JA | Hiện ở đâu | Chặn gửi? |
|---|---|---|---|---|:---:|
| `P0001` | `userId` bị thiếu khỏi `FormData` (guard phòng thủ — chỉ bị khi DOM bị can thiệp) | "Không tìm thấy bản ghi." | "該当するレコードが見つかりません。" | `<p role="alert">` dưới select, trên nút submit | ● |
| `generic` | `newRole` gửi lên không thuộc 4 giá trị hợp lệ (DOM bị can thiệp) | "Có lỗi xảy ra, vui lòng thử lại." | "エラーが発生しました。もう一度お試しください。" | nt | ● |
| `P0005` | người gọi không phải `sys_admin` — bắt ở Server Action TRƯỚC khi chạm RPC, hoặc RPC tự kiểm role thất bại | "Chức danh của bạn không được phép thực hiện thao tác này." | "この操作を行う権限がありません。" | nt | ● |
| `P0007` | RPC phát hiện `p_user_id` == chính người gọi (tự đổi quyền chính mình) — **Server Action KHÔNG tự kiểm điều kiện này**, toàn bộ chặn nằm ở RPC | "Không thể tự đổi quyền của chính mình." | "自分自身の権限は変更できません。" | nt | ● |

`error.code` lấy trực tiếp từ `PostgrestError.code` (`extractErrorCode`, `actions.ts:17-19`); 4 mã trên **không đọc `error.details`** vì không cần dựng câu thông báo động (khác `SCR010` dùng `details` để chèn giá trị).

### B) Server-side

#### Đổi chức danh
- **Endpoint:** Server Action `changeRole` → `rpc('change_user_role', { p_user_id, p_new_role, p_note })`
- **Request:** `userId`, `newRole`, `note` (hoặc `null`)
- **Success:** `jsonb { user_id, old_role, new_role }` → `{ error: null, success: true }` + `revalidatePath("/accounts/{id}")`
- **Errors:** `P0001` Không tìm thấy bản ghi | `P0005` Chức danh không được phép | `P0007` Không thể tự đổi quyền | `generic`
- **Trigger:** click "Xác nhận đổi quyền" (submit form)
- **Source:** `app/(app)/accounts/[id]/actions.ts:29-62`; RPC `supabase/migrations/0012_change_user_role_self_guard.sql:11-34` (thay bản `0008_rpc_functions_admin.sql:46`)

Đổi quyền **bắt buộc qua RPC** — `app_users` không có policy ghi trực tiếp (tài liệu gốc trong repo, không nộp: `db-schema.md` §6), đây là đường ghi DUY NHẤT.

## Interaction Patterns

- **Select mặc định bằng chức danh hiện tại; gửi mà không đổi gì vẫn gọi RPC** — source: `role-change-form.tsx:52`.
- **Sau khi thành công, form không reset; select hiện role mới nhờ `revalidatePath`** — source: `actions.ts:60`.
- Không có yêu cầu bàn phím riêng (không thuộc 3 màn ⌨️).

## Accessibility

| Aspect | Status | Notes |
|--------|--------|-------|
| ARIA roles/labels | partial | `Field` gắn `aria-describedby`/`aria-invalid` (`components/ui/field.tsx:28-32`); banner lỗi `role="alert"`, thành công `role="status"` |
| Keyboard navigation | supported | form chuẩn: select → textarea → nút |
| Focus management | unmanaged | không focus banner khi lỗi (khác `SCR010`) |
| Screen reader compatibility | unknown | chưa kiểm |

## Conditional Rendering

| Condition | Type | Renders | Hidden | Notes |
|-----------|------|---------|--------|-------|
| role ≠ `sys_admin` | auth | — | cả route | `proxy.ts` → `/403`; `changeRole` tự kiểm tiếp (`actions.ts:34`) |
| `state.error` | ui-state | banner đỏ | — | `role-change-form.tsx:71-75` |
| `state.success` | ui-state | banner xanh | — | `role-change-form.tsx:76-80` |
| `account === null` | ui-state | `notFound()` | toàn trang | `page.tsx:27` |

## Component Variants

Không áp dụng — màn không có component đổi hình dạng theo biến thể; khác biệt theo chức danh/trạng thái đã mô tả ở Conditional Rendering.

## Security Surface

| Guard | Type | Consequence if bypassed |
|-------|------|------------------------|
| `ROUTE_POLICY { prefix: "/accounts" }` (khớp cả `/accounts/[id]`) chỉ `sys_admin` | role | role khác → `/403` (`route-policy.ts:25`) |
| **Tầng 2** — Server Action `changeRole` tự kiểm `currentUser.role !== "sys_admin"` → `P0005` (lý do trong code: Server Function là POST tới chính route chứa nó — matcher đổi hoặc route bị refactor có thể âm thầm mất lớp proxy) | role | `actions.ts:33-36` |
| **Tầng 3** — RPC `change_user_role` (SECURITY DEFINER, bỏ qua RLS) tự kiểm role ở dòng đầu → `P0005`; chặn tự đổi quyền chính mình → `P0007` (nguồn sự thật DUY NHẤT của ràng buộc này) | role | `0012_change_user_role_self_guard.sql:15-25` |
| RLS `role_change_log_select`: chỉ `sys_admin` đọc; không policy ghi | permission | `0009_rls_policies.sql:26-27` |

3 lớp kiểm độc lập cho cùng một ràng buộc (ADR-004: bất biến giữ ở tầng thấp nhất).

## Source References

1. Page/View: `app/(app)/accounts/[id]/page.tsx:18-56`
2. Form: `app/(app)/accounts/[id]/role-change-form.tsx:22-91`
3. Server Action: `app/(app)/accounts/[id]/actions.ts:29-62`
4. Lịch sử: `app/(app)/accounts/[id]/role-history-table.tsx:13-42`
5. Truy vấn: `lib/domain/accounts.ts:59-70` (`getAccountById`), `:87-121` (`listRoleChangeHistory`)
6. RPC: `supabase/migrations/0012_change_user_role_self_guard.sql:11-34`; RLS `0009_rls_policies.sql:20-27`
7. i18n: `lib/i18n/{vi,ja}/accounts.ts`; `common.errors` (`lib/i18n/vi/common.ts:121-126`)

## Source Walkthrough

1. **File:** `supabase/migrations/0012_change_user_role_self_guard.sql:11-34` — bắt đầu từ RPC: ràng buộc nghiệp vụ nằm ở đây.
2. **File:** `app/(app)/accounts/[id]/actions.ts:29-62` — Server Action: tầng phòng thủ thứ hai + ánh xạ mã lỗi.
3. **File:** `app/(app)/accounts/[id]/role-change-form.tsx:22-91` — form.
4. **File:** `lib/domain/accounts.ts:87-121` — vì sao lịch sử dùng truy vấn thứ hai.
5. **File:** `app/(app)/accounts/[id]/page.tsx:18-56` — lắp ghép.

### Call Hierarchy

```text
AccountDetailPage -> getAccountById / listRoleChangeHistory -> RoleChangeForm -> changeRole -> rpc change_user_role
                                                          \-> RoleHistoryTable
```

### Read Calls *(mục bổ sung)*

| Thao tác | Cách gọi | Tham số | Trả về |
|---|---|---|---|
| Đọc chi tiết tài khoản | `select` từ `app_users` qua `getAccountById()` | `id` (từ URL) | `AccountRow` hoặc `null` |
| Đọc lịch sử đổi quyền | `select` từ `role_change_log` (lọc `target_user_id`), rồi `select` thứ hai từ `app_users` để tra tên người thao tác | `targetUserId` | mảng `RoleChangeEntry`, mới nhất trước |
| Đổi chức danh | `rpc('change_user_role', { p_user_id, p_new_role, p_note })` | `userId`, `newRole`, `note` hoặc `null` | `jsonb { user_id, old_role, new_role }`; raise `P0001`/`P0005`/`P0007` |

### Dependencies & Risks *(mục bổ sung)*

- Phụ thuộc `SCR003_AccountList` để điều hướng vào — không có cách nhập trực tiếp `id` từ UI nào khác.
- RPC `change_user_role` là nguồn sự thật DUY NHẤT cho "không tự đổi quyền chính mình" (`P0007`) — `changeRole()` **không** kiểm `targetUserId === currentUser.id`. Dễ đọc nhầm khi chỉ xem `actions.ts` mà tưởng chưa có chặn.
- `SCR004` hiện **chỉ đổi được `role`**, không đổi được `status` (`active`/`suspended`) — khoá/mở khoá tài khoản (thao tác nhạy cảm theo (tài liệu gốc trong repo, không nộp) `architecture.md` §C1, `TBL-ROLE-01`) không có UI, phải thao tác tay trên DB.
- Ma trận quyền (9 role, PREMISE C-09): `sys_admin` xem ●/ghi ●; `receiving`, `trading`, `reconcile` ✗/✗; 5 role đề xuất (`party_admin`, `rule_admin`, `ops_admin`, `market_manager`, `internal_audit`) ✗/✗. Cờ `can_approve` không liên quan quyền mở màn này.
- Link chéo: [F001](../../features/F001_AdminAndPermissions/screens.md); ADR-001 (chức danh từ `app_users`), ADR-004 (bất biến ở DB).

### Unresolved Questions *(mục bổ sung)*

- `SCR004` chưa có cách khoá/mở khoá tài khoản từ UI dù đây được kiến trúc gọi là thao tác nhạy cảm — cần bổ sung hay giữ thao tác tay?
- Hành vi khi mất mạng lúc bấm "Xác nhận đổi quyền" chưa được định nghĩa (không có xử lý mạng riêng).
- Bản đã xây chỉ cho chọn 4 role (`ROLE_OPTIONS`) và chưa có cách bật/tắt cờ `app_users.can_approve` (đã chốt ở PREMISE C-09). Khi làm thật: mở rộng danh sách lên 9 role, thêm ô bật cờ (ghi qua RPC, vào `role_change_log`), và quyết định ai được cấp cờ cho người có quyền duyệt (tránh `sys_admin` tự cấp cho chính mình — `P0007` hiện chỉ chặn đổi `role`).
