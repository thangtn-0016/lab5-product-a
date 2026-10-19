# SCR002_Login — Screen Spec

> Alias LAB-1: **SC-02** · ログイン · Đăng nhập

**Screen**: SCR002_Login: Đăng nhập
**Feature**: F001_AdminAndPermissions — [screens.md](../../features/F001_AdminAndPermissions/screens.md)
**Type**: atomic
**Route**: `/login` (query tuỳ chọn `?reason=suspended`)
**Generated**: 2026-10-06
**Trạng thái**: Đã xây ở LAB-3
**Tính năng LAB-1**: `F01-01` `F01-02` *(`F01-02` — xác thực hai lớp — **chưa xây**, ngoài phạm vi LAB-3; thiết kế MFA phân tầng đã chốt ở PREMISE C-04: MFA cho admin và các role duyệt nội bộ, bên ngoài single-factor + device binding; prototype vẫn một lớp)*
**User story**: không có US riêng trong (tài liệu gốc trong repo, không nộp) `product-a-user-stories.md` — màn nền (xác thực), không nằm trong chuỗi US-01..US-15
**Ảnh**: [VI](../../design/screenshots/sc02-login-vi.png) · [JA](../../design/screenshots/sc02-login-ja.png)

## Purpose

Cổng vào duy nhất của toàn hệ thống — mọi URL khác đều bị `proxy.ts` đẩy về đây nếu chưa có phiên đăng nhập hợp lệ. Dùng bởi mọi chức danh (4 role đã có trong prototype; bộ spec chốt 9 role, PREMISE C-09), vào đầu ca (cửa sổ dịch vụ 02:00–10:00 JST theo RFP). Màn này được viết ở Phase 2, **trước khi có i18n** — namespace `auth` (`lib/i18n/{vi,ja}/auth.ts`) được thêm sau để vá song ngữ, không thuộc namespace `common`.

## Screen Layout

Màn nằm **ngoài** route group `(app)` nên không có header/nav: một cột giữa trang, nền `zinc-50`, khung rộng tối đa `max-w-sm` gồm tiêu đề + mô tả, (tuỳ chọn) dòng cảnh báo `suspended`, rồi form 2 ô + nút (`app/login/page.tsx:17-34`). Đơn giản một cột, không có breakpoint riêng.

### Layout Sketch

```
┌──────────────────────────────────────┐
│ R1: Khung giữa trang (flex center)   │
│  ┌────────────────────────────────┐  │
│  │ R2: Tiêu đề + mô tả            │  │
│  │ R3: Banner reason=suspended - -│  │ (chỉ khi ?reason=suspended)
│  │ R4: LoginForm (Email, Mật khẩu,│  │
│  │     banner lỗi, nút Đăng nhập) │  │
│  └────────────────────────────────┘  │
└──────────────────────────────────────┘
```

### Layout Regions

| Region ID | Name | Position | Scrollable | Key Components | Responsive Behavior |
|-----------|------|----------|------------|----------------|---------------------|
| R1 | Khung trang | static | no | `div.flex items-center justify-center` | fluid, `max-w-sm` cho cột |
| R2 | Tiêu đề | static | no | `h1`, `p` (`t.auth.login.title/subtitle`) | always |
| R3 | Banner tài khoản bị khoá | static (có điều kiện) | no | `<p role="alert">` | chỉ khi `reason === "suspended"` |
| R4 | Form đăng nhập | static | no | `LoginForm` (`useActionState`) | always |

## Child Routes

Không áp dụng — màn này là một route đơn, không phải khung chứa route con.

## User Flow

> **Scope:** within-screen interactions only. Điều hướng chéo màn: [screen-flow.md](../../generated/screen-flow.md).

### Happy Path

1. Người dùng mở bất kỳ URL nào khi chưa có phiên → `proxy.ts` redirect `/login` (`lib/supabase/proxy.ts:58-63`).
2. Form hiện, focus tự động ở ô Email (R4).
3. Gõ email, Tab/click sang Mật khẩu, gõ mật khẩu.
4. Enter (ở ô bất kỳ) hoặc click "Đăng nhập" → submit.
5. Supabase tạo session → `redirect("/")` → vào `SCR001_RoleHome` theo đúng role (`app/login/actions.ts:36`).

### Branches

