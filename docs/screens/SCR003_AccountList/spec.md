# SCR003_AccountList — Screen Spec

> Alias LAB-1: **SC-03** · 社内アカウント一覧 · Danh sách tài khoản nội bộ

**Screen**: SCR003_AccountList: Danh sách tài khoản nội bộ
**Feature**: F001_AdminAndPermissions — [screens.md](../../features/F001_AdminAndPermissions/screens.md)
**Type**: atomic
**Route**: `/accounts`
**Generated**: 2026-10-06
**Trạng thái**: Đã xây ở LAB-3
**Tính năng LAB-1**: `F01-03`
**User story**: không có US riêng — màn quản trị hỗ trợ (`F01`), không thuộc chuỗi US-01..US-15
**Ảnh**: [sc03-accounts-vi.png](../../design/screenshots/sc03-accounts-vi.png)

## Purpose

Danh sách tài khoản nội bộ cho `sys_admin` quản lý — xem ai đang giữ chức danh gì, lọc theo chức danh/trạng thái, bấm vào một dòng để sang `SCR004_AccountDetail` đổi quyền. Đây là **một trong hai màn duy nhất** `sys_admin` mở được (cùng `SCR004`) — quyết định có chủ ý: `sys_admin` **không** mở được bất kỳ màn nghiệp vụ nào (`SCR007`..`SCR018`), để chứng minh phân quyền là hạn chế thật chứ không phải "admin thấy tất cả" (tài liệu gốc trong repo, không nộp: `architecture.md` §C1; ADR-001).

## Screen Layout

Khung `AppLayout` (header + nav + `<main>`); vùng nội dung xếp dọc: `PageHeader` (tiêu đề), thanh lọc `FilterBar` (GET form, 2 `<select>` + nút Áp dụng/Xoá lọc), rồi `DataTable` 5 cột (`app/(app)/accounts/page.tsx:87-103`). Bảng cuộn ngang khi hẹp (`overflow-x-auto`, `components/ui/data-table.tsx:29`).

### Layout Sketch

```
┌────────────────────────────────────────────────────────────┐
│ R1: AppHeader                                              │
├──────────────┬─────────────────────────────────────────────┤
│ R2: AppNav   │ R3: PageHeader "Danh sách tài khoản nội bộ" │
│              │ R4: FilterBar [Chức danh▾][Trạng thái▾][Áp dụng][Xoá lọc]
│              │ R5: DataTable  Họ tên | Email | Chức danh |  │
│              │                Trạng thái | Ngày tạo         │
└──────────────┴─────────────────────────────────────────────┘
```

### Layout Regions

| Region ID | Name | Position | Scrollable | Key Components | Responsive Behavior |
|-----------|------|----------|------------|----------------|---------------------|
| R1 | App header | static | no | `AppHeader` | always |
| R2 | Nav dọc | static | no | `AppNav` | `w-64` cố định |
| R3 | Tiêu đề | static | no | `PageHeader` | always |
| R4 | Thanh lọc | static | no | `FilterBar` (`components/ui/filter-bar.tsx:25-66`) | `flex-wrap` |
| R5 | Bảng tài khoản | static | ngang (`overflow-x-auto`) | `DataTable`, `StatusBadge`, `<Link>` | tràn ngang khi hẹp |

## Child Routes

Không áp dụng — màn này là một route đơn, không phải khung chứa route con.

## User Flow

> **Scope:** within-screen interactions only.

### Happy Path

1. `sys_admin` mở `/accounts` từ menu hoặc link trực tiếp.
2. Mặc định không lọc, hiện toàn bộ tài khoản sắp theo `full_name` tăng dần (`lib/domain/accounts.ts:45-48`).
3. Chọn bộ lọc chức danh/trạng thái (R4) → bấm "Áp dụng" (GET form, reload với query string mới).
4. Bấm "Xoá lọc" → quay về `/accounts` không query string.
5. Bấm vào tên một tài khoản (R5) → sang `SCR004_AccountDetail` (`/accounts/[id]`).

### Branches

| Decision point | Condition | Outcome on this screen | Source |
|----------------|-----------|------------------------|--------|
| Bước 3 | `searchParams.role`/`status` chứa giá trị không khớp enum | **bỏ qua lặng lẽ**, coi như không lọc (không có thông báo) | `accounts/page.tsx:29-32` |
| Bước 2 | không có dòng khớp | `EmptyState` | `components/ui/data-table.tsx:24-26` |

## Data Inventory

