# Permissions

**Project**: Product A — 水産物卸売市場 業務支援システム
**Generated**: 2026-10-06
**Analysis Scope**: toàn hệ thống — 4 chức danh **ĐÃ TRIỂN KHAI** trong prototype LAB-3 (đọc từ `lib/auth/route-policy.ts` và `supabase/migrations/0009`, `0013`); 5 chức danh **ĐỀ XUẤT** lấy từ bộ screen spec LAB-4, chưa có trong database.

> Trang tóm tắt cho PM/BA: ai làm được gì, ranh giới ở đâu. Thiết kế ma trận chi tiết theo từng màn nằm ở spec từng màn (`docs/screens/SCR###_*/spec.md`, mục "Ai mở được"); layout spec-standard của bộ này không có file `permissions-matrix.md` riêng nên phần ma trận được đặt ở **Phụ lục** cuối trang. Quy ước mã chức danh của bộ này: chỉ dùng `sys_admin`, `receiving`, `trading`, `reconcile` (đã có) và `party_admin`, `rule_admin`, `ops_admin`, `market_manager`, `internal_audit` (đề xuất).

## Authorization System Type

**System Type**: `rbac` — chức danh trong bảng `app_users` quyết định route nào mở được và dòng dữ liệu nào đọc được. Bộ thiết kế đề xuất nâng thành `hybrid` (RBAC + cờ thẩm quyền duyệt `can_approve` + quy tắc người lập ≠ người duyệt) *(ĐỀ XUẤT)*.

| System Type | Áp dụng ở đây |
|-------------|---------------|
| `rbac` | **Có** — 4 chức danh nội bộ, ĐÃ TRIỂN KHAI |
| `ownership` | Chỉ một điểm: mọi chức danh tự xem được hồ sơ `app_users` của chính mình |
| `hybrid` | ĐỀ XUẤT — thêm cờ `can_approve` và maker ≠ checker cho F07, tư cách tham gia, quy chế/料率 |
| `abac`, `acl` | Không dùng |

**Identified Roles**:

ĐÃ TRIỂN KHAI (enum `app_role`, `0001_extensions_and_enums.sql:7`):
- `sys_admin` — Quản trị hệ thống (システム管理者)
- `receiving` — Tiếp nhận & Giao hàng (荷受け)
- `trading` — Giao dịch (取引担当)
- `reconcile` — Đối chiếu (照合事務局)

ĐỀ XUẤT (chưa có trong DB):
- `party_admin` — Quản lý người tham gia
- `rule_admin` — Quản trị quy chế (ROLE-RULE-ADMIN)
- `ops_admin` — Vận hành
- `market_manager` — Ban quản lý
- `internal_audit` — Kiểm toán nội bộ (role riêng, không dùng cờ `can_audit`)

Ánh xạ với 7 chức danh của LAB-1 (`TBL-ROLE-01`):

| LAB-1 | Chức danh trong hệ thống | Ghi chú |
|---|---|---|
| ROLE-SYS-ADMIN | `sys_admin` | Có |
| ROLE-INTAKE | `receiving` | Có |
| ROLE-DELIVERY (配送部門) | `receiving` | **Gộp có chủ ý** (D-03; đã chốt PREMISE C-10) để giữ 4 chức danh; nếu làm thật phải tách lại vì phân tách người nhận và người giao hàng là kiểm soát nội bộ |
| ROLE-TRADE | `trading` | Có |
| ROLE-SETTLEMENT | `reconcile` | Có |
| ROLE-JUDGE (目利き) | `receiving` | **Gộp có chủ ý, đã chốt (PREMISE C-10)**: không có role đánh giá riêng; [SCR009](../screens/SCR009_LotDetail/spec.md) dùng `receiving` |
| ROLE-RULE-ADMIN | `rule_admin` *(ĐỀ XUẤT)* | |

Prototype có 4 chức danh, LAB-1 có 7, bộ spec dùng 9. Đã chốt ở [PREMISE](../../PREMISE.md) C-09: dùng cả hai, 9 role key quyết định ai mở màn nào, cờ `can_approve` trên tài khoản quyết định ai được duyệt. Giữ đúng 4 thì maker-checker không hiện thực hoá được.