| Decision point | Condition | Outcome on this screen | Source |
|----------------|-----------|------------------------|--------|
| Bước 5 | `signInWithPassword` trả lỗi | banner đỏ dưới ô mật khẩu, ở lại form, **dữ liệu gõ không mất** | `app/login/actions.ts:32-34` |
| Bước 4 | email/password rỗng sau ràng buộc `required` của trình duyệt | banner `missingCredentials` | `app/login/actions.ts:25-27` |
| Bị khoá | tài khoản đang có session bị chuyển `suspended` → request kế tiếp bất kỳ bị `proxy.ts` phát hiện → tự `signOut()` → redirect `/login?reason=suspended` | banner cảnh báo riêng hiện phía trên form | `lib/supabase/proxy.ts:68-80`, `app/login/page.tsx:26-30` |
| Đã đăng nhập mở lại `/login` | có claims + path công khai | proxy **cho qua thẳng**, form vẫn hiện (không tự đá về `/`) | `lib/supabase/proxy.ts:65-66` |

## Data Inventory

| Field | Display Label | Source | Format | Empty Behavior | Cross-ref |
|-------|---------------|--------|--------|----------------|-----------|
| `t.auth.login.title`, `subtitle` | tiêu đề + mô tả | dictionary `auth` | text theo locale | không áp dụng | `lib/i18n/{vi,ja}/auth.ts` |
| `reason` | banner "Tài khoản đã bị vô hiệu hoá. Liên hệ quản trị hệ thống." | query param `?reason=suspended` do `proxy.ts` gắn | text | ẩn khi không có tham số | `app_users.status` |

Hai ô nhập (email, password) thuộc `## Validation & Error Feedback`.

## UI States

| State | Trigger | Visual Behavior | User Action Available | Source |
|-------|---------|----------------|-----------------------|--------|
| empty (mặc định) | mọi lần vào trang | form trống, focus tự động vào ô Email | nhập | `login-form.tsx:31` |
| saving | sau khi bấm submit, trước khi Server Action trả kết quả | nút disabled, đổi nhãn "Đang đăng nhập..." / "ログイン中..." (`t.submitPending`), không có spinner | none | `login-form.tsx:58-64` |
| success | `signInWithPassword` không lỗi | `redirect("/")` ngay trong Server Action — người dùng không thấy banner thành công nào trên chính trang này | — | `actions.ts:36` |
| error (nghiệp vụ) | `missingCredentials` / `invalidCredentials` | banner đỏ dưới ô mật khẩu; **cả hai ô giữ nguyên nội dung đã gõ** — không có lệnh reset trong `login-form.tsx`, không có full-page reload | sửa & gửi lại | `login-form.tsx:52-56` |
| error (mạng) | request tới Server Action không tới được server (mất mạng khi bấm submit) | **không được xử lý riêng** — không có `try/catch` quanh lời gọi transport, không có `error.tsx` trong cây route này; hành vi thực tế không xác định được chỉ từ đọc code (xem `### Unresolved Questions`) | — | `login-form.tsx:19` |
| suspended | `?reason=suspended` | banner cảnh báo phía trên `LoginForm` | đăng nhập lại / liên hệ admin | `page.tsx:26-30` |
| read-only | không áp dụng — đăng nhập không gắn với ngày nghiệp vụ, luôn ghi được | — | — | — |

## Validation & Error Feedback

### A) Client-side

| Field | Type | Required | Constraints | Async Check | Error Message |
|-------|------|----------|-------------|-------------|---------------|
| `email` | text (`type="email"`, `autoFocus`, `autoComplete="username"`) | yes | không rỗng; `type="email"` chỉ ép định dạng phía trình duyệt, Server Action chỉ kiểm "là string khác rỗng", không kiểm lại định dạng | không | (trình duyệt) / `missingCredentials` |
| `password` | password (`autoComplete="current-password"`) | yes | không rỗng; **không có ràng buộc độ dài/độ phức tạp** ở cả client lẫn server lúc đăng nhập (độ phức tạp chỉ áp dụng lúc Supabase Auth tạo tài khoản, không kiểm lại khi sign-in) | không | như trên |

#### Field Labels & Defaults *(mục bổ sung)*

| Field | Nhãn VI | Nhãn JA | Mặc định | Nguồn dữ liệu |
|---|---|---|---|---|
| `email` | Email | メールアドレス | rỗng | input người dùng |
| `password` | Mật khẩu | パスワード | rỗng | input người dùng |

#### Mã lỗi hiển thị

