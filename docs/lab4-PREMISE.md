# PREMISE — Tiền đề của bài LAB-4

> Product A — 水産物卸売市場 業務支援システム · Người thực hiện: Trần Ngọc Thắng · Cập nhật: 2026-10-09
> Đọc trước khi mở bộ spec: file này ghi **những gì đã quyết thêm / đổi / loại bỏ kể từ LAB-1**, để người đọc spec không phải đoán vì sao spec lệch bản phân tích.

## Tóm tắt

LAB-1 để lại bản phân tích đầy đủ (35 màn, 63 tính năng, 39 requirement) nhưng để ngỏ nhiều hướng chờ khách trả lời. LAB-3 chỉ dựng 11/35 màn, phần còn lại thành MOCK-1…9. Bài LAB-4 lấy **phạm vi đầy đủ của LAB-1 làm gốc** (giảng viên làm rõ: LAB-3 cắt thì LAB-4 vẫn đặc tả theo LAB-1).

- Đặc tả đủ 35 màn, **thêm** hai màn ngoài LAB-1 (SCR036, SCR037) và tính năng F012 để vá hai lỗ hổng đề bài nêu đích danh: luồng trạng thái riêng cho 許可 vs 承認, và quy tắc vận hành riêng từng chợ. Không tính năng nào của LAB-1 bị bỏ.
- Chỗ LAB-1 có nhiều hướng: **chốt một hướng**, ghi hướng bị loại (mục B). Chỗ khách chưa trả lời: ghi là **giả định** (mục C), không viết như sự thật.
- Theo **spec-standard của Takumi** (giả định G1): 37 màn, 12 nhóm tính năng, 7 ADR, 3 flow.
- Trạng thái: **11 màn đặc tả ngược từ code đã chạy**, **26 màn đặc tả thiết kế, chưa có code để kiểm** (mục D).

## Mã tham chiếu

| Mã | Nghĩa |
|---|---|
| `Q-01`, `Q-03`, `Q-04`, `Q-06`, `Q-08`, "câu 16" | Số câu hỏi trong RFP Q&A của LAB-1 (repo gốc `docs/lab1-baseline/`, không nộp), chờ khách trả lời |
| `A1`, `A3`, `A4` | Giả định báo giá của LAB-1 (A1 offline hẹp, A3 một cấp duyệt, A4 kế toán CSV) |
| `D1`, `D5` | Quyết định phạm vi của LAB-3 (D1 ngân sách giờ, D5 UI song ngữ) |
| `S1`, `S3`, `MOCK-1`…`9` | Điểm LAB-3 cố ý làm khác LAB-1 (S1 gộp role giao hàng, S3 chỉ một lớp xác thực) và hạng mục LAB-3 chỉ mock |
| `D-01`…`D-03` | Phát hiện trong bộ thiết kế database của LAB-4: D-01 許可/承認 bị làm phẳng, D-02 không có khái niệm chợ, D-03 gộp 配送部門 |
| `B-01`…`B-05` | Lỗi đã biết của code LAB-3, lộ ra khi viết spec (xem G3, `docs/system/business-rules.md`) |
| `P-xx` · `C-xx` · `G-xx` | Dòng trong bảng A (phạm vi) · B (đã chốt) · C (giả định) của file này |
| `P0001`…`P0071` | Mã lỗi nghiệp vụ; `P0010` trở lên là đề xuất (G5) |
| maker-checker | Người lập và người duyệt phải khác nhau (`GOV-RULE-01`) |

## Cách đọc bảng

- **So với LAB-1:** `THÊM` (mới, không có ở LAB-1) · `GIỮ` (LAB-1 đã có, LAB-4 không đổi) · `CHỐT` (LAB-1 nhiều hướng / để ngỏ → LAB-4 chọn một) · `BỎ` (LAB-1 có, LAB-4 không làm).
- **Đường dẫn** tính từ thư mục này (`submission/lab-4/specs/`). Mã SCR/F/role theo sổ đăng ký tên đã cấp phát; alias SC-xx là mã LAB-1.
- `(đề xuất)` = do bài này đưa ra, chưa được khách xác nhận.