## Curated View

**ĐÃ TRIỂN KHAI:**

- **`sys_admin`** xem và quản lý tài khoản nội bộ: danh sách tài khoản, đổi chức danh (có lịch sử), đọc nhật ký kiểm toán và lịch sử đổi quyền. **Không** mở được bất kỳ màn nghiệp vụ nào (lô, giao dịch, giao hàng, đối chiếu) — cố ý, để chứng minh phân quyền là hạn chế thật chứ không phải "admin thấy tất cả". Không đổi được quyền của chính mình.
- **`receiving`** nhập lô, xem danh sách lô, chuyển lô sang "mở bán", ghi từng lần giao hàng của một giao dịch. Đọc được giao dịch và giao hàng (để thấy "đã giao / còn lại") nhưng **không** chốt giao dịch và không chốt sổ.
- **`trading`** xem lô, chốt giao dịch thỏa thuận (相対), ghi kết quả đấu giá (せり), xem danh sách giao dịch. **Không** nhập lô, không ghi giao hàng, không chốt sổ.
- **`reconcile`** xem danh sách giao dịch, xem bảng đối chiếu ngày và **chốt sổ**. Không tạo giao dịch hay ghi giao hàng.
- **Mọi chức danh** vào được trang chủ theo chức danh và tự xem hồ sơ của mình.

**ĐỀ XUẤT** (theo screen spec):

- **`party_admin`** lập phiếu chuyển trạng thái tư cách (maker); với cờ `can_approve` thì duyệt phiếu khác người lập (checker). Quản lý hồ sơ người tham gia.
- **`rule_admin`** lập bản nháp quy chế/料率 (maker); với `can_approve` thì duyệt/quay lui (checker). Quản lý chợ và quy tắc vận hành, lịch nghỉ chợ (chế độ nhập tay).
- **`ops_admin`** xử lý hàng cảnh báo vận hành, theo dõi batch kế toán và thông báo; thử lại thủ công, nhận xử lý, đóng cảnh báo.
- **`market_manager`** xem danh mục báo cáo và nội dung báo cáo (trừ log điều chỉnh sau chốt); không thấy hạ tầng batch.
- **`internal_audit`** tra cứu nhật ký kiểm toán, chỉ đọc.
- **`reconcile`** (mở rộng) lập phiếu điều chỉnh sau chốt; với `can_approve` thì duyệt phiếu **không phải do chính mình lập**; tính tiền khuyến khích; tạo và thử lại lô xuất kế toán; xem tranh chấp, báo cáo, nhật ký kiểm toán (cần nới RLS).
- **`trading`** (mở rộng) tra cứu danh sách người tham gia trước khi chốt (giảm số lần gặp lỗi hết hạn tư cách).

## Access Boundaries