| Mã | Điều kiện kích hoạt | Thông báo VI | Thông báo JA | Hiện ở đâu | Chặn gửi? |
|---|---|---|---|---|:---:|
| `missingCredentials` | `email` hoặc `password` rỗng/không phải string (kiểm trong Server Action `signIn()`, SAU ràng buộc `required` của trình duyệt) | "Vui lòng nhập đầy đủ email và mật khẩu." | "メールアドレスとパスワードを入力してください。" | `<p role="alert">` dưới ô mật khẩu, trên nút submit | ● |
| `invalidCredentials` | `supabase.auth.signInWithPassword()` trả lỗi (sai email, sai mật khẩu, hoặc email không tồn tại — **cả ba trả CÙNG một thông báo**, không phân biệt) | "Email hoặc mật khẩu không đúng." | "メールアドレスまたはパスワードが正しくありません。" | như trên | ● |
| (không mã) `reason=suspended` | query param do `proxy.ts` gắn khi `app_users.status` khác `active` tại request kế tiếp (không phải lỗi submit — xảy ra TRƯỚC khi vào lại form) | "Tài khoản đã bị vô hiệu hoá. Liên hệ quản trị hệ thống." | "アカウントが無効化されています。システム管理者にお問い合わせください。" | `<p role="alert">` phía trên `LoginForm` | N/A — không phải submit |

Màn này không dùng mã lỗi database `P00xx`.

### B) Server-side

#### Đăng nhập
- **Endpoint:** Server Action `signIn` (`app/login/actions.ts:17-37`) → `supabase.auth.signInWithPassword({ email, password })`
- **Request:** `email`, `password` từ `FormData`
- **Success:** session set qua cookie → `redirect("/")`
- **Errors:** `{ error: t.auth.errors.missingCredentials | invalidCredentials }` — **không `throw`**
- **Trigger:** Enter ở ô bất kỳ hoặc click nút submit (hành vi mặc định của `<form>`)
- **Source:** `app/login/actions.ts:17-37`

#### Đăng xuất *(gọi từ header ở màn khác, ghi nhận để đầy đủ luồng xác thực)*
- **Endpoint:** Server Action `signOut` → `supabase.auth.signOut()` → `redirect("/login")`
- **Source:** `app/login/actions.ts:40-44`

#### Kiểm phiên + role mỗi request *(không phải hành động UI)*
- `lib/supabase/proxy.ts`: `supabase.auth.getClaims()` rồi `select role, status from app_users where id = claims.sub` → dùng cho `canAccess()` và kiểm `status==='active'` (`proxy.ts:53-87`). Toàn bộ qua Supabase Auth SDK, không có RPC tuỳ biến.

## Interaction Patterns

- **Ô Email nhận focus ngay khi mở trang** — source: `app/login/login-form.tsx:31` (`autoFocus`).
- **Enter ở ô bất kỳ submit form** theo hành vi mặc định của `<form>` (không viết tay) — source: `login-form.tsx:14-15`, `:22`.
- **Không yêu cầu bàn phím riêng** (không thuộc 3 màn ⌨️ bắt buộc `SCR007`/`SCR010`/`SCR014`).

## Accessibility

| Aspect | Status | Notes |
|--------|--------|-------|
| ARIA roles/labels | partial | `<label htmlFor>` cho cả 2 ô (`login-form.tsx:24,39`); banner lỗi `role="alert"` (`:53`, `page.tsx:27`) |
| Keyboard navigation | supported | Tab theo DOM: Email → Mật khẩu → nút |
| Focus management | partial | `autoFocus` ở Email; thất bại không chuyển focus, không có focus trap |
| Screen reader compatibility | unknown | `autoComplete="username"/"current-password"` cho trình quản lý mật khẩu; chưa kiểm bằng trình đọc |

## Conditional Rendering

| Condition | Type | Renders | Hidden | Notes |
|-----------|------|---------|--------|-------|
| `reason === "suspended"` | auth | banner "Tài khoản đã bị vô hiệu hoá…" | — | `app/login/page.tsx:26-30` |
| `state.error` có giá trị | ui-state | `<p role="alert">` | — | `login-form.tsx:52-56` |
| `isPending` | ui-state | nhãn `submitPending`, nút `disabled` | nhãn `submit` | `login-form.tsx:60,63` |

## Component Variants

Không áp dụng — màn không có component đổi hình dạng theo biến thể; khác biệt theo chức danh/trạng thái đã mô tả ở Conditional Rendering.