## A. Phạm vi — thêm / bỏ so với LAB-1

| # | Quyết định | So với LAB-1 | Lý do | Ảnh hưởng tới spec |
|---|---|---|---|---|
| P-01 | Đặc tả **đủ 35 màn của LAB-1**, không theo bản 11 màn LAB-3 | **GIỮ** phạm vi LAB-1 (LAB-3 đã cắt 24 màn → LAB-4 khôi phục). Hướng bị loại: chỉ đặc tả phần đã xây | Giảng viên làm rõ giữa chừng; bản đầu chỉ làm 15 màn rồi phải mở rộng lên 37 | `docs/screens/SCR001…SCR035/spec.md` · `docs/generated/screen-list.md` |
| P-02 | **Thêm F012 Quản lý chợ & quy tắc** + màn **SCR037** `/markets` (đề xuất) | **THÊM** — LAB-1 không có màn nào cho việc này (gần nhất là SC-21 lịch nghỉ chợ) | Đề LAB-4 nêu đích danh "quy tắc vận hành riêng của từng chợ"; truy vấn `information_schema` tìm `market` ra 0 dòng (D-02). Đây là lỗ hổng từ khâu phân tích | `docs/features/F012_MarketManagement/` · `docs/screens/SCR037_MarketManagement/spec.md` · `docs/decisions/ADR-007-*.md` |
| P-03 | **Thêm SCR036** Xét duyệt / tạm dừng tư cách `/qualifications` (đề xuất) | **THÊM** — không có trong 35 màn LAB-1 | Cần chỗ thực thi hai máy trạng thái 許可/承認 với maker-checker (D-01) | `docs/screens/SCR036_QualificationReview/spec.md` · `docs/features/F002_ParticipantsAndQualification/` · `docs/flows/FL-03-*.md` |
| P-04 | **Không bỏ** tính năng nào của LAB-1; 9 hạng mục LAB-3 mock (F07–F11, tranh chấp, lịch chợ, SC-06, offline…) đều có spec | **GIỮ** | LAB-3 mock vì giới hạn giờ (D1 20–40h), không phải vì LAB-1 bỏ. Spec phải là đầu vào của LAB-5 | `docs/features/F007…F011/` |
| P-05 | Giữ nguyên **sáu `SCOPE-OUT`** và bốn điểm `CAP-HUMAN-01` (không OCR/AI cho 手やり, không tự đánh giá chất lượng, không chuyển tiền ngân hàng…) | **GIỮ** | Bỏ là disqualifier (RFP 13-01). SCR011 vẫn là nhập tay, không ô tải ảnh, không gợi ý | `docs/system/business-rules.md` · `docs/screens/SCR011_AuctionResultEntry/spec.md` |
| P-06 | **相対取引 (90%) là luồng chính, せり (10%) là phụ** | **GIỮ** | Coi là auction-only = bị loại trực tiếp (RFP 12-06). Đề LAB-3 gọi Product A là "sàn đấu giá" — hiểu vậy là sai | `docs/screens/SCR010_NegotiatedTradeConfirm/spec.md` · `docs/features/F004_TradeRecording/` |

## B. Chỗ LAB-1 có nhiều hướng — đã chốt một

