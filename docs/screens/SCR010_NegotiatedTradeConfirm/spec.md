# SCR010_NegotiatedTradeConfirm — Screen Spec

> Alias LAB-1: **SC-10** · 相対取引の確定 · Chốt giao dịch thỏa thuận

**Screen**: SCR010_NegotiatedTradeConfirm: Chốt giao dịch thỏa thuận
**Feature**: F004_TradeRecording — [screens.md](../../features/F004_TradeRecording/screens.md)
**Type**: composite *(REG001 form chốt · REG002 vùng tham khảo lô/người mua — xem [screen-list.md](../../generated/screen-list.md))*
**Route**: `/trades/negotiated`
**Generated**: 2026-10-06
**Trạng thái**: Đã xây ở LAB-3 · ⌨️ bàn phím bắt buộc · cần có mạng (không có đường offline)
**Tính năng LAB-1**: `F04-01` `F04-02` `F04-04` *(chốt giao dịch 相対取引 + cổng chốt)*
**User story**: `US-03` (chốt giao dịch thỏa thuận), `US-04` (bị chặn khi người mua hết hạn — `P0003`), `US-05` (bị chặn khi lô không đủ số lượng — `P0004`)
**Ảnh**: [VI](../../design/screenshots/sc10-trade-negotiated-vi.png) · [JA](../../design/screenshots/sc10-trade-negotiated-ja.png) · [vùng tham khảo](../../design/screenshots/sc10-preview-vi.png) · [từ chối P0003](../../design/screenshots/sc10-reject-p0003-vi.png) · [từ chối P0004](../../design/screenshots/sc10-reject-p0004-vi.png)

## Purpose

Chốt một giao dịch thỏa thuận (相対取引, chiếm ~90% khối lượng theo (tài liệu gốc trong repo, không nộp) `architecture.md` §2.1): người bàn điều hành (`trading`) nhập mã lô, mã người mua, số lượng, đơn giá rồi xác nhận. Đây là **cổng chống bán vượt lô** (FR-AITAI-02) — một lô cá là vật thể duy nhất đã lên bờ, không bổ sung được, nên hai phép kiểm nghiệp vụ (tư cách người mua, số lượng còn lại) **bắt buộc chạy trong một transaction phía server với khoá dòng**, không phải ở client (ADR-002, ADR-004). Thao tác diễn ra ở bàn điều hành (05:00–08:00 JST), nhiều giao dịch liên tiếp trong ca — form phải xoá sạch sau mỗi lần chốt và không tự điền lại dữ liệu lần trước (máy dùng chung theo ca).

## Screen Layout

Khung `AppLayout`; nội dung xếp dọc `gap-6` trong một container bắt phím (`onKeyDown`): `PageHeader`, banner thành công (`role="status"`), banner lỗi (`role="alert" aria-live="assertive"`, tự nhận focus), dòng gợi ý phím tắt, khối form 4 ô + nút (`max-w-lg`), rồi **vùng tham khảo** lưới 2 cột (lô | người mua) + dòng "Thành tiền tạm tính" + ghi chú "Thông tin tham khảo…" (`app/(app)/trades/negotiated/negotiated-form.tsx:53-147`). Vùng tham khảo không có input/button nào nên ngoài tab order.

### Layout Sketch

```
┌────────────────────────────────────────────────────────────┐
│ R1: AppHeader + R2: AppNav (khung chung)                   │
├────────────────────────────────────────────────────────────┤
│ R3: PageHeader "Chốt giao dịch thỏa thuận" + mô tả         │
│ R4: Banner thành công (xanh) - - - - - - - - - - - - - - - │
│ R5: Banner lỗi (đỏ, tự nhận focus) - - - - - - - - - - - - │
│ R6: Gợi ý phím tắt (xs)                                    │
│ R7: Form  Mã lô* → Mã người mua* → Số lượng* → Đơn giá*    │
│          [Chốt giao dịch]                                  │
│ R8: Vùng tham khảo  ┌ Thông tin lô ┐ ┌ Thông tin người mua ┐│
│                     └──────────────┘ └─────────────────────┘│
│                     Thành tiền tạm tính  ·  ghi chú tham khảo│
└────────────────────────────────────────────────────────────┘
```

### Layout Regions