1. **`sys_admin` ≠ nghiệp vụ.** Quản trị kỹ thuật không quyết định, không xem số liệu kinh doanh (số tiền, báo cáo). Ở các màn đề xuất, `sys_admin` cùng lắm chỉ **xem để hỗ trợ**, không ghi.
2. **Người ghi ≠ người duyệt (maker ≠ checker)** *(ĐỀ XUẤT)*: phiếu điều chỉnh sau chốt, phiếu chuyển trạng thái tư cách, bản quy chế/料率 đều cần hai người khác nhau. Cưỡng chế **trong thân RPC**, không chỉ ẩn nút trên giao diện (`US-17`: "cưỡng chế ở tầng dữ liệu"). Hệ thống không tự duyệt (CAP-HUMAN-01).
3. **Ba lượt phòng thủ độc lập** *(ĐÃ TRIỂN KHAI)*: (1) `proxy.ts` chặn route theo chức danh — sai chức danh bị chuyển tới trang `/403`; (2) Server Action tự kiểm chức danh; (3) mỗi RPC tự kiểm chức danh ở dòng đầu và RLS giới hạn dòng đọc được. Lý do kỹ thuật: Server Function là một POST tới chính route chứa nó nên lớp proxy có thể âm thầm biến mất khi đổi `matcher` hay refactor route — [architecture.md §5](architecture.md#5-phân-quyền-hai-tầng--lý-do-kỹ-thuật).
4. **Ghi chỉ qua RPC với bảng giao dịch.** `trades`, `deliveries` (và nhật ký) không có policy ghi; người có khoá API công khai cũng không ghi thẳng được ([ADR-002](../decisions/ADR-002-no-write-policy-on-trades-deliveries.md)). Ngoại lệ có chủ ý: `lots` cho phép `receiving` ghi trực tiếp (nhập lô), nhưng vẫn bị chặn khi ngày đã chốt.
5. **Đổi quyền có hiệu lực ngay** — chức danh đọc từ bảng ở mỗi request, không nằm trong token ([ADR-001](../decisions/ADR-001-role-from-table-not-jwt.md)); tài khoản bị tạm dừng bị đăng xuất ở request kế tiếp.
6. **Hồ sơ người tham gia là thông tin cá nhân (APPI 09-06).** Hiện `participants`, `participant_qualifications` đọc được bởi **mọi** chức danh đã đăng nhập; thiết kế đề xuất siết lại — nhưng làm vậy sẽ phá lớp tra mã→tên (`lib/domain/master-data.ts`) mà SC-07 và SC-10 dùng, cần cập nhật cùng lúc.
7. **Hạn chế đã biết của RLS `app_users`:** chỉ cho tự xem mình và `sys_admin`, nên người duyệt không thấy tên người lập phiếu ("Người 起票 = —") — duyệt mà không biết phiếu của ai (rủi ro còn mở; xem [SCR024](../screens/SCR024_RequestDetailApproval/spec.md) mục Dependencies and Risks).

## Special Conditions

- **Tài khoản `suspended`** bị đăng xuất ngay ở tầng request, kể cả khi phiên còn hạn.
- **Ngày nghiệp vụ đã chốt**: mọi thao tác ghi trực tiếp bị chặn bất kể chức danh (`P0009`); chỉ đường điều chỉnh có duyệt (ĐỀ XUẤT, [FL-02](../flows/FL-02-post-close-correction.md)).
- **Hiệu lực tư cách người mua** quyết định ai bị từ chối chốt giao dịch — xét theo ngày hôm nay ([FL-03](../flows/FL-03-participant-qualification-lifecycle.md)).
- **Thứ tự prefix trong `ROUTE_POLICY`**: duyệt tuần tự, khớp entry đầu tiên → prefix cụ thể phải đứng trước prefix chung (vd `/lots/new` trước `/lots`; `/trades/negotiated` trước `/trades`). Prefix mới đề xuất (`/notifications/settings` trước `/notifications`; `/deliveries/new` khớp chính xác trước `/deliveries`) — thiết kế đề xuất, chưa có code.
- **MFA** phân tầng — **đã chốt (PREMISE C-04)**: MFA cho quản trị và chức danh duyệt nội bộ (NFR-SEC-01); bên ngoài (参加者) single-factor + device binding (`SEC-IAM-02`). Prototype **chưa có** MFA (MOCK-1) — đây là thiết kế, không phải hành vi đang chạy.
- **Mất kết nối**: SC-10/SC-11/SC-18 không dùng được offline; SC-07/SC-14 được phép nhập bù (ĐỀ XUẤT). Hàng đợi offline thuộc về **thiết bị**, nên mỗi mục phải mang theo người tạo khi ca sau đăng nhập cùng máy.
- **Môi trường**: không có điều kiện theo IP, giờ hay feature flag trong prototype. Cửa sổ dịch vụ 02:00–10:00 JST là mục tiêu khả dụng (NFR-AVL-01), không phải rào cản truy cập.

## Phụ lục — Ma trận chức danh × màn hình / bảng

### A. Màn hình ĐÃ XÂY (nguồn: `lib/auth/route-policy.ts`; phạm vi LAB-3 theo tài liệu gốc trong repo, không nộp: `lab3-scope-lock`)

● = mở được. `ROUTE_POLICY` hiện có: `/lots/new` receiving · `/lots` receiving+trading · `/trades/negotiated` và `/trades/auction` trading · `/trades` trading+reconcile · `/deliveries` receiving · `/reconciliation` reconcile · `/accounts` sys_admin · `/` mọi chức danh.

| Màn hình | Route | `sys_admin` | `receiving` | `trading` | `reconcile` |
|---|---|:---:|:---:|:---:|:---:|
| [SCR001_RoleHome](../screens/SCR001_RoleHome/spec.md) Trang chủ | `/` | ● | ● | ● | ● |
| [SCR003_AccountList](../screens/SCR003_AccountList/spec.md) Danh sách tài khoản | `/accounts` | ● | | | |
| [SCR004_AccountDetail](../screens/SCR004_AccountDetail/spec.md) Chi tiết tài khoản & quyền | `/accounts/[id]` | ● | | | |
| [SCR007_LotEntry](../screens/SCR007_LotEntry/spec.md) Nhập lô hàng | `/lots/new` | | ● | | |
| [SCR008_LotList](../screens/SCR008_LotList/spec.md) Danh sách lô hàng | `/lots` | | ● | ● | |
| [SCR010_NegotiatedTradeConfirm](../screens/SCR010_NegotiatedTradeConfirm/spec.md) Chốt giao dịch thỏa thuận | `/trades/negotiated` | | | ● | |
| [SCR011_AuctionResultEntry](../screens/SCR011_AuctionResultEntry/spec.md) Ghi kết quả đấu giá | `/trades/auction` | | | ● | |
| [SCR012_TradeList](../screens/SCR012_TradeList/spec.md) Danh sách giao dịch | `/trades` | | | ● | ● |
| [SCR014_DeliveryEntry](../screens/SCR014_DeliveryEntry/spec.md) Ghi lần giao hàng | `/deliveries/new` | | ● | | |
| [SCR018_DailyReconciliation](../screens/SCR018_DailyReconciliation/spec.md) Bảng đối chiếu & chốt sổ | `/reconciliation` | | | | ● |

SCR002 (`/login`) là đường **công khai duy nhất** (`PUBLIC_PATHS` trong `lib/supabase/proxy.ts`). Trang `/403` (đích chuyển hướng khi sai chức danh) khớp prefix `/` nên mọi chức danh đã đăng nhập đều mở được. Route `/deliveries` chỉ cho `receiving` và **chuyển hướng thẳng sang `/deliveries/new`** (`app/(app)/deliveries/page.tsx`) — danh sách giao hàng SC-15 chưa xây.

### B. Quyền đọc/ghi theo bảng (RLS — ĐÃ TRIỂN KHAI)

Nguồn: `0009_rls_policies.sql:19-61`, cập nhật theo `0013_trades_select_receiving.sql`. Cả 11 bảng bật RLS.

| Bảng | SELECT | INSERT / UPDATE / DELETE |
|---|---|---|
| `app_users` | tự xem mình (mọi chức danh) + `sys_admin` xem tất cả | không có policy — ghi qua RPC `change_user_role` |
| `role_change_log`, `audit_log` | `sys_admin` | không có policy — ghi qua RPC `SECURITY DEFINER` |
| `qualification_basis_rules`, `participants`, `participant_qualifications`, `commodities`, `business_days` | mọi chức danh đã đăng nhập | không có policy ghi (`business_days` ghi qua RPC `close_business_day`; dữ liệu nền nạp bằng seed) |
| `lots` | `receiving`, `trading` | INSERT/UPDATE cho `receiving` (nhập lô, mở bán) |
| `trades` | `receiving`, `trading`, `reconcile` *(`receiving` mở thêm ở `0013` — D-05)* | **không có policy** — chỉ RPC `confirm_negotiated_trade` / `record_auction_result` |
| `deliveries` | `receiving`, `trading`, `reconcile` | **không có policy** — chỉ RPC `record_delivery` |

> Tài liệu gốc trong repo, không nộp (`db-schema`, §6) vẫn ghi `trades` SELECT chỉ `trading`+`reconcile` — bản đó chưa cập nhật theo `0013`; trang này theo migration.

### C. RPC và chức danh được gọi (ĐÃ TRIỂN KHAI)

| RPC | Chức danh bắt buộc | Từ chối bằng |
|---|---|---|
| `confirm_negotiated_trade`, `record_auction_result` | `trading` | `P0005` |
| `record_delivery` | `receiving` | `P0005` |
| `close_business_day` | `reconcile` | `P0005` |
| `change_user_role` | `sys_admin` (và không tự đổi của chính mình) | `P0005`, `P0007` |
| `log_blocked_write_attempt` | bất kỳ chức danh đã đăng nhập | `P0005` nếu chưa đăng nhập |

### D. Màn hình ĐỀ XUẤT — ai xem/ghi

Tóm tắt từ mục "Ai mở được" của từng spec (chi tiết ở spec). ● xem · ◐ xem + ghi (nếu có `can_approve` thì duyệt) · — không mở.

| Màn | `sys_admin` | `receiving` | `trading` | `reconcile` | Chức danh đề xuất |
|---|:---:|:---:|:---:|:---:|---|
| [SCR005_ParticipantList](../screens/SCR005_ParticipantList/spec.md) | — | — | ● | ● | `party_admin` ◐ |
| [SCR036_QualificationReview](../screens/SCR036_QualificationReview/spec.md) | — | — | — | ● | `party_admin` ◐ (maker; checker khi `can_approve`) |
| [SCR037_MarketManagement](../screens/SCR037_MarketManagement/spec.md) | ● | — | — | ● | `rule_admin` ◐; `party_admin` ● |
| [SCR009_LotDetail](../screens/SCR009_LotDetail/spec.md) | — | ◐ | ● | ● *(cần nới RLS `lots`)* | — |
| [SCR022_CorrectionRequestCreate](../screens/SCR022_CorrectionRequestCreate/spec.md) | — | — | — | ◐ (maker) | — |
| [SCR023_PendingRequestList](../screens/SCR023_PendingRequestList/spec.md) / [SCR024_RequestDetailApproval](../screens/SCR024_RequestDetailApproval/spec.md) | — | — | — | ● ; duyệt khi `can_approve` và khác người lập | — |
| [SCR025_IncentiveResult](../screens/SCR025_IncentiveResult/spec.md) | — | — | — | ◐ | `rule_admin` ● |
| [SCR026_RateVersionList](../screens/SCR026_RateVersionList/spec.md) / [SCR027_RateVersionApproval](../screens/SCR027_RateVersionApproval/spec.md) | ● | — | — | ● | `rule_admin` ◐ |
| [SCR028_ReportCatalog](../screens/SCR028_ReportCatalog/spec.md) | — | — | — | ● | `market_manager` ● |
| [SCR030_AccountingBatch](../screens/SCR030_AccountingBatch/spec.md) | ● | — | — | ◐ | `ops_admin` ◐ |
| [SCR031_NotificationInbox](../screens/SCR031_NotificationInbox/spec.md) | ◐ (của mình) | ◐ | ◐ | ◐ | mọi chức danh đề xuất ◐ |
| [SCR033_OpsAlertQueue](../screens/SCR033_OpsAlertQueue/spec.md) | ● | — | — | ● | `ops_admin` ◐ |
| [SCR034_AuditLogSearch](../screens/SCR034_AuditLogSearch/spec.md) | ● | — | — | ● *(cần nới RLS `audit_log`)* | `internal_audit` ● |
| [SCR035_SyncQueue](../screens/SCR035_SyncQueue/spec.md) | — | ◐ | — | — | — |
| [SCR019_DisputeList](../screens/SCR019_DisputeList/spec.md) | — | ● | ● *(đề xuất)* | ● | `internal_audit` ● *(đề xuất)* |
| [SCR021_MarketCalendar](../screens/SCR021_MarketCalendar/spec.md) | ● | — | — | ● | `rule_admin` ◐ (nhập tay); `ops_admin` ● |

Các màn SCR013, SCR015–SCR017, SCR020, SCR029, SCR032 không liệt kê ở đây: phân quyền theo cùng nhóm tính năng với màn cùng F### (xem mục Security Surface của từng spec).