| # | Vấn đề | Hướng CHỐT | Hướng bị LOẠI | Lý do | Ảnh hưởng tới spec |
|---|---|---|---|---|---|
| C-01 | **Estimate** A (995 MD, top-down 39 requirement) vs B (1.210,5 MD, bottom-up 35 màn) — LAB-1 trình bày cả hai | **CHỐT** phạm vi theo **nền B**: 35 màn hình làm đơn vị, vì B dựng từ 35 màn nên khớp cách LAB-4 đặc tả theo từng màn; feature-list LAB-3 cũng đã đối chiếu MD theo B. Bài LAB-4 không định giá lại | A (không đi qua function → feature → màn hình, không đủ để đặc tả từng màn) | LAB-4 đặc tả theo màn; chưa có rate thật nên không kết luận bid/no-bid ở đây | `docs/generated/screen-list.md` |
| C-02 | **Phạm vi offline** (Q-01): hẹp vs rộng | **CHỐT hẹp (giả định A1):** chỉ nháp lô (`F03-01`) và ghi lần giao (`F05-01`) có hàng đợi cục bộ; 確定 / chốt sổ / duyệt bắt buộc online | Offline toàn phần (+80…120 MD theo LAB-1) | `FR-AITAI-02` cần trạng thái server khi chốt → 確定 không thể offline. Lý luận nghiệp vụ, không phải lời hứa | `docs/screens/SCR035_SyncQueue/spec.md` · `docs/screens/SCR007_LotEntry/spec.md` · `SCR014_DeliveryEntry` |
| C-03 | **Tích hợp kế toán** `IF-ACC-01` (Q-04): CSV một chiều vs API hai chiều | **CHỐT CSV một chiều qua SFTP (giả định A4)** — màn biết lô *đã gửi*, không biết *đã được chấp nhận* | API hai chiều (+25…35 MD) — chỉ ghi ở mục "nếu đổi" của spec, không thiết kế song song | Phương thức kết nối quyết ở インターフェース検討会 chưa diễn ra; chọn baseline rẻ, option tách riêng (RFP 04-05) | `docs/screens/SCR030_AccountingBatch/spec.md` · `docs/features/F009_ReportsAndExport/` |
| C-03b | **Cấp duyệt điều chỉnh sau chốt** (Q-03): một cấp vs ba cấp theo ngưỡng tiền | **CHỐT một cấp (giả định A3)**, maker ≠ checker | Ba cấp (+10…14 MD, thêm màn định nghĩa ngưỡng) | Khách chưa gửi 業務状態マトリクス; một cấp là baseline | `docs/features/F007_PostCloseCorrection/` · `docs/screens/SCR024_RequestDetailApproval/spec.md` · `docs/flows/FL-02-*.md` |
| C-04 | **MFA** (Q-06): MFA cho mọi người vs phân tầng | **GIỮ phân tầng của LAB-1:** MFA cho admin + role duyệt nội bộ; bên ngoài single-factor + device binding. Prototype vẫn một lớp (S3/MOCK-1) | MFA cho 参加者 ngoài (hardware token chưa có ngân sách; găng tay + 02:00) | Xung đột `NFR-USE-01`. Spec ghi là *thiết kế*, không phải hành vi đang chạy | `docs/system/permissions.md` · `docs/screens/SCR002_Login/spec.md` |
| C-05 | **SC-06**: 1 màn 4 biến thể hay 4 màn (LAB-1 để ngỏ, tách thì thành 38 màn) | **CHỐT 1 màn SCR006, 4 biến thể form** theo căn cứ tham gia | Tách 4 màn (38 màn, không khớp tổng 37) | Bốn căn cứ có trường khác bản chất nhưng cùng một thực thể "hồ sơ người tham gia" | `docs/screens/SCR006_ParticipantProfile/spec.md` · `docs/design/wireframes/sc-06-*.html` |
| C-06 | **Mô hình dữ liệu 許可 vs 承認** (D-01; LAB-1: "hai mô hình dữ liệu riêng") | **CHỐT bảng riêng + enum riêng cho mỗi căn cứ có luồng vi phạm** (`qualification_license`, `qualification_approval`), `_assert_buyer_qualified` rẽ nhánh theo `basis_type`. `市場登録` và đăng ký tại điểm GD giữ `qualification_status` cũ | (1) mở rộng enum `qualification_status` — chính ADR-06 bản đầu, đã sửa vì sai; (2) bốn bảng riêng — vi phạm DRY; (3) giữ cột `violation_action` làm dữ liệu chết | Bảng riêng vì hai căn cứ có trường không chia sẻ được (許可 cần cơ quan cấp ngoài chợ; 承認 cần hạn mức + người duyệt là chính chợ) | `docs/decisions/ADR-006-*.md` · `docs/features/F002_ParticipantsAndQualification/technical-spec.md` · `docs/generated/entities.md` |
| C-07 | **Chốt sổ khi còn giao dịch chưa giao xong** (`US-11` "điều kiện phải đạt": chặn hay cảnh báo?) | **CHỐT cảnh báo** — `close_business_day` trả `pending_conditions`, vẫn cho chốt (ADR-05) | Chặn cứng (không ngày nào chốt được vì giao hàng kéo dài qua ngày); chặn kèm ghi đè (ngoài ngân sách) — ghi như hướng mở | `F05` ghi một giao dịch giao nhiều lần do xe gom. Là **cách đọc khác nghĩa đen của `US-11`**, đã ghi là sai lệch có ý thức | `docs/decisions/ADR-005-*.md` · `docs/screens/SCR018_DailyReconciliation/spec.md` |
| C-08 | **Lịch nghỉ chợ** (câu 16): nhập tay vs đồng bộ nguồn ngoài | **CHỐT mặc định `manual`** (giả định báo giá) | `external_sync` làm hướng chính. Spec chỉ giữ một cờ `calendar_mode` để không phải vẽ lại nếu khách chọn khác — **không** thiết kế song song hai màn | Hai hướng là hai khối công việc khác nhau; hướng đồng bộ phát sinh hạng mục chưa ước lượng | `docs/screens/SCR021_MarketCalendar/spec.md` |
| C-09 | **Số chức danh**: LAB-1 7 · LAB-3 4 · cờ `can_approve` hay thêm role? | **CHỐT 9 role key + cờ `can_approve`:** 4 đã có (`sys_admin` `receiving` `trading` `reconcile`) + 5 đề xuất (`party_admin` `rule_admin` `ops_admin` `market_manager` `internal_audit`). Role quyết định ai mở màn nào; cờ `can_approve` trên tài khoản quyết định ai được duyệt (maker ≠ checker ở SCR024, SCR027, SCR036, SCR037) | Chỉ dùng role, gắn quyền duyệt theo role (phải thêm role "người duyệt" cho từng nghiệp vụ); giữ 4 role (maker-checker không hiện thực hoá được); quay về 7 chức danh LAB-1 (phải đổi tên role trên cả 37 spec) | Tên khác (`ops`, `admin`…) bị cấm để không lệch giữa các file. 9 role **không ánh xạ 1-1** với 7 chức danh LAB-1 | `docs/system/permissions.md` · `docs/generated/` |
| C-10 | **配送部門** và **目利き** (`ROLE-JUDGE`, LAB-1 có chức danh riêng) vs gộp vào `receiving` (LAB-3 S1/D-03 mới gộp 配送部門) | **GỘP cả hai vào `receiving`** — 配送部門 theo S1 của LAB-3, 目利き gộp cùng cách (khớp `docs/system/permissions.md`). Ghi là **nợ**: làm thật phải tách lại vì phân tách nhiệm vụ nhận/giao/đánh giá hàng là kiểm soát nội bộ | Tách thêm role cho từng chức danh ngay trong LAB-4 | Không đủ màn hiện trường để chứng minh thêm | `docs/system/permissions.md` · `docs/screens/SCR014_DeliveryEntry/spec.md` |