| Region ID | Name | Position | Scrollable | Key Components | Responsive Behavior |
|-----------|------|----------|------------|----------------|---------------------|
| R1/R2 | Header + Nav | static | no | khung chung | — |
| R3 | Tiêu đề | static | no | `PageHeader` | always |
| R4 | Banner thành công | static (có điều kiện) | no | `div[role=status]` | state riêng, không bị `revalidatePath` thổi bay |
| R5 | Banner lỗi | static (có điều kiện) | no | `div[role=alert][tabIndex=-1]` | tự `.focus()` khi có lỗi |
| R6 | Gợi ý phím tắt | static | no | `<p>` `t.negotiated.keyboardHint` | always |
| R7 | Form chốt (REG001) | static | no | 4 `Field` + nút | `max-w-lg` |
| R8 | Vùng tham khảo (REG002) | static | no | `LotBuyerPreview` | `sm:grid-cols-2`; dưới `sm` 1 cột |

## Child Routes

Không áp dụng — màn này là một route đơn, không phải khung chứa route con.

## User Flow

> **Scope:** within-screen interactions only. Điều hướng chéo màn: [screen-flow.md](../../generated/screen-flow.md).

### Happy Path

1. Trang tải, focus vào ô mã lô.
2. Gõ mã lô → Tab (rời ô kích hoạt `handleLotBlur`: tra `v_lot_availability` + `lots.status` + `commodities`, hiện vùng "Thông tin lô (tham khảo)").
3. Gõ mã người mua → Tab (rời ô kích hoạt `handleBuyerBlur`: tra `participant_qualifications`, hiện vùng "Thông tin người mua (tham khảo)").
4. Gõ số lượng → Tab (vùng "Thành tiền tạm tính" cập nhật ngay khi cả số lượng và đơn giá hợp lệ).
5. Gõ đơn giá → `Enter` (hoặc Tab tới nút rồi bấm, hoặc `Ctrl+Enter` từ bất kỳ đâu).
6. Client kiểm định dạng (`validateFormat`) — rỗng hoặc không dương → banner lỗi, dừng, không gọi server.
7. Server Action `confirmTrade` tự kiểm role → tra `lotCode`/`buyerCode` thành `id` → không tìm thấy → `P0001`.
8. Gọi RPC `confirm_negotiated_trade` trong transaction: khoá dòng lô (`FOR UPDATE`) → kiểm ngày đã chốt chưa (`P0002`) → kiểm tư cách người mua (`P0003`) → kiểm số lượng còn lại (`P0004`) → nếu qua hết: insert `trades` + cập nhật `lots.status = '確定'` nếu `remaining = 0`.
9. **Nhánh thành công:** banner xanh hiện `trade_code` + số còn lại, form xoá sạch, focus về ô mã lô, quay lại bước 2.

### Branches

| Decision point | Condition | Outcome on this screen | Source |
|----------------|-----------|------------------------|--------|
| Bước 6 | định dạng sai | banner `V-format`/`V-positive`, không gọi server | `use-negotiated-form.ts:53-72` |
| Bước 7–8 | bị từ chối (P0001/P0002/P0003/P0004/P0005/generic) | banner đỏ tự nhận focus, dữ liệu 4 field **giữ nguyên** để sửa rồi gửi lại (ví dụ giảm số lượng sau `P0004`) | `use-negotiated-form.ts:93-103` |
| Bước 7–8 | mất mạng (`fetch` throw) | cùng banner đỏ, thông báo mạng riêng; **4 field KHÔNG bị xoá** | `use-negotiated-form.ts:104-106` |
| Bước 2/3 | mã không tồn tại | vùng tham khảo hiện "không tìm thấy" (đỏ) — **không chặn submit** | `lot-buyer-preview.tsx:65-67,93-95` |
| Bất kỳ lúc nào | `Escape` | xoá 4 field + vùng tham khảo, focus về ô mã lô | `use-negotiated-form.ts:132-136` |

## Data Inventory

> Vùng tham khảo (REG002) **chỉ hiển thị thông tin**, không chặn hay cho phép submit — comment `lot-buyer-preview.tsx:19-25`: "TUYỆT ĐỐI không thêm `if (remaining < qty)` hay bất kỳ điều kiện chặn submit nào". Giá trị có thể đã cũ tại thời điểm bấm chốt (race hai người chốt cùng lúc) — đúng dòng chữ trong ảnh `sc10-preview-vi.png`: "Thông tin tham khảo — hệ thống vẫn kiểm tra lại đầy đủ khi bấm chốt, kết quả cuối cùng có thể khác nếu dữ liệu vừa thay đổi."

