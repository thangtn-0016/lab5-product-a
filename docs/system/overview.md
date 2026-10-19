# System Overview

**Project**: Product A — 水産物卸売市場 業務支援システム (hệ thống hỗ trợ nghiệp vụ chợ đầu mối thủy sản)
**Generated**: 2026-10-06
**Architecture Type**: Monolith Next.js 16 (App Router) trên Vercel + Supabase Postgres; logic nghiệp vụ bất biến nằm ở tầng database (RPC `SECURITY DEFINER`, RLS, trigger, view dẫn xuất)

## Executive Summary

Product A hỗ trợ một chợ đầu mối thủy sản sát cảng (発注者: 株式会社 桜水産市場, thành phố hư cấu 桜浜) vận hành chuỗi **tiếp nhận lô → giao dịch → giao hàng → đối chiếu và chốt sổ**. Giao dịch gồm 相対取引 (thỏa thuận, ≈ 90% giá trị — luồng chính) và せり (đấu giá, ≈ 10% — ghi nhận tay). Nguồn gốc: RFP `PRODUCT-A-RFP-JA-V1.0-RC7` (tài liệu演習, số liệu hư cấu), phân tích ở LAB-1, prototype chạy thật ở LAB-3, bộ thiết kế mở rộng ở LAB-4.

Cái khó của hệ thống **không nằm ở scale** (60 người dùng đồng thời, tối đa 1.200 giao dịch/ngày) mà ở bốn thứ: **tính đúng đắn** (không bán vượt lô cá không tái tạo được), **bất biến lịch sử** (ngày đã chốt không sửa trực tiếp), **dấu vết kiểm toán**, và **quy tắc có phiên bản** (quy chế/料率 đổi tối đa 4 lần/năm mà không viết lại quá khứ). Môi trường người dùng cũng quyết định thiết kế: nhân viên hiện trường đeo găng tay, làm từ 2h sáng, dùng chung terminal theo ca, mạng cầu cảng chập chờn → ba màn **keyboard-first** và biên offline do nghiệp vụ vẽ.

Hai ràng buộc đặc thù mà đề LAB-4 nêu đích danh, kiểm trên database đang chạy đều **chưa được đáp ứng** trong prototype: phân biệt `許可` (cần xét duyệt) với `承認` (chấp thuận nhẹ hơn) — D-01 — và quy tắc vận hành riêng từng chợ — D-02. Bộ thiết kế này đặc tả cách vá hai lỗ hổng đó (hai màn mới, [ADR-006](../decisions/ADR-006-split-license-approval-state-machines.md), [ADR-007](../decisions/ADR-007-market-rules-as-versioned-data.md)) và đặc tả đủ 35 màn của LAB-1 cộng 2 màn mới.

For architecture diagrams and tech stack details, see [architecture.md](architecture.md).

### Phạm vi và trạng thái

| Hạng mục | Trạng thái | Ghi chú |
|---|---|---|
| 11 màn LAB-3 (SC-01, 02, 03, 04, 07, 08, 10, 11, 12, 14, 18) | **ĐÃ TRIỂN KHAI** | Đọc/ghi Supabase thật; deploy Vercel |
| 24 màn còn lại của LAB-1 | **ĐỀ XUẤT** (đặc tả thiết kế, chưa có code) | SC-05, 06, 09, 13, 15–17, 19–35 |
| 2 màn mới SC-N1 (xét duyệt tư cách), SC-N2 (quản lý chợ) | **ĐỀ XUẤT** | Vá D-01, D-02; mã `SCR036`, `SCR037` |
| 4 chức danh `sys_admin` `receiving` `trading` `reconcile` | **ĐÃ TRIỂN KHAI** | [permissions.md](permissions.md) |
| 5 chức danh `party_admin` `rule_admin` `ops_admin` `market_manager` `internal_audit` | **ĐỀ XUẤT** | Chưa có trong DB |
| 5 migration nền `0001–0013`, `0020–0021`, `0030–0035`, `0040–0041` | **ĐÃ TRIỂN KHAI** | Nguồn sự thật: `supabase/migrations/` |

Danh sách màn và mã: [SCR001 … SCR037](../screens/) · tính năng [F001 … F012](../features/). Số liệu nền (11 bảng · 83 cột · 17 khoá ngoại · 11 enum · 3 view): [entities.md](../generated/entities.md).

### Mục tiêu kinh doanh đo được (RFP 03-02)

| ID | Mục tiêu | Baseline | Mục tiêu sau 6 tháng vận hành |
|---|---|---|---|
| OBJ-01 | Giảm nhập lại cùng giao dịch trên nhiều kênh | 2,4 lần/giao dịch | ≤ 1,2 |
| OBJ-02 | Thời gian tra cứu bản ghi giao dịch/đối chiếu | 15–30 phút/yêu cầu | ≤ 2 phút |
| OBJ-03 | Giao dịch có 監査証跡 đầy đủ | 35% | ≥ 98% |
| OBJ-04 | Thời gian lập báo cáo ngày | 90 phút | ≤ 15 phút |
| OBJ-05 | Sai lệch đối chiếu do nhập tay | 1,8% dòng | ≤ 0,3% |