## C. Giả định mới (chưa khách nào xác nhận)

| # | Giả định | Nếu sai thì | Ảnh hưởng tới spec |
|---|---|---|---|
| G1 | **Format: spec-standard của Takumi** (`rebuild-spec`: `system/ decisions/ flows/ generated/ features/ screens/`; 4 file mỗi feature). Lý do: **không có bộ spec-standard nào khác được cung cấp**; người thực hiện chọn Takumi. Heading giữ tiếng Anh của template, văn xuôi tiếng Việt, thuật ngữ JP giữ nguyên | Nếu công ty có chuẩn riêng → phải chuyển lại 37 spec | Toàn bộ `docs/` |
| G2 | **Một chợ hay nhiều chợ** chưa rõ → mô hình dữ liệu hỗ trợ N chợ (`markets`, `market_rules`), mức ưu tiên SCR037 coi là *nên có* nếu thực tế một chợ | Thực tế một chợ = trừu tượng hoá sớm (rủi ro đã ghi ở ADR-07) | `docs/decisions/ADR-007-*.md` · SCR037 |
| G3 | **Hành vi đúng của B-01/B-02** là hành vi được đặc tả: `v_trade_delivery_status` phải *trừ* `returned`/`partial_cancel`; hoàn tất giao hàng ghi ở cấp giao dịch (`trades.delivery_completed_at`, chốt bằng thao tác ở SCR016), lô chỉ chuyển `引渡し完了` khi mọi giao dịch của lô đã hoàn tất. Code LAB-3 hiện sai ở hai chỗ này (xác minh trên DB đang chạy); B-04, B-05 tìm thêm ở lượt nộp lại, xem `docs/system/business-rules.md`. B-05 (view `v_daily_reconciliation` cộng nhầm) chưa sửa; đã có bản vá nháp `supabase/migrations/0042_fix_daily_reconciliation_amount.sql` chưa áp, chưa commit | Spec 4 màn (SCR014/015/016/018) mô tả hành vi chưa có trong code | `docs/features/F005_DeliveryTracking/edge-cases.md` |
| G4 | **`条件付き` hạn chế cái gì** — tài liệu khách chỉ có cụm "hạn chế có điều kiện". Nội dung cụ thể trong spec là **suy luận rủi ro cao**; mã `P0010` không có gì để kiểm nếu khách nói khác | `P0010` và nhánh 許可 của SCR036 phải viết lại | `docs/features/F002_ParticipantsAndQualification/` |
| G5 | **Mã lỗi `P0010` trở lên là đề xuất**, chưa cài; `P0001–P0009` đã dùng hết trong prototype. Mỗi nhóm có dải riêng để không đụng nhau (68 mã distinct sau lượt nộp lại, không mã nào hai nghĩa; ba mã `P0069`–`P0071` cấp lại 2026-10-06) | Đổi dải = sửa spec nhiều file | `docs/features/*/technical-spec.md` |
| G6 | **Fixture CSV / test vector Appendix C không có trong tay** (LAB-1 đã ghi là câu hỏi mở) → spec F008 không cố định số; công thức giữ theo `BR-INC-01`, chờ test vector để verify `FR-INC-01` | Công thức tiền khuyến khích chưa kiểm được bằng số | `docs/screens/SCR025_IncentiveResult/spec.md` · `docs/features/F008_IncentiveAndRateVersion/` |
| G7 | **Ngôn ngữ:** spec văn xuôi tiếng Việt, tên tiếng Nhật giữ nguyên từ LAB-1, UI song ngữ VN+JA theo D5 của LAB-3. Phân tầng JA/EN của deliverable (Q-08) chưa khách xác nhận | +5…10% effort nếu khách đòi toàn bộ tài liệu tiếng Nhật | Toàn bộ `docs/` |
| G8 | **Ai duyệt `再審査`** (SCR036; khách chưa trả lời): chọn **`party_admin` có cờ `can_approve`**, khác người lập (`P0013`). Căn cứ: `GOV-RULE-01` đòi *tách người*, không đòi tách chức danh; cờ có sẵn từ C-09 nên không thêm role. **Hướng loại:** chức danh `qualification_approver` riêng (thành chức danh thứ 10, ma trận role phình thêm); ban lãnh đạo duyệt ngoài hệ thống (màn chỉ ghi kết quả, mất dấu vết hai người) | Nếu khách chọn hướng khác → đổi khối duyệt SCR036, `P0013`, quyền SCR037 | `docs/screens/SCR036_QualificationReview/spec.md` · `docs/system/permissions.md` |
| G9 | **Hạn hiệu lực theo căn cứ** (người làm bài đã chốt; khách chưa xác nhận): `valid_to` **không bắt buộc** (NULL = không thời hạn) với hai căn cứ mà LAB-1 không ghi ngày hết hạn — đăng ký chợ (F02-01, 卸売業者, `basis_type = registration`, `市場登録`) và đăng ký tại điểm giao dịch (F02-04, 買出人, `basis_type = market_registration`, `卸売場登録`). `許可` (F02-02) và `承認` (F02-03) **bắt buộc** có ngày hiệu lực + hết hạn. Căn cứ: feature list LAB-1 (repo gốc `docs/lab1-baseline/product-a-feature-list.md`, không nộp) dòng F02-01…F02-04 (F02-02, F02-03 ghi rõ "kèm ngày hiệu lực và ngày hết hạn", F02-01, F02-04 thì không). Prototype hiện `participant_qualifications.valid_to NOT NULL` → đây là **thay đổi ràng buộc đề xuất** (migration ở LAB-5, phá tương thích với dữ liệu seed giả định đủ hạn); máy trạng thái `有効 → 失効` tự động chỉ áp cho căn cứ có hạn, căn cứ không thời hạn chỉ rời `有効` bằng `取消`. **Hướng loại:** bắt buộc có hạn cho mọi căn cứ (giữ `NOT NULL`, không đổi schema, nhưng phải bịa ngày hết hạn cho căn cứ mà tài liệu khách không có) | Nếu khách nói bốn căn cứ đều có hạn → giữ `NOT NULL`, bỏ nhánh NULL ở SCR005/SCR006/SCR036 và ghi chú `V-04` mở rộng cho mọi biến thể | `docs/screens/SCR005_ParticipantList/spec.md` · `docs/screens/SCR006_ParticipantProfile/spec.md` · `docs/screens/SCR036_QualificationReview/spec.md` · `docs/features/F002_ParticipantsAndQualification/` |