| Field | Display Label | Source | Format | Empty Behavior | Cross-ref |
|-------|---------------|--------|--------|----------------|-----------|
| `fullName` | Họ tên / 氏名 | `app_users.full_name` | readonly text, có `<Link>` sang `/accounts/[id]` | — | `app_users` |
| `email` | Email / メール | `app_users.email` | readonly text | — | `app_users` |
| `role` | Chức danh / 職掌 | `app_users.role` | dịch qua `common.roles`; 1 trong 4 giá trị enum `app_role` của bản đã xây (5 role đề xuất chưa có trong enum) | — | `app_users.role` |
| `status` | Trạng thái / 状態 | `app_users.status` | badge: `active` → positive, `suspended` → danger | — | `app_users.status` |
| `createdAt` | Ngày tạo / 作成日 | `app_users.created_at` | `.toLocaleDateString()` | — | `app_users` |

Cột/nhãn: `accounts/page.tsx:58-85`. Toàn bộ #1–#5 là bắt buộc (không null).

## UI States

| State | Trigger | Visual Behavior | User Action Available | Source |
|-------|---------|----------------|-----------------------|--------|
| loading | — | không có — Server Component chờ xong truy vấn mới render, không `loading.tsx` | none | `accounts/page.tsx:21` |
| empty | không có tài khoản khớp bộ lọc | `EmptyState`: "Không có tài khoản nào khớp bộ lọc." / 条件に一致するアカウントがありません。 | "Xoá lọc" | `accounts/page.tsx:100`, `data-table.tsx:24-26` |
| error | `listAccounts()` `throw` khi Supabase lỗi, không bắt ở `page.tsx`, không có `error.tsx` | rơi xuống trang lỗi mặc định Next.js | tải lại | `lib/domain/accounts.ts:54` |
| saving / success | không áp dụng — không có ghi trên màn này | — | — | — |
| read-only | không áp dụng — màn vốn chỉ đọc | — | — | — |

## Validation & Error Feedback

### A) Client-side

Không có form ghi — chỉ có bộ lọc GET; không mã lỗi nào kích hoạt được.

| Field | Type | Required | Constraints | Async Check | Error Message |
|-------|------|----------|-------------|-------------|---------------|
| bộ lọc `role` | select, 4 lựa chọn + "Tất cả" | no | thuộc 4 role hoặc rỗng; giá trị lạ trong URL bị **bỏ qua lặng lẽ**, coi như không lọc (`ROLE_VALUES.includes()`) | không | (không có thông báo — lặng lẽ rơi về "tất cả") |
| bộ lọc `status` | select, 2 lựa chọn + "Tất cả" | no | thuộc `active`/`suspended` hoặc rỗng, tương tự | không | (không có thông báo) |

#### Field Labels & Defaults *(mục bổ sung)*

| Field | Nhãn VI | Nhãn JA | Mặc định | Nguồn dữ liệu |
|---|---|---|---|---|
| bộ lọc `role` | Chức danh | 職掌 | rỗng = tất cả | `searchParams.role`, kiểm bằng `ROLE_VALUES.includes()` (`page.tsx:13,29`) |
| bộ lọc `status` | Trạng thái | 状態 | rỗng = tất cả | `searchParams.status` (`page.tsx:14,30-32`) |

### B) Server-side

N/A — không có hành động submit kiểu ghi. Bộ lọc là GET; "chặn gửi" không áp dụng. Không có mã `P00xx`.

## Interaction Patterns

- **Bấm tên tài khoản mở chi tiết** `SCR004_AccountDetail` — source: `app/(app)/accounts/page.tsx:62-66`.
- **"Áp dụng"/"Xoá lọc" tải lại trang bằng GET** (không cần JS client) — source: `components/ui/filter-bar.tsx:27-64`.
- Không có yêu cầu bàn phím riêng.

## Accessibility

| Aspect | Status | Notes |
|--------|--------|-------|
| ARIA roles/labels | partial | `<label htmlFor>` cho select (`filter-bar.tsx:34`); `<th scope="col">` (`data-table.tsx:35`) |
| Keyboard navigation | supported | link/select/nút theo thứ tự DOM |
| Focus management | unmanaged | trang tải lại hoàn toàn khi lọc |
| Screen reader compatibility | unknown | bảng ngữ nghĩa chuẩn `<table>`; chưa kiểm bằng trình đọc |

## Conditional Rendering

| Condition | Type | Renders | Hidden | Notes |
|-----------|------|---------|--------|-------|
| role ≠ `sys_admin` | auth | — | cả route | `proxy.ts` đẩy `/403`; mục menu "Tài khoản nội bộ" cũng không hiện (`app-nav.tsx:17`) |
| `rows.length === 0` | ui-state | `EmptyState` | `<table>` | `data-table.tsx:24-26` |
| `status === "active"` | ui-state | badge positive | badge danger | `accounts/page.tsx:76` |