## Security Surface

| Guard | Type | Consequence if bypassed |
|-------|------|------------------------|
| `PUBLIC_PATHS = ["/login"]` — **mục duy nhất** công khai | auth | thêm path vào đây = mở route không cần đăng nhập — `lib/supabase/proxy.ts:5-9` |
| Chưa đăng nhập → redirect `/login` | auth | `proxy.ts:58-63` |
| `app_users.status ≠ active` hoặc thiếu dòng `app_users` → `signOut` + `/login?reason=suspended` | auth | `proxy.ts:74-80` — **thông báo "đã vô hiệu hoá" gây hiểu lầm khi thực chất là thiếu dữ liệu** (xem Risks) |
| Không chặn theo chức danh; không rate limit/khoá tạm sau N lần sai (mặc định Supabase Auth) | — | brute-force chỉ bị giới hạn bởi Supabase Auth |

## Source References

1. Page/View: `app/login/page.tsx:8-36`
2. Form: `app/login/login-form.tsx:18-67`
3. Server Actions: `app/login/actions.ts:17-44`
4. Chặn route + phiên: `proxy.ts:9-17`, `lib/supabase/proxy.ts:19-91`
5. Lấy user hiện tại: `lib/auth/current-user.ts:17-39`
6. i18n: `lib/i18n/vi/auth.ts`, `lib/i18n/ja/auth.ts`

## Source Walkthrough

1. **File:** `lib/supabase/proxy.ts:5-91` — bắt đầu từ đây: định nghĩa path công khai và mọi nhánh redirect của luồng xác thực.
2. **File:** `app/login/actions.ts:17-44` — Server Action `signIn`/`signOut`.
3. **File:** `app/login/login-form.tsx:18-67` — form `useActionState`.
4. **File:** `app/login/page.tsx:8-36` — trang, đọc `searchParams` (Promise ở Next.js 16, phải `await`).

### Call Hierarchy

```text
LoginPage -> LoginForm -> signIn (Server Action) -> supabase.auth.signInWithPassword -> redirect("/")
proxy.ts -> updateSession -> getClaims -> app_users(role,status) -> canAccess | redirect
```

### Dependencies & Risks *(mục bổ sung)*

- Phụ thuộc `app_users` có đúng 1 dòng ứng với mỗi `auth.users` (seed tạo cả hai cùng lúc). Thiếu dòng `app_users`, `proxy.ts` coi như `!appUser` → tự `signOut()` + redirect `?reason=suspended` — **thông báo "đã vô hiệu hoá" gây hiểu lầm** cho trường hợp thiếu dữ liệu.
- Người đã đăng nhập mở lại `/login` **không** bị đá về `/` — dễ bị hiểu nhầm là bug nếu không đọc đoạn `isPublicPath` trong `lib/supabase/proxy.ts:65-66`.
- Không có `error.tsx`/thử lại tay cho trường hợp mất mạng ngay lúc submit.
- Không có rate limit/khoá tạm sau N lần sai liên tiếp — Supabase Auth mặc định, không có lớp tự viết thêm.
- Liên kết: ADR-001 (chức danh đọc từ bảng `app_users`, không nhét vào JWT) — lý do `proxy.ts` phải truy vấn `app_users` mỗi request.

### Unresolved Questions *(mục bổ sung)*

- Hành vi khi mất kết nối mạng đúng lúc bấm "Đăng nhập" chưa được định nghĩa (không có banner "mất mạng" riêng như các màn ⌨️ khác, ví dụ `SCR007`/`SCR010`) — cần quyết định: hiện thông báo gì, nút có tự trở lại trạng thái bấm được không.
- Có nên đổi hành vi "đã đăng nhập mở `/login`" sang tự động `redirect("/")`, hay giữ nguyên cho qua thẳng như hiện tại?
- Không có cơ chế khoá tạm sau nhiều lần đăng nhập sai — cần hay không nằm ngoài phạm vi đã quyết ở LAB-3, nhưng đáng nêu nếu làm thật.
- MFA phân tầng (PREMISE C-04) chưa có bước nhập mã thứ hai trên màn này: cần bổ sung luồng nhập mã/đăng ký thiết bị khi làm thật, và nêu rõ cách xử lý người thuộc nhóm bắt buộc MFA (admin, role duyệt nội bộ) mà chưa đăng ký.