| Field | Display Label | Source | Format | Empty Behavior | Cross-ref |
|-------|---------------|--------|--------|----------------|-----------|
| Mặt hàng (#5) | Mặt hàng / 品目 | `commodities.name_vi`/`name_ja` qua `v_lot_availability` → `lots.commodity_id` (`trade-preview.ts:26-63`) | text theo locale | ẩn khi chưa tra | `commodities` |
| Trạng thái lô (#6) | Trạng thái lô / ロット状態 | `lots.status` qua `StatusBadge` + `t.common.lotStatus` (tone `neutral`) | badge | — | `lots.status` |
| Còn lại (#7) | Còn lại / 残数量 | `v_lot_availability.remaining = lots.quantity − Σ trades.quantity (status='confirmed')` | `toLocaleString()` + đơn vị | — | `v_lot_availability` |
| Tên người mua (#8) | Tên / 氏名 | `participants.name` | text | — | `participants` |
| Tư cách tham gia (#9) | (badge) | `qualified = true` nếu có dòng `participant_qualifications` với `status='active'` và ngày hôm nay trong `[valid_from, valid_to]`; ngược lại hiện "Tư cách đã hết hạn từ {expired_on}" (`trade-preview.ts:105-125`) | badge positive/danger | — | `participant_qualifications` |
| Thành tiền tạm tính (#10) | Thành tiền tạm tính / 概算金額 | `quantity × unitPrice`, tính phía client; chỉ hiện khi cả hai ô khác rỗng và là số hữu hạn (`Number.isFinite`, kể cả âm hoặc 0 — việc kiểm dương chỉ chạy lúc submit, nên nhập `-5` vẫn hiện thành tiền âm) (`use-negotiated-form.ts:139-142`) | `toLocaleString()` | ẩn | tính phía client |
| ghi chú tham khảo | `t.negotiated.previewNote` | dictionary | text xs | — | — |

## UI States

| State | Trigger | Visual Behavior | User Action Available | Source |
|-------|---------|----------------|-----------------------|--------|
| empty | mở trang lần đầu / sau khi chốt thành công | 4 ô trắng, vùng tham khảo trống, focus ở ô mã lô | nhập | `use-negotiated-form.ts:44-51` |
| loading (preview) | đang tra lô/người mua sau `onBlur` | "đang tra…" trong từng thẻ tham khảo | none | `lot-buyer-preview.tsx:41,72` |
| saving | `isPending` (đang gọi RPC) | nút "Chốt giao dịch" đổi nhãn "Đang chốt…", disabled | none | `negotiated-form.tsx:126-133` |
| success | `confirm_negotiated_trade` trả `{trade_code, remaining_after}` | banner xanh `role="status"`: "Đã chốt giao dịch {trade_code}. Lô còn lại {remaining} {unit}."; form xoá sạch cả 4 field + vùng tham khảo; focus quay về ô mã lô | nhập giao dịch kế | `use-negotiated-form.ts:82-91` |
| error (nghiệp vụ) | RPC trả lỗi (P0001–P0005) hoặc validate định dạng thất bại | banner đỏ `role="alert" aria-live="assertive"`, **tự nhận focus** (`tabIndex={-1}` + `.focus()` trong `useEffect`) để trình đọc màn hình đọc ngay — môi trường hiện trường ồn, lỗi không được chỉ "hiện ra" im lặng | sửa & gửi lại | `use-negotiated-form.ts:40-42`, `negotiated-form.tsx:68-78` |
| error (mạng) | `fetch` throw trong `startTransition` | cùng banner đỏ, thông báo riêng; **dữ liệu 4 field KHÔNG bị xoá** — chỉ cần bấm chốt lại | bấm chốt lại | `use-negotiated-form.ts:104-106` |
| read-only | ngày nghiệp vụ của lô đã chốt sổ | không có giao diện chỉ-đọc riêng — submit vẫn cho phép gõ, RPC trả `P0002`, banner lỗi hiện như mọi lỗi nghiệp vụ khác | — | `0007_rpc_functions.sql:65-69` |

## Validation & Error Feedback

### A) Client-side

> `validateFormat()` trong `use-negotiated-form.ts:53-63` **chỉ kiểm định dạng** (rỗng, không phải số dương) — bình luận trong code: "KHÔNG BAO GIỜ kiểm tư cách người mua hay số lượng lô còn lại" (`:20-23`).

| Field | Type | Required | Constraints | Async Check | Error Message |
|-------|------|----------|-------------|-------------|---------------|
| `lotCode` | text | yes | không rỗng (định dạng); phải tồn tại trong `lots.lot_code` (kiểm ở Server Action khi submit, không kiểm khi gõ) | `onBlur` → `lookupLotPreview` (chỉ tham khảo) | `V-format` / `P0001` |
| `buyerCode` | text | yes | không rỗng (định dạng); phải tồn tại trong `participants.code` | `onBlur` → `lookupBuyerPreview` (chỉ tham khảo) | `V-format` / `P0001` |
| `quantity` | number (`step=0.001`) | yes | `> 0` | không | `V-format` / `V-positive` |
| `unitPrice` | number (`step=0.01`) | yes | `> 0` | không | `V-format` / `V-positive` |

#### Field Labels & Defaults *(mục bổ sung)*

| # | Field | Nhãn VI | Nhãn JA | Mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|
| 1 | `lotCode` | Mã lô | ロット番号 | rỗng | nhập tay → tra `lots.id` |
| 2 | `buyerCode` | Mã người mua | 買参人コード | rỗng | nhập tay → tra `participants.id` |
| 3 | `quantity` | Số lượng | 数量 | rỗng | nhập tay → `trades.quantity` |
| 4 | `unitPrice` | Đơn giá | 単価 | rỗng | nhập tay → `trades.unit_price` |

#### Mã lỗi hiển thị

| Mã | Điều kiện kích hoạt | Thông báo VI | Thông báo JA | Hiện ở đâu | Chặn gửi? |
|---|---|---|---|---|:---:|
| `V-format` | một trong 4 field bắt buộc rỗng | Vui lòng nhập đủ mã lô, mã người mua, số lượng và đơn giá. | ロット番号・買参人コード・数量・単価をすべて入力してください。 | banner `aria-live="assertive"` đầu vùng form | ● |
| `V-positive` | `quantity`/`unitPrice` không phải số hoặc `≤ 0` | Số lượng và đơn giá phải là số dương. | 数量と単価は正の数で入力してください。 | banner `aria-live="assertive"` | ● |
| `P0001` | mã lô hoặc mã người mua không tra được `id` (tra ở Server Action, trước khi gọi RPC) | Không tìm thấy mã lô hoặc mã người mua đã nhập. Kiểm tra lại hai mã này. | 入力されたロット番号または買参人コードが見つかりません。両方のコードを確認してください。 | banner `aria-live="assertive"` | ● |
| `P0002` | `business_days.status='closed'` cho ngày của lô. `{closed_at}` định dạng **UTC** `YYYY-MM-DD HH:mm` (`toISOString`, `trade-rejection-message.ts:17-22`), không đổi sang JST — lệch 9 giờ so với giờ nghiệp vụ; giới hạn đã biết | Ngày nghiệp vụ {business_date} đã chốt sổ lúc {closed_at}, không thể ghi thêm giao dịch. | 業務日{business_date}はすでに{closed_at}に締め処理されており、取引を追加できません。 | banner | ● |
| `P0003` | tư cách người mua hết hiệu lực tại `current_date` (kiểm **trong RPC**, dựng từ `error.details`: `expired_on, basis_type, basis_label, contact_office`; `{buyer_name}` lấy từ vùng tham khảo, không từ RPC — xem Unresolved Questions) | Người mua {buyer_name} đã hết hạn tham gia từ {expired_on}, căn cứ {basis_label}. Liên hệ {contact_office} để xử lý. | 買参人{buyer_name}の参加資格は{expired_on}に失効しています（根拠：{basis_label}）。{contact_office}にお問い合わせください。 | banner | ● |
| `P0004` | `remaining < quantity` (kiểm **trong RPC**, dựng từ `error.details`: `remaining, requested, sold_to[]`; `{unit}` lấy từ vùng tham khảo — xem Unresolved Questions) | Lô {lot_code} chỉ còn {remaining} {unit}, không đủ cho yêu cầu {requested} {unit}. Đã bán cho: {sold_to}. | ロット{lot_code}の残数量は{remaining} {unit}で、要求数量{requested} {unit}に不足しています。販売済み取引：{sold_to}。 | banner | ● |
| `P0005` | role caller ≠ `trading` (Server Action tự kiểm, hoặc RPC tự kiểm) | Chức danh của bạn không được phép thực hiện thao tác này. | この操作を行う権限がありません。 | banner | ● |
| (generic) | lỗi Postgres khác không map được | Có lỗi xảy ra khi gửi giao dịch, vui lòng thử lại. | 取引の送信中にエラーが発生しました。もう一度お試しください。 | banner | ● |
| (network) | `fetch`/Server Action throw (mất mạng) | Mất kết nối mạng — không thể gửi giao dịch. Kiểm tra lại kết nối rồi thử lại. Chưa có bản ghi nào được tạo. | ネットワーク接続が切断されました — 取引を送信できません。接続を確認して再度お試しください。取引は作成されていません。 | banner | ● |

**Then chốt (bắt buộc đọc đúng, LAB-5 dễ hiểu sai nếu bỏ qua):**

- `P0003` và `P0004` **tuyệt đối không bao giờ chạy phía client.** Hai phép kiểm này chạy **duy nhất** bên trong RPC `confirm_negotiated_trade`, trong **một transaction**, sau khi đã `SELECT ... FOR UPDATE` khoá dòng lô — xem sequence diagram ở `architecture.md` §2.2 (`supabase/migrations/0007_rpc_functions.sql:41-96`).
- Rẽ nhánh thông báo dựa vào `error.code` (SQLSTATE), **không bao giờ** so khớp `error.message` (`lib/domain/rpc-errors.ts:46-66`, `mapRpcError`). `error.details` là chuỗi JSON, phải `JSON.parse` trước khi đọc field.
- Khi locale = `ja`, `basis_label`/`contact_office` của `P0003` được tra lại bản tiếng Nhật qua `localizeP0003Details` (bảng `qualification_basis_rules.label_ja`/`contact_office_ja`) — vì RPC gốc luôn trả bản tiếng Việt trong `details`, không biết locale UI (`lib/domain/trade-preview.ts:127-149`).
- Vùng tham khảo (lô/người mua) **chỉ hiển thị thông tin**, không chặn submit (xem `## Data Inventory`).

### B) Server-side

#### Chốt giao dịch thỏa thuận
- **Endpoint:** Server Action `confirmTrade` (`app/(app)/trades/actions.ts:56-88`) → `rpc('confirm_negotiated_trade', { p_lot_id, p_buyer_id, p_quantity, p_unit_price })`
- **Request:** `lotCode, buyerCode, quantity, unitPrice` (server tra `lotCode`/`buyerCode` → `id` qua `findLotByCode`/`findParticipantByCode`)
- **Success:** `jsonb {trade_code, remaining_after}` → `revalidatePath("/trades")` + `("/lots")`; banner xanh, form xoá, focus ô mã lô
- **Errors:** `P0001` | `P0002` | `P0003` | `P0004` | `P0005` | `generic` | (network) — ánh xạ qua `buildTradeRejectionMessage` (`lib/domain/trade-rejection-message.ts:24-60`)
- **Trigger:** `Enter` ở ô đơn giá, nút "Chốt giao dịch", hoặc `Ctrl+Enter` từ bất kỳ đâu
- **Source:** `actions.ts:56-88`; RPC `0007_rpc_functions.sql:41-106`

#### Tra tham khảo lô / người mua *(không phải cổng kiểm tra)*
- **Endpoint:** Server Action `lookupLotPreview`, `lookupBuyerPreview` (`app/(app)/trades/negotiated/preview-actions.ts:20-48`) → `select` thẳng
- **Success:** `LotPreview | null`, `BuyerQualificationPreview | null`; lỗi hoặc `null` chỉ làm vùng tham khảo hiện "không tìm thấy"
- **Trigger:** `onBlur` ô mã lô / mã người mua

## Interaction Patterns

- ⌨️ **Thứ tự Tab:** Mã lô (autofocus) → Mã người mua → Số lượng → Đơn giá → nút "Chốt giao dịch" — source: `negotiated-form.tsx:83-133`.
- ⌨️ **`Enter` (không giữ Ctrl) chỉ submit khi đang đứng ở ô đơn giá** (`handlePriceFieldKeyDown`, gắn riêng trên input đó); Enter ở ba ô còn lại không có tác dụng đặc biệt — source: `use-negotiated-form.ts:118-123`.
- ⌨️ **`Ctrl+Enter` chốt từ bất kỳ đâu** trong vùng form, kể cả khi không có input nào được focus (ví dụ đang đứng trong banner lỗi) — bắt ở cấp container ngoài cùng (`onKeyDown` trên `<div>` bọc toàn bộ form + banner, không phải trên `<form>`) — source: `use-negotiated-form.ts:126-131`, `negotiated-form.tsx:53`.
- ⌨️ **`Escape` xoá toàn bộ 4 field + vùng tham khảo, focus quay về ô mã lô** — cùng cấp container với `Ctrl+Enter` — source: `use-negotiated-form.ts:132-136`.
- **Focus ban đầu:** ô mã lô (`autoFocus`) — `negotiated-form.tsx:89`. **Focus sau khi gửi:** thành công → tự quay về ô mã lô (`lotInputRef.current?.focus()`), sẵn sàng nhập giao dịch kế tiếp không rời bàn phím; thất bại → banner lỗi tự nhận focus (không phải input nào) để đọc được ngay bằng trình đọc màn hình — `use-negotiated-form.ts:40-42,89`.
- **Dialog chặn giữa luồng:** không có — toàn bộ phản hồi là banner inline, không phải modal.
- **Rời ô mã lô/mã người mua (`onBlur`) kích hoạt tra cứu tham khảo** — lệnh gọi mạng phụ, không phải một phần luồng Tab chính, nhưng xảy ra tự nhiên mỗi khi Tab rời hai ô đó — source: `use-trade-preview-lookup.ts:20-50`.
- **Banner lỗi và banner thành công dùng state riêng** (`errorMessage`, `successBanner`), không bị cuốn theo `revalidatePath` — chỉ đổi khi người dùng submit lần kế tiếp — source: `use-negotiated-form.ts:33-34`.

## Accessibility

| Aspect | Status | Notes |
|--------|--------|-------|
| ARIA roles/labels | present | lỗi `role="alert" aria-live="assertive"`; thành công `role="status" aria-live="polite"`; `Field` gắn `aria-describedby` (`components/ui/field.tsx:23-57`) |
| Keyboard navigation | supported | toàn bộ luồng không cần chuột (3 phím tắt ở trên) |
| Focus management | managed | autofocus ô 1; về ô 1 sau thành công; banner lỗi tự nhận focus |
| Screen reader compatibility | unknown | thiết kế cho trình đọc (banner nhận focus); chưa kiểm thực tế |

## Conditional Rendering

| Condition | Type | Renders | Hidden | Notes |
|-----------|------|---------|--------|-------|
| role ≠ `trading` | auth | — | cả route | `proxy.ts` → `/403`; `confirmTrade` tự kiểm tiếp (`actions.ts:58`) |
| `successBanner` | ui-state | banner xanh | — | `negotiated-form.tsx:54-66` |
| `errorMessage` | ui-state | banner đỏ nhận focus | — | `negotiated-form.tsx:68-78` |
| `lotPreview`/`buyerPreview` có giá trị | ui-state | khối tham khảo tương ứng | — | `lot-buyer-preview.tsx:42-67,73-95` |
| `buyerPreview.qualified` | ui-state | badge xanh "Tư cách tham gia còn hiệu lực" (JA: 参加資格は有効です) / badge đỏ "Tư cách đã hết hạn từ {expired_on}." (`t.negotiated.previewBuyerQualified`, `previewBuyerExpiredSince` — `lib/i18n/vi/trades.ts`, `ja/trades.ts`) | — | `lot-buyer-preview.tsx:80-90` |
| `estimatedAmount !== null` | ui-state | dòng "Thành tiền tạm tính" | — | `lot-buyer-preview.tsx:98-103` |

## Component Variants

Không áp dụng — màn không có component đổi hình dạng theo biến thể; khác biệt theo chức danh/trạng thái đã mô tả ở Conditional Rendering.

## Security Surface

| Guard | Type | Consequence if bypassed |
|-------|------|------------------------|
| `ROUTE_POLICY { prefix: "/trades/negotiated", roles: ["trading"] }` — `proxy.ts` | role | role khác → `/403`; entry đứng **trước** `/trades` để không bị prefix chung nuốt (`route-policy.ts:20,22`) |
| **Tầng 2** — Server Action `confirmTrade` tự kiểm `user.role !== "trading"` → `errorCode: "P0005"` **trước khi gọi RPC** | role | `actions.ts:57-64` |
| **Tầng 3** — RPC `confirm_negotiated_trade` (`SECURITY DEFINER`) tự kiểm `current_app_role()` ở dòng đầu | role | không thể bỏ qua vì hàm chạy quyền definer — `0007_rpc_functions.sql:52-56`; `architecture.md` §5 (ba lượt độc lập) |
| `trades` không có policy INSERT/UPDATE/DELETE qua RLS — đường ghi duy nhất là `confirm_negotiated_trade`/`record_auction_result` | permission | (tài liệu gốc trong repo, không nộp) `db-schema.md` §2; `0009_rls_policies.sql:53-55`; ADR-002 |
| `v_lot_availability` `security_invoker = true` | permission | RLS gốc của `trades`/`lots` áp dụng qua view — `0006_derived_views.sql:6-8` |

## Source References

1. Page/View: `app/(app)/trades/negotiated/page.tsx:12-24`
2. Form: `app/(app)/trades/negotiated/negotiated-form.tsx:23-149`
3. State + bàn phím: `app/(app)/trades/negotiated/use-negotiated-form.ts:25-173`
4. Vùng tham khảo: `app/(app)/trades/negotiated/lot-buyer-preview.tsx:26-108`, `use-trade-preview-lookup.ts:11-70`, `preview-actions.ts:20-48`, `lib/domain/trade-preview.ts:26-149`
5. Server Action: `app/(app)/trades/actions.ts:31-88`
6. Ánh xạ lỗi: `lib/domain/rpc-errors.ts:14-78`, `lib/domain/trade-rejection-message.ts:24-60`
7. RPC: `supabase/migrations/0007_rpc_functions.sql:12-38` (`_assert_buyer_qualified`), `:41-96` (`_confirm_trade`), `:98` (`confirm_negotiated_trade`)
8. Phân quyền: `lib/auth/route-policy.ts:20`; RLS `0009_rls_policies.sql:53-55`, `0013_trades_select_receiving.sql:16-21`
9. i18n: `lib/i18n/{vi,ja}/trades.ts`

## Source Walkthrough

1. **File:** `supabase/migrations/0007_rpc_functions.sql:41-96` — bắt đầu từ `_confirm_trade`: nơi hai phép kiểm P0003/P0004 chạy trong một transaction.
2. **File:** `app/(app)/trades/actions.ts:31-88` — Server Action: tầng phòng thủ thứ hai + ánh xạ lỗi.
3. **File:** `app/(app)/trades/negotiated/use-negotiated-form.ts:25-173` — state controlled, bàn phím, `validateFormat`.
4. **File:** `app/(app)/trades/negotiated/negotiated-form.tsx:23-149` — JSX, thứ tự Tab.
5. **File:** `app/(app)/trades/negotiated/lot-buyer-preview.tsx:26-108` — vùng tham khảo (không phải cổng).
6. **File:** `lib/domain/trade-rejection-message.ts:24-60` — dựng thông báo từ `{code, details}`.

### Call Hierarchy

```text
NegotiatedTradePage -> NegotiatedForm -> useNegotiatedForm -> confirmTrade -> findLotByCode + findParticipantByCode -> rpc confirm_negotiated_trade -> _confirm_trade
                                      \-> useTradePreviewLookup -> lookupLotPreview / lookupBuyerPreview -> trade-preview.ts
                     \-> LotBuyerPreview (chỉ hiển thị)
```

### Read Calls *(mục bổ sung)*

| Thao tác | Cách gọi | Tham số | Trả về |
|---|---|---|---|
| Tham khảo thông tin lô (`onBlur` mã lô) | `select` từ `v_lot_availability` + `lots` + `commodities` qua Server Action `lookupLotPreview` (`preview-actions.ts` → `getLotPreview`) | `lotCode` | `LotPreview \| null` — chỉ hiển thị, không chặn |
| Tham khảo tư cách người mua (`onBlur` mã người mua) | `select` từ `participants` + `participant_qualifications` + `qualification_basis_rules` qua `lookupBuyerPreview` (`getBuyerQualification`) | `buyerCode` | `BuyerQualificationPreview \| null` — chỉ hiển thị, không chặn |
| Chốt giao dịch | `rpc('confirm_negotiated_trade', {...})` gọi từ `confirmTrade`, sau khi tra `lotCode`/`buyerCode` → `id` | `p_lot_id, p_buyer_id, p_quantity, p_unit_price` | `jsonb {trade_code, remaining_after}` hoặc raise `P0001`–`P0005` |

Chốt giao dịch **bắt buộc** qua RPC vì `trades` không có policy INSERT/UPDATE/DELETE qua RLS. Hai lệnh tham khảo dùng `select` thẳng vì chỉ đọc, không phải cổng kiểm tra.

### Dependencies & Risks *(mục bổ sung)*

- Cần ít nhất một lô `status='公開'` với `remaining > 0` để demo chốt thành công, và một người mua có tư cách hết hạn (`PTI-01`, hết hạn `2026-09-14` theo seed) để demo `P0003`.
- **Rủi ro triển khai lớn nhất:** nhầm vùng tham khảo là nơi kiểm tra hợp lệ rồi thêm điều kiện `if (remaining < quantity) disable submit` — vi phạm nguyên tắc "server luôn là cổng duy nhất", và còn sai về dữ liệu vì giá trị tham khảo có thể đã cũ (race hai bàn điều hành chốt cùng lúc).
- `P0003`/`P0004` đến từ **cùng một RPC call**, không phải hai lần gọi riêng — nếu LAB-5 tách thành hai bước kiểm riêng sẽ phá vỡ tính transaction + `FOR UPDATE` mà thiết kế gốc yêu cầu.
- Banner lỗi và banner thành công dùng **state riêng** (`errorMessage`, `successBanner`), không bị cuốn theo `revalidatePath`.
- **Quan sát từ code (2026-10-06):** vùng tham khảo tính "còn tư cách" bằng ngày UTC (`new Date().toISOString().slice(0,10)`, `trade-preview.ts:106`) còn RPC dùng `current_date` của Postgres (`0007_rpc_functions.sql:71`) — có thể lệch quanh nửa đêm; chỉ ảnh hưởng phần tham khảo, không ảnh hưởng cổng.
- **Tương lai (F002/ADR-006):** khi có hai trạng thái `条件付き`/`再審査`, RPC này phải trả thêm `P0010`/`P0011` (xem `SCR005`/`SCR006`/`SCR036`; `P0010`/`P0011` là mã đề xuất, chưa cài).
- Ma trận quyền (xem/ghi, đủ 9 role — PREMISE C-09): `trading` ●/●; `sys_admin`, `receiving`, `reconcile` ✗/✗; 5 role đề xuất `party_admin`, `rule_admin`, `ops_admin`, `market_manager`, `internal_audit` ✗/✗. Cờ `can_approve` không liên quan (chốt giao dịch không cần duyệt). Tra cứu tư cách trước khi chốt: `trading` mở được `SCR005` ở chế độ chỉ xem (`docs/system/permissions.md`, nhóm `trading` mở rộng).
- Link chéo: [F004](../../features/F004_TradeRecording/screens.md); flow FL-01; ADR-002, ADR-004, ADR-003 (số còn lại là view).

### Unresolved Questions *(mục bổ sung)*

Giới hạn đã biết của bản đã xây, chưa sửa (đề xuất cho LAB-5):

1. **Thông báo phụ thuộc state vùng tham khảo.** `buildTradeRejectionMessage` lấy `buyer_name`, `unit` từ `preview.buyerPreview`/`preview.lotPreview` (`use-negotiated-form.ts:86,98-100`). Nếu người dùng gõ xong rồi bấm `Ctrl+Enter` mà chưa Tab rời ô (blur chưa chạy) thì `P0003` ra "Người mua  đã hết hạn…", `P0004` ra "còn 0 " (unit rỗng) và banner thành công ra "Lô còn lại 5 ." — đây lại là đường keyboard-first chính. Đề xuất: Server Action trả `buyer_name`, `unit` trong kết quả (SCR011 đã có `unit` trong `TradeActionResult`, `actions.ts:25`).
2. **`{closed_at}` của `P0002` hiển thị giờ UTC**, không phải JST (lệch 9 giờ) — người kiểm thử sẽ kỳ vọng JST. Đề xuất hiển thị JST.
3. **`P0003` khi không có tư cách `active` hợp lệ nhưng có dòng khác.** RPC chọn dòng `order by valid_to desc limit 1` **không lọc** `status`/`valid_from` (`0007_rpc_functions.sql:27-31`): (a) dòng `valid_from` ở tương lai hoặc `status` khác `active` vẫn được nói là "hết hạn từ {valid_to}" (ngày tương lai — sai nghĩa); (b) người mua không có dòng tư cách nào → `v_exp` null, `details` toàn null, thông báo thành "hết hạn từ , căn cứ . Liên hệ ." Đề xuất chuẩn hoá thông báo ("chưa có tư cách" / "chưa hiệu lực từ …").
4. Khi có `条件付き`/`再審査` (F002), vùng tham khảo cần hiển thị thêm hai trạng thái này; hiện chỉ có `qualified` đúng/sai.