## Component Variants

Không áp dụng — màn không có component đổi hình dạng theo biến thể; khác biệt theo chức danh/trạng thái đã mô tả ở Conditional Rendering.

## Security Surface

| Guard | Type | Consequence if bypassed |
|-------|------|------------------------|
| `ROUTE_POLICY { prefix: "/accounts", roles: ["sys_admin"] }` — `proxy.ts`/`canAccess()` | role | role khác bị đẩy `/403` (`lib/auth/route-policy.ts:25`, `lib/supabase/proxy.ts:83-87`) |
| RLS `app_users`: mọi role tự xem dòng của mình (`app_users_select_self`), chỉ `sys_admin` xem **toàn bộ** (`app_users_select_all_sys_admin`) | permission | role khác gọi thẳng PostgREST chỉ thấy 1 dòng — `supabase/migrations/0009_rls_policies.sql:20-23` |
| `page.tsx` chỉ kiểm đăng nhập (`if (!user) redirect("/login")`), không kiểm role | auth | dựa vào 2 tầng trên — `accounts/page.tsx:22-23` |

Chặn ở **cả hai tầng**: route + RLS (khớp đúng nhau). Không có thao tác ghi để cần tầng thứ ba.

## Source References

1. Page/View: `app/(app)/accounts/page.tsx:21-104`
2. Truy vấn: `lib/domain/accounts.ts:40-56`
3. Component dùng chung: `components/ui/filter-bar.tsx:25-66`, `components/ui/data-table.tsx:23-63`, `components/ui/status-badge.tsx:16-24`
4. Phân quyền: `lib/auth/route-policy.ts:25`; RLS `supabase/migrations/0009_rls_policies.sql:20-23`
5. i18n: `lib/i18n/{vi,ja}/accounts.ts`, `lib/i18n/{vi,ja}/common.ts` (`roles`, `userStatus`)

## Source Walkthrough

1. **File:** `lib/domain/accounts.ts:6-56` — kiểu `AccountRow` và `listAccounts`: dữ liệu đọc ra là gì.
2. **File:** `app/(app)/accounts/page.tsx:21-104` — trang: đọc `searchParams`, dựng cột/bộ lọc.
3. **File:** `components/ui/filter-bar.tsx:25-66` — GET form dùng chung.

### Call Hierarchy

```text
AccountsPage -> listAccounts -> app_users (select, RLS) ; AccountsPage -> FilterBar + DataTable -> StatusBadge/Link
```

### Read Calls *(mục bổ sung)*

| Thao tác | Cách gọi | Tham số | Trả về |
|---|---|---|---|
| Đọc danh sách | `select` từ `app_users` qua `listAccounts()` (`lib/domain/accounts.ts`) | `role?` (`.eq`), `status?` (`.eq`) | mảng `AccountRow`, sắp `full_name` tăng dần |

Select thẳng qua PostgREST, không qua RPC — đọc thuần, RLS đã đủ chặn.

### Dependencies & Risks *(mục bổ sung)*

- Phụ thuộc `app_users` đã có dữ liệu seed (tối thiểu 4 tài khoản demo).
- Cột "Ghi" ở ma trận quyền là ✗ vì bản thân `SCR003` không có form ghi — đừng nhầm với `SCR004` (cùng `sys_admin`), nơi có ghi thật qua RPC.
- `sys_admin` không mở được màn nghiệp vụ nào (`SCR007`..`SCR018`) là quyết định kiến trúc có chủ ý, không phải lỗ hổng khi review ma trận quyền (ADR-001).
- Ma trận quyền (9 role, PREMISE C-09): `sys_admin` xem ●/ghi ✗; `receiving`, `trading`, `reconcile` ✗/✗; 5 role đề xuất (`party_admin`, `rule_admin`, `ops_admin`, `market_manager`, `internal_audit`) ✗/✗. Cờ `can_approve` không ảnh hưởng quyền mở màn này.
- Link chéo: [F001](../../features/F001_AdminAndPermissions/screens.md).

### Unresolved Questions *(mục bổ sung)*

Không phân trang — danh sách tải toàn bộ `app_users` một lần. Chấp nhận được ở quy mô demo hiện tại (vài chục dòng); cần quyết định ngưỡng phân trang nếu số tài khoản tăng đáng kể.

Cột `can_approve` chưa có trong bảng/bộ lọc (bản đã xây không có cờ này, PREMISE C-09): khi làm thật cần thêm cột và bộ lọc "có quyền duyệt" để `sys_admin` thấy ai đang giữ cờ.