Ba điều tuyệt đối không được (逸脱禁止事項, 03-07): không buộc tổ chức đổi mô hình giao dịch cho khớp software; không loại con người khỏi khâu đánh giá và xác nhận kết quả; không đặt mục tiêu hiệu năng thấp hơn data profile đã công bố. Coi Product A là "auction-only system" là **điều kiện bị loại trực tiếp** (12-06, 13-01).

## Key Design Decisions

Bảy ADR, mỗi cái đủ bốn phần (bối cảnh · lựa chọn · phương án đã bỏ · hệ quả). Năm cái đã triển khai (viết sau khi prototype chạy), hai cái là đề xuất.

### Decision 1: Đọc chức danh từ bảng, không nhét vào JWT — [ADR-001](../decisions/ADR-001-role-from-table-not-jwt.md) *(Đã triển khai)*

**Context**: `SC-04` đổi quyền và `F01-04` yêu cầu đổi quyền có lưu vết; JWT sống ~1 giờ nên token cũ vẫn mang chức danh cũ.

**Decision**: `current_app_role()` đọc `app_users` ở mỗi request, dùng chung cho `proxy.ts` và mọi RLS policy.

**Rationale**: một nguồn sự thật, đổi quyền có hiệu lực tức thì. Đổi lại thêm một round-trip mỗi request — không đáng kể ở 60 người dùng đồng thời.

### Decision 2: `trades` và `deliveries` không có policy ghi — [ADR-002](../decisions/ADR-002-no-write-policy-on-trades-deliveries.md) *(Đã triển khai)*

**Context**: hai phép kiểm khi chốt (tư cách + số lượng) phải nằm trong một transaction phía server; PostgREST phơi ra ngoài với publishable key công khai.

**Decision**: chỉ có policy SELECT; đường ghi duy nhất là RPC `SECURITY DEFINER`.

**Rationale**: biến quy ước "không kiểm ở client" thành ràng buộc vật lý. Giá: logic dồn vào PL/pgSQL, mỗi đường ghi mới là một RPC mới.

### Decision 3: Số còn lại là view dẫn xuất — [ADR-003](../decisions/ADR-003-derived-remaining-quantity-views.md) *(Đã triển khai)*

**Context**: SC-08/10/14 cần số còn lại; cách thường là cột `remaining` cập nhật bằng trigger.

**Decision**: ba view `v_lot_availability`, `v_trade_delivery_status`, `v_daily_reconciliation`; `lots.quantity` bất biến.

**Rationale**: không thể lệch (một nguồn sự thật). Nợ đã biết: mỗi lần đọc là một lần tính; `security_invoker` kéo theo mở quyền đọc `trades` cho `receiving` (D-05).

### Decision 4: Bất biến dữ liệu giữ ở tầng database — [ADR-004](../decisions/ADR-004-invariants-at-database-layer.md) *(Đã triển khai, mất 3 vòng migration)*

**Context**: dòng `auction` phải luôn có `auction_session_at` + `auction_confirmed_by_name`, nhưng `_confirm_trade` insert trước rồi `UPDATE` sau trong cùng transaction.

**Decision**: `CONSTRAINT TRIGGER … DEFERRABLE INITIALLY DEFERRED` đọc lại dòng theo `id` thay vì tin `NEW` (`P0008`).

**Rationale**: `CHECK` thường bắn ngay tại INSERT; kiểm ở Server Action biến bất biến thành quy ước code; trigger tin `NEW` vẫn thấy `NULL`. `DEFERRABLE` chỉ hoãn *khi nào* kiểm, không đổi *kiểm cái gì*.

### Decision 5: Chốt sổ cảnh báo chứ không chặn — [ADR-005](../decisions/ADR-005-close-day-warn-not-block.md) *(Đã triển khai; chốt ở PREMISE C-07, khách chưa xác nhận)*

**Context**: `US-11` ghi "điều kiện phải đạt" (đọc sát là chặn) nhưng `F05` cho phép một giao dịch giao nhiều lần qua ngày.

**Decision**: `close_business_day` trả `pending_conditions` nhưng vẫn cho chốt.

**Rationale**: chặn cứng thì không ngày nào chốt được. Sai lệch có ý thức so với nghĩa đen `US-11`; sau chốt vẫn có đường điều chỉnh (F07).

### Decision 6: Tách hai luồng trạng thái cho `許可` và `承認` — [ADR-006](../decisions/ADR-006-split-license-approval-state-machines.md) *(ĐỀ XUẤT)*

**Context**: LAB-1 xếp gộp hai căn cứ này là điều kiện bị loại; prototype dùng một enum ba giá trị và một phép kiểm đồng nhất.

**Decision**: bảng và enum riêng `qualification_license` / `qualification_approval`; `_assert_buyer_qualified` rẽ nhánh theo `basis_type`; phiếu chuyển trạng thái maker-checker. (Bản đầu "mở rộng enum" đã được sửa.)

**Rationale**: `条件付き` vẫn cho giao dịch, `再審査` chặn hẳn; hai căn cứ có trường riêng không chia sẻ được. Giá: SC-06 bốn biến thể, thêm màn xét duyệt, di trú một bảng → ba.

