# Glossary

Thuật ngữ nghiệp vụ Nhật (giữ nguyên, không dịch lại — lấy từ RFP Appendix E và các deliverable JA của LAB-1) kèm tiếng Việt và nghĩa trong ngữ cảnh hệ thống này; cùng thuật ngữ kỹ thuật/mã dùng trong bộ spec. Chỉ liệt kê thuật ngữ mà người đọc ngoài ngành chợ cá hoặc ngoài dự án sẽ khó hiểu.

Sắp xếp theo nhóm nghiệp vụ, trong mỗi nhóm theo thứ tự xuất hiện của chuỗi nghiệp vụ (khác thứ tự bảng chữ cái của template vì tiếng Nhật không có thứ tự chung với tiếng Việt). Điều hướng: [Tham gia & tư cách](#1-người-tham-gia-và-tư-cách) · [Lô & giao dịch](#2-lô-và-giao-dịch) · [Giao hàng, đối chiếu, chốt sổ](#3-giao-hàng-đối-chiếu-chốt-sổ) · [Quy tắc, tiền, kiểm toán](#4-quy-tắc-tiền-kiểm-toán) · [Tổ chức & dự án](#5-tổ-chức-và-dự-án) · [Mã & ký hiệu kỹ thuật](#6-mã-và-ký-hiệu-kỹ-thuật).

## 1. Người tham gia và tư cách

| Term (JA / code) | Tiếng Việt | Definition | Used In |
|------|------|------------|---------|
| 参加者 | Người tham gia | Mọi đối tác tham gia chợ: chủ hàng và người mua. Lưu ở bảng `participants`. | [entities.md](../generated/entities.md) |
| 卸売業者 | Đơn vị bán buôn | Nhóm đối tác tạo lô, bán qua 相対取引 hoặc せり. Căn cứ tham gia: 市場登録. `party_type = wholesaler`. | [permissions.md](permissions.md) · [FL-03](../flows/FL-03-participant-qualification-lifecycle.md) |
| 仲卸 | Đơn vị trung gian có giấy phép | Mua, nhận, đối chiếu, thanh toán. Căn cứ tham gia: **許可**. `party_type = intermediary`. | [FL-03](../flows/FL-03-participant-qualification-lifecycle.md) |
| 売買参加者 | Người tham gia mua bán được chấp thuận | Mua theo quy chế chợ. Căn cứ tham gia: **承認**. `party_type = approved_trader`. | [FL-03](../flows/FL-03-participant-qualification-lifecycle.md) |
| 買出人 | Người mua tại chợ | Mua liên tục theo quy chế; đăng ký tại điểm giao dịch. `party_type = market_buyer`. | [FL-03](../flows/FL-03-participant-qualification-lifecycle.md) |
| 参加資格 | Tư cách tham gia | Điều kiện để một 参加者 được giao dịch; có 4 căn cứ khác bản chất. Bảng `participant_qualifications`. | [business-rules.md](business-rules.md) |
| 市場登録 | Đăng ký chợ | Căn cứ tham gia của 卸売業者 (`basis_type = registration`). Vi phạm → thu hồi chấp thuận. | [entities.md](../generated/entities.md#qualification_basis_rules) |
| 許可 | Giấy phép tham gia | Căn cứ do **cơ quan hành chính ngoài chợ** cấp (`basis_type = license`). Vi phạm → 条件付き. **Không** được gộp với 承認 (RFP 12-06). | [ADR-006](../decisions/ADR-006-split-license-approval-state-machines.md) |
| 承認 | Chấp thuận tham gia | Căn cứ do **chính chợ** cấp (`basis_type = approval`). Vi phạm → 再審査. Cũng là từ chỉ việc "duyệt" ở các bước phê duyệt. | [ADR-006](../decisions/ADR-006-split-license-approval-state-machines.md) |
| 卸売場登録 | Đăng ký tại điểm giao dịch | Nhãn JA của `basis_type = market_registration` trong DB; căn cứ của 買出人; hành động vi phạm "theo quy chế chợ". | [entities.md](../generated/entities.md#qualification_basis_rules) |
| 条件付き | Hạn chế có điều kiện | Trạng thái *đề xuất* của 許可 vi phạm: **vẫn giao dịch được** trong điều kiện; vượt điều kiện → `P0010`. Nội dung điều kiện chưa chốt (câu hỏi khách #1). | [FL-03](../flows/FL-03-participant-qualification-lifecycle.md) |
| 再審査 | Phải xét lại | Trạng thái *đề xuất* của 承認 vi phạm: **dừng giao dịch** tới khi có người duyệt lại; chặn bằng `P0011`. | [FL-03](../flows/FL-03-participant-qualification-lifecycle.md) |
| 有効 · 失効 · 取消 | Còn hiệu lực · Hết hạn · Thu hồi | Ba trạng thái của một tư cách. Prototype dùng `active` · `expired` · `revoked`. | [entities.md](../generated/entities.md) |
| 一時停止 | Tạm dừng | Trạng thái trong máy trạng thái chung FIG-010 của LAB-1; thiết kế LAB-4 thay bằng 条件付き/再審査 theo từng căn cứ. | [FL-03](../flows/FL-03-participant-qualification-lifecycle.md) |
| 申請 | Nộp đơn | Hành động nộp đơn xin xét lại (ví dụ từ 失効 sang 再審査). Cũng là "phiếu xin" ở 修正申請票. | [FL-03](../flows/FL-03-participant-qualification-lifecycle.md) |
| 有効日 · 失効日 | Ngày hiệu lực · ngày hết hạn | Hệ thống phải hiển thị cả hai, và trạng thái 一時停止 khi có (RFP 7.2). Cột `valid_from`, `valid_to`. | [entities.md](../generated/entities.md) |

## 2. Lô và giao dịch

| Term (JA / code) | Tiếng Việt | Definition | Used In |
|------|------|------------|---------|
| ロット · lô | Lô hàng | Một lô cá đã lên bờ, có mã duy nhất `LOT-YYYYMMDD-0001`. Là vật thể không tái tạo được nên không được bán vượt số lượng. | [F003](../features/F003_LotsAndInspection/technical-spec.md) |
| 受付 | Đã tiếp nhận | Trạng thái lô sau khi nhập ở SC-07. Số lượng ban đầu khoá, bất biến. | [FL-01](../flows/FL-01-lot-trade-delivery-reconciliation.md) |
| 下見 · 下見済 | Xem hàng trước giao dịch · Đã đánh giá | Bước xem/đánh giá cảm quan lô. `下見済` có trong enum nhưng chưa có đường ghi. | [FL-01](../flows/FL-01-lot-trade-delivery-reconciliation.md) |
| 目利き | Đánh giá cảm quan | Đánh giá độ tươi/chất lượng bởi người có kinh nghiệm; một trong bốn điểm bắt buộc giữ con người (CAP-HUMAN-01). | [architecture.md §3.6](architecture.md#36-cap-human-01--bốn-điểm-cố-ý-không-tự-động-hoá) |
| 公開 | Đã mở bán | Trạng thái lô sẵn sàng giao dịch; đổi từ SC-08. | [FL-01](../flows/FL-01-lot-trade-delivery-reconciliation.md) |
| 確定 | Đã bán (chốt) | Trạng thái lô khi đã bán hết số lượng. Cũng là "chốt" của giao dịch. | [FL-01](../flows/FL-01-lot-trade-delivery-reconciliation.md) |
| 引渡し完了 | Đã giao xong | Trạng thái cuối của lô khi **mọi** giao dịch `confirmed` của lô đã hoàn tất giao hàng (hoàn tất ghi ở cấp giao dịch, `trades.delivery_completed_at`; PREMISE G3). | [FL-01](../flows/FL-01-lot-trade-delivery-reconciliation.md) |
| 相対取引 | Giao dịch thỏa thuận trực tiếp | Luồng chính, ≈ **90%** giá trị giao dịch. `trade_type = negotiated`. Coi hệ thống chỉ có đấu giá là điều kiện bị loại. | [SCR010_NegotiatedTradeConfirm](../screens/SCR010_NegotiatedTradeConfirm/spec.md) |
| せり | Phiên đấu giá công khai | ≈ **10%** giá trị. Hệ thống chỉ **ghi kết quả cuối do người nhập**. `trade_type = auction`. | [SCR011_AuctionResultEntry](../screens/SCR011_AuctionResultEntry/spec.md) |
| 手やり · 符丁 | Ký hiệu tay · ký hiệu riêng của chợ | Ký hiệu hiện trường khi đấu giá. Hệ thống **không** nhận dạng, không OCR, không gợi ý (FR-SERI-01). | [business-rules.md](business-rules.md) |
| 過剰販売 | Bán vượt | Bán nhiều hơn số lượng của lô. Chống bằng khoá dòng lô + kiểm số còn lại trong một transaction. | [ADR-002](../decisions/ADR-002-no-write-policy-on-trades-deliveries.md) |
| 直荷引き | Giao trực tiếp không qua せり/相対 | Ngoài phạm vi số hoá. | [overview.md](overview.md) |
| 業務日 | Ngày nghiệp vụ | Đơn vị chốt sổ (`business_days.business_date`); mọi quy tắc theo ngày này, không theo giờ hệ thống. | [FL-01](../flows/FL-01-lot-trade-delivery-reconciliation.md) |
| 休市日 | Ngày nghỉ chợ | Ngày chợ không hoạt động, theo lịch chợ (SC-21). | [SCR021_MarketCalendar](../screens/SCR021_MarketCalendar/spec.md) |

## 3. Giao hàng, đối chiếu, chốt sổ

| Term (JA / code) | Tiếng Việt | Definition | Used In |
|------|------|------------|---------|
| 荷受け | Nhận hàng, tiếp nhận | Khâu nhận hàng; cũng là tên chức danh `receiving`. | [permissions.md](permissions.md) |
| 配送部門 | Bộ phận giao hàng | Chức danh LAB-1 tách khỏi 荷受け; prototype gộp (D-03). | [entities.md § D-03](../generated/entities.md#d-03--gộp-hai-chức-danh-thành-một) |
| 引渡し | Giao hàng | Giao hàng cho người mua; một giao dịch có thể giao **nhiều lần** (lịch xe gom, hạ hàng). | [F005](../features/F005_DeliveryTracking/technical-spec.md) |
| 引渡不足 · 引渡超過 · 返却 | Giao thiếu · giao thừa · trả hàng | Ba trong bốn lý do ngoại lệ (cùng "huỷ một phần"): `short` · `over` · `returned` · `partial_cancel`. | [entities.md](../generated/entities.md#deliveries) |
| 累積 · 残数量 | Số đã giao tích luỹ · số còn lại | Tính theo từng giao dịch từ dữ liệu, không lưu cột. | [ADR-003](../decisions/ADR-003-derived-remaining-quantity-views.md) |
| 精算 | Thanh toán / quyết toán | Quyết toán một giao dịch; "khớp rule 精算" (BR-DEL-03) chưa định nghĩa được. | [business-rules.md](business-rules.md) |
| 照合 | Đối chiếu | Đối chiếu giao dịch + giao hàng + ngoại lệ thành bảng đối chiếu ngày (SC-18). | [SCR018_DailyReconciliation](../screens/SCR018_DailyReconciliation/spec.md) |
| 照合事務局 | Văn phòng đối chiếu | Bộ phận chốt kỳ, sửa có phê duyệt, xuất đối chiếu; chức danh `reconcile`. | [permissions.md](permissions.md) |
| 係争 | Tranh chấp | Hồ sơ tranh chấp giao hàng/giao dịch (SC-19, SC-20). | [SCR019_DisputeList](../screens/SCR019_DisputeList/spec.md) |
| chốt sổ · lock | Chốt sổ ngày nghiệp vụ | `close_business_day` chuyển `business_days` sang `closed`; sau đó ghi trực tiếp bị chặn (`P0009`). | [FL-01](../flows/FL-01-lot-trade-delivery-reconciliation.md) |
| pending_conditions | Điều kiện còn tồn khi chốt | Danh sách giao dịch chưa giao đủ lúc chốt sổ; cảnh báo, không chặn ([ADR-005](../decisions/ADR-005-close-day-warn-not-block.md)). | [business-rules.md](business-rules.md) |
| sold_to | Danh sách giao dịch đã mua lô | Mảng mã giao dịch trả về cùng `P0004` để người dùng biết ai đã mua. | [FL-01](../flows/FL-01-lot-trade-delivery-reconciliation.md) |

## 4. Quy tắc, tiền, kiểm toán

| Term (JA / code) | Tiếng Việt | Definition | Used In |
|------|------|------------|---------|
| 業務規程 | Quy định vận hành của chợ | Quy chế chợ; thay đổi tối đa 4 lần/năm, có version và 発効日. | [ADR-007](../decisions/ADR-007-market-rules-as-versioned-data.md) |
| 料率 | Tỷ lệ | Tỷ lệ dùng tính 完納奨励金; có phiên bản, 発効日 và quay lui. | [F008](../features/F008_IncentiveAndRateVersion/technical-spec.md) |
| 完納奨励金 | Tiền khuyến khích thanh toán đúng hạn | Tính theo version quy chế/料率; hệ số thuế 110/100, làm tròn xuống JPY, không có 猶予期間; quá 支払期日 → 0. | [business-rules.md](business-rules.md) |
| 支払期日 · 猶予期間 | Hạn thanh toán · thời gian ân hạn | Hạn để được hưởng khuyến khích · **không áp dụng** ân hạn. | [business-rules.md](business-rules.md) |
| 切り捨て | Làm tròn xuống | Làm tròn xuống theo đơn vị JPY sau khi nhân hệ số thuế. | [business-rules.md](business-rules.md) |
| 発効日 | Ngày có hiệu lực | Ngày một version quy chế/料率/tư cách bắt đầu có hiệu lực; có thể ở tương lai. | [ADR-007](../decisions/ADR-007-market-rules-as-versioned-data.md) |
| ロールバック | Quay lui | Đưa quy chế/料率 về bản trước, phải truy vết được. | [ADR-007](../decisions/ADR-007-market-rules-as-versioned-data.md) |
| 起票・承認分離 | Tách người lập và người duyệt (maker-checker) | Người lập ≠ người duyệt; cưỡng chế trong RPC. | [FL-02](../flows/FL-02-post-close-correction.md) |
| 差分 | Chênh lệch (delta) | Bản ghi điều chỉnh ghi ở **kỳ kế tiếp**, không sửa kỳ trước. Cùng khái niệm **reverse/delta** của F07. | [FL-02](../flows/FL-02-post-close-correction.md) |
| reverse · delta | Bản ghi đảo · bản ghi chênh lệch | Hai kiểu bản ghi điều chỉnh sinh khi duyệt phiếu; không ghi đè bản gốc. `correction_entries`. | [FL-02](../flows/FL-02-post-close-correction.md) |
| 修正申請票 | Phiếu xin điều chỉnh | Phiếu lập sau chốt sổ (SC-22); bản ghi gốc không đổi cho tới khi duyệt. | [SCR022_CorrectionRequestCreate](../screens/SCR022_CorrectionRequestCreate/spec.md) |
| 監査証跡 | Dấu vết kiểm toán | Ai · khi nào · 変更前 · 変更後 · lý do. Hiện `audit_log` + `role_change_log`. | [architecture.md §C5](architecture.md#c5--nhật-ký-kiểm-toán-một-phần) |
| 変更前 · 変更後 · 実行主体 | Trước thay đổi · sau thay đổi · chủ thể thực hiện | Ba trong năm trường kiểm toán bắt buộc (cùng thời điểm và lý do). | [business-rules.md](business-rules.md) |
| 高感度 | Thao tác nhạy cảm | Thao tác phải có 監査証跡: sửa giao dịch, đổi quy chế, lock/unlock account, mở lại kỳ đã chốt… | [permissions.md](permissions.md) |
| 人による最終判断 | Quyết định cuối cùng của con người | Nguyên tắc: hệ thống không tự quyết ở bốn điểm CAP-HUMAN-01. | [architecture.md §3.6](architecture.md#36-cap-human-01--bốn-điểm-cố-ý-không-tự-động-hoá) |
| 責任分離の原則 | Nguyên tắc phân tách trách nhiệm | RFP 02-08: 許可 ≠ 承認; thao tác 高感度 cần dấu vết. | [architecture.md §C1](architecture.md#c1--chặn-route-theo-chức-danh-có) |
| 行ステータス | Trạng thái dòng (đối chiếu kế toán) | `保留` · `確認済み` · `調整済み` trong hợp đồng dữ liệu IF-ACC-01. | [architecture.md §4](architecture.md#4-điểm-tích-hợp--if-acc-01-chưa-làm) |
| APPI · 個人情報保護 | Luật bảo vệ thông tin cá nhân (Nhật) | Hồ sơ 許可/承認 của 参加者 là thông tin cá nhân → RBAC + access log (RFP 09-06). | [architecture.md §C2](architecture.md#c2--rls-trên-postgres-có) |

## 5. Tổ chức và dự án

| Term (JA / code) | Tiếng Việt | Definition | Used In |
|------|------|------------|---------|
| 発注者 | Chủ đầu tư | 株式会社 桜水産市場 (hư cấu, thành phố 桜浜); quyết định cuối cùng ở mỗi cổng 受入. | [overview.md](overview.md) |
| 財務部門 | Bộ phận tài chính | Vận hành hệ thống kế toán nhận dữ liệu qua IF-ACC-01. | [architecture.md §4](architecture.md#4-điểm-tích-hợp--if-acc-01-chưa-làm) |
| インターフェース検討会 | Cuộc họp chốt giao diện hệ thống | Nơi quyết phương thức kết nối IF-ACC-01; chưa diễn ra. | [architecture.md §4](architecture.md#4-điểm-tích-hợp--if-acc-01-chưa-làm) |
| 受入 | Nghiệm thu | Cổng chấp nhận bàn giao (UAT, ACC-GATE). | [overview.md](overview.md) |
| 移行 | Di trú dữ liệu | Chuyển dữ liệu từ hệ thống hiện hành (phase 11 WBS LAB-1). | [business-rules.md](business-rules.md) |
| 初期運用支援 | Hỗ trợ vận hành ban đầu (hypercare) | 30 ngày sau go-live. | [overview.md](overview.md) |
| サービス提供時間帯 | Khung giờ cung cấp dịch vụ | **02:00–10:00 JST**, 6 ngày/tuần, trừ 休市日. | [overview.md](overview.md) |
| JST | Giờ chuẩn Nhật Bản | Múi giờ mọi tính toán nghiệp vụ. | [overview.md](overview.md) |

## 6. Mã và ký hiệu kỹ thuật

| Term (JA / code) | Tiếng Việt | Definition | Used In |
|------|------|------------|---------|
| SC-nn · SCR0nn | Mã màn hình | `SC-10` (LAB-1) tương ứng thư mục `SCR010_NegotiatedTradeConfirm`. `SC-N1`/`SC-N2` = `SCR036`/`SCR037` (màn thiết kế mới). | [overview.md](overview.md) |
| F### · F01-01 | Mã tính năng | `F004_TradeRecording` = nhóm LAB-1 F04; tính năng con giữ mã LAB-1 (`F04-01`). `F012` mới ở LAB-4. | [overview.md](overview.md) |
| FL-01…03 · ADR-001…007 | Mã flow · mã quyết định kiến trúc | Chuỗi nghiệp vụ; bản ghi quyết định (bối cảnh · lựa chọn · phương án đã bỏ · hệ quả). | [overview.md](overview.md) |
| D-01…D-06 · B-01…B-05 | Sai lệch thiết kế · lỗi dữ liệu | D-01 gộp 許可/承認; D-02 không có chợ; D-03 gộp chức danh; D-04 chốt sổ cảnh báo; D-05 `receiving` đọc `trades`; D-06 9 hạng mục ngoài phạm vi. B-01…05 xem [business-rules.md](business-rules.md#năm-lỗi-tầng-dữ-liệu-đã-phát-hiện). | [entities.md](../generated/entities.md#đối-chiếu-thiết-kế--bảng-thật) |
| MOCK-1…9 | Hạng mục được mock ở LAB-3 | Phần ngoài phạm vi prototype kèm "nếu làm thật cần gì" (tài liệu gốc trong repo, không nộp: `lab3-scope-lock`). | [entities.md § D-06](../generated/entities.md#d-06--chín-hạng-mục-ngoài-phạm-vi-prototype) |
| P0001…P0009 · P0010+ | Mã lỗi nghiệp vụ | SQLSTATE tự định nghĩa; `P0001–P0009` đã dùng, `P0010+` đề xuất. | [business-rules.md](business-rules.md#bảng-mã-lỗi-đã-triển-khai) |
| BR-… · FR-… · NFR-… · GOV-RULE-01 | Mã yêu cầu của RFP/LAB-1 | Business rule, yêu cầu chức năng, phi chức năng, quản trị quy chế — giữ nguyên nghĩa xuyên suốt. | [business-rules.md](business-rules.md) |
| CAP-HUMAN-01 | Bốn điểm bắt buộc giữ con người | Đánh giá lô · chốt kết quả せり · phê duyệt sửa dữ liệu nhạy cảm · xác nhận hoàn tất 荷受け khi tranh chấp. | [architecture.md §3.6](architecture.md#36-cap-human-01--bốn-điểm-cố-ý-không-tự-động-hoá) |
| RPC | Hàm nghiệp vụ trong database | Hàm PL/pgSQL `SECURITY DEFINER` gọi qua `supabase.rpc()`; đường ghi duy nhất vào `trades`/`deliveries`. | [ADR-002](../decisions/ADR-002-no-write-policy-on-trades-deliveries.md) |
| RLS | Row Level Security | Chính sách giới hạn dòng theo chức danh, bật trên 11/11 bảng. | [permissions.md](permissions.md) |
| SECURITY DEFINER | Hàm chạy với quyền chủ sở hữu | Bỏ qua RLS → mỗi hàm phải tự kiểm chức danh ở dòng đầu. | [architecture.md §5](architecture.md#5-phân-quyền-hai-tầng--lý-do-kỹ-thuật) |
| security_invoker | View chạy với quyền người gọi | Giữ RLS của bảng gốc khi đọc qua view. | [ADR-003](../decisions/ADR-003-derived-remaining-quantity-views.md) |
| `proxy.ts` · ROUTE_POLICY | Lớp chặn route · bảng định tuyến theo chức danh | Next.js 16 dùng `proxy.ts` thay `middleware.ts`; `ROUTE_POLICY` là hằng số duy nhất chặn route. | [architecture.md §C1](architecture.md#c1--chặn-route-theo-chức-danh-có) |
| `can_approve` | Cờ thẩm quyền duyệt | Cột đề xuất trên `app_users` thay vì thêm chức danh; luôn đi kèm quy tắc khác người lập. | [permissions.md](permissions.md) |
| `as_of` | Ngày tham chiếu nghiệp vụ | Tham số ngày để tra quy tắc/báo cáo; **không** dùng `now()`. | [ADR-007](../decisions/ADR-007-market-rules-as-versioned-data.md) |
| ⌨️ · 📴 | Keyboard-first · chạy được offline | Ký hiệu của màn hình LAB-1: ⌨️ ở SC-07/10/14; 📴 ở SC-07/14. | [architecture.md §3.4](architecture.md#34-keyboard-first-ở-ba-màn-hiện-trường-sc-07--sc-10--sc-14) |
| IF-ACC-01 | Tích hợp hệ thống kế toán | Xuất bảng đối chiếu ngày sau khi lock; giả định A4 CSV một chiều qua SFTP. | [architecture.md §4](architecture.md#4-điểm-tích-hợp--if-acc-01-chưa-làm) |