## D. Điểm trung thực về trạng thái

- **11 màn đã xây** (SCR001–004, 007, 008, 010, 011, 012, 014, 018) được đặc tả **từ code thật + ảnh chụp bản deploy** — là đặc tả viết *sau* code (đặc tả ngược). **26 màn còn lại** (24 của LAB-1 + SCR036/037) chỉ là **thiết kế**, chưa có code để kiểm bằng chạy thật.
- "Đã xây" nghĩa là chạy được đầu-cuối trên dữ liệu thật; lỗi đã biết ghi riêng ở B-01…B-05, không làm đổi số đếm. Cùng tiêu chí với bài LAB-3.
- Nguồn LAB-3 (tài liệu gốc trong repo, không nộp: `lab3-scope-lock.md`) ghi "10 màn" nhưng bảng liệt kê **11 dòng**; con số 11 khớp với sổ đăng ký và quick-report. Bản đầu của chính bộ LAB-4 cũng ghi nhầm "10 + 5" và phải sửa thành "11 + 4" (commit `d2063a8`).

### Câu hỏi cần khách xác nhận

Chưa quyết, không còn hướng nào mở để tự chọn. Tổng hợp gốc ở (tài liệu gốc trong repo, không nộp) `screen-specs/README.md` §"Điểm phải quyết", không phải `docs/` trong thư mục này.

1. `条件付き` cụ thể hạn chế gì (G4).
2. "Khớp quy tắc `精算`" (`BR-DEL-03`) nghĩa là gì.
3. SLA tranh chấp.
4. Test vector Appendix C (G6).
5. Xác nhận hai giả định đã chọn hướng: G8 (ai duyệt `再審査`) và G9 (`valid_to` có thể NULL với `市場登録` và `卸売場登録`; `許可`, `承認` bắt buộc có hạn).

## E. Nguồn

Trong repo gốc: `docs/lab1-baseline/` (verdict, WBS plan, RFP Q&A, screen/feature list) · `docs/lab3-scope-lock.md` (D1–D6, S1–S3, MOCK-1…9) · bộ thiết kế LAB-4 gốc (database-diagram D-01…D-06, adr, architecture, screen-specs/README, quick-report; tài liệu gốc trong repo, không nộp) · sổ đăng ký tên `plans/261006-1431-lab3-lab4-resubmission/naming-registry.md`.