### Decision 7: Quy tắc vận hành là dữ liệu có phiên bản theo chợ — [ADR-007](../decisions/ADR-007-market-rules-as-versioned-data.md) *(ĐỀ XUẤT)*

**Context**: đề nêu đích danh "quy tắc riêng từng chợ"; prototype không có thực thể chợ.

**Decision**: `markets` + `market_rules` có 発効日 và phiên bản; `resolve_market_rule(…, p_as_of)` tra theo ngày nghiệp vụ.

**Rationale**: thêm chợ là thêm dữ liệu; số liệu quá khứ không bị viết lại (`FR-INC-03`). Giá: mọi truy vấn mang ngữ cảnh chợ; có thể là trừu tượng hoá sớm nếu chỉ một chợ.

## Security Overview

- **Authentication**: Supabase Auth email/password; session qua cookie `@supabase/ssr`, kiểm bằng `supabase.auth.getClaims()` trong `proxy.ts`. **MFA (NFR-SEC-01) chưa có trong prototype** (MOCK-1); thiết kế đã chốt phân tầng (PREMISE C-04): MFA cho admin và chức danh duyệt, bên ngoài single-factor + device binding (`SEC-IAM-02`). Tài khoản `suspended` bị `signOut()` ngay tại tầng request.
- **Authorization**: RBAC theo chức danh trong bảng `app_users`; phòng thủ **ba lượt độc lập** — `proxy.ts` (`ROUTE_POLICY`) → Server Action tự kiểm role (`P0005`) → RPC `current_app_role()` + RLS trên 11/11 bảng ([architecture.md §5](architecture.md#5-phân-quyền-hai-tầng--lý-do-kỹ-thuật)). Maker ≠ checker (đề xuất) cưỡng chế trong thân RPC.
- **Data Encryption**: TLS khi truyền (NFR-SEC-04) do Vercel/Supabase cung cấp; repo không có cấu hình mã hoá riêng — **chưa xác minh** mã hoá at-rest hay chính sách backup/RPO.
- **API Security**: publishable key công khai theo thiết kế nên mọi RPC thu EXECUTE khỏi `anon`/`PUBLIC` (bài học `0011`, `0040`, `0041`); client rẽ nhánh theo `error.code` (SQLSTATE `P00xx`), không theo `error.message`; `service_role` chỉ phía server, `.env.local` không commit. Thông tin 許可/承認 là thông tin cá nhân thuộc APPI (09-06) → RLS + access log.

## Scalability

- **Current Capacity**: prototype chạy trên Supabase + Vercel (`https://lab3-product-a.vercel.app`; ADR-001 ghi gói Free); seed ~30 người tham gia, ~20 mặt hàng, 2 ngày nghiệp vụ. Chưa có số đo tải thật. Tải thiết kế của RFP (`FIG-LOAD-01`): 180 account, 60 đồng thời, 350/600 lô và 700/1.200 giao dịch mỗi ngày (thường/cao điểm).
- **Scaling Strategy**: không có ý định scale-out — cố ý KISS (microservice, event streaming ngược budget). Hai điểm sẽ xem lại trước tiên nếu tải tăng một bậc: round-trip `current_app_role()` mỗi request ([ADR-001](../decisions/ADR-001-role-from-table-not-jwt.md)) và chi phí tính view trên bảng giao dịch lớn dần ([ADR-003](../decisions/ADR-003-derived-remaining-quantity-views.md)). Dữ liệu tham chiếu online 7 năm (DR-RET-01), ảnh/chứng từ 3 năm rồi chuyển cold.
- **Performance Targets** (RFP 09, **chưa đo** trong prototype): tra cứu thông thường p95 ≤ 2 giây (NFR-PERF-01); không làm chậm giao dịch core ở load FIG-LOAD-01 (NFR-PERF-02); khả dụng 99,5%/tháng trong 02:00–10:00 JST (NFR-AVL-01); RTO ≤ 4 giờ, RPO ≤ 15 phút (NFR-AVL-03); batch báo cáo ngày xong trước 11:00 JST.

## Điều hướng bộ tài liệu

| Cần gì | Đọc |
|---|---|
| Thành phần, luồng, ràng buộc đặc thù | [architecture.md](architecture.md) |
| Quy tắc nghiệp vụ, mã lỗi `P00xx`, điểm chưa chốt trước LAB-5 | [business-rules.md](business-rules.md) |
| Ai làm được gì | [permissions.md](permissions.md) |
| Thuật ngữ VI · JA | [glossary.md](glossary.md) |
| Lý do từng quyết định | [`decisions/`](../decisions/) (ADR-001 … ADR-007) |
| Chuỗi nghiệp vụ & vòng đời | [FL-01](../flows/FL-01-lot-trade-delivery-reconciliation.md) · [FL-02](../flows/FL-02-post-close-correction.md) · [FL-03](../flows/FL-03-participant-qualification-lifecycle.md) |
| Mô hình dữ liệu, ERD, sai lệch D-01…D-06 | [entities.md](../generated/entities.md) |
