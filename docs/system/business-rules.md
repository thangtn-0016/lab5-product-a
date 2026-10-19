# Business Rules

Quy tắc mà hệ thống thi hành, viết bằng ngôn ngữ nghiệp vụ. Mã quy tắc giữ nguyên mã LAB-1/RFP (`BR-PERM-01`, `FR-LOT-01`…); quy tắc không có mã gốc ghi "—". Mỗi quy tắc nói rõ **ĐÃ TRIỂN KHAI** (có trong migration/code LAB-3) hay **ĐỀ XUẤT** (thiết kế LAB-4, chưa xây). Chi tiết kỹ thuật (file:line) nằm ở spec tính năng `docs/features/F###_*/technical-spec.md`; bảng mã lỗi và điểm chưa chốt ở cuối trang này.

Điều hướng: [A. Quy tắc nghiệp vụ](#a-quy-tắc-nghiệp-vụ) · [B. Quy tắc kỹ thuật bắt buộc](#b-quy-tắc-kỹ-thuật-bắt-buộc-cho-mọi-thay-đổi-schema) · [Bảng mã lỗi đã triển khai](#bảng-mã-lỗi-đã-triển-khai) · [Mã lỗi đề xuất](#mã-lỗi-đề-xuất-chưa-cài) · [Phủ yêu cầu LAB-1](#phủ-yêu-cầu-lab-1) · [Năm lỗi dữ liệu](#năm-lỗi-tầng-dữ-liệu-đã-phát-hiện) · [Điểm chưa chốt trước LAB-5](#điểm-chưa-chốt-trước-lab-5)

## A. Quy tắc nghiệp vụ

### BR-PERM-01 — Kiểm hiệu lực tư cách tham gia tại thời điểm giao dịch
**Trạng thái:** ĐÃ TRIỂN KHAI một luật chung (D-01); tách theo căn cứ là ĐỀ XUẤT
**Applies when:** chốt một giao dịch (SC-10, SC-11) hoặc ghi một lần giao hàng (SC-14) — hệ thống kiểm người mua.
**Says:** người mua phải có một tư cách đang `active` và ngày hiệu lực bao trùm **ngày hôm nay** (`current_date`, không phải ngày nghiệp vụ của lô). Nếu không, từ chối với mã `P0003` và nói rõ: hết hạn ngày nào, theo căn cứ gì, liên hệ cơ quan nào — **không tạo bản ghi nào**. Hiện bốn căn cứ (市場登録, 許可, 承認, đăng ký tại điểm giao dịch) dùng chung một phép kiểm; thiết kế đề xuất tách riêng (xem rule kế tiếp).
**Source artifact:** [F002_ParticipantsAndQualification](../features/F002_ParticipantsAndQualification/technical-spec.md) · [FL-03](../flows/FL-03-participant-qualification-lifecycle.md) · [entities.md § D-01](../generated/entities.md#d-01--bốn-căn-cứ-tham-gia-bị-gộp-thành-một-luật-chung)

---

### 許可 ≠ 承認 — hai căn cứ không bao giờ gộp thành một luật
**Trạng thái:** ĐỀ XUẤT (prototype đang gộp — vi phạm RFP 12-06, điều kiện bị loại); mã gốc: 責任分離の原則 (02-08)
**Applies when:** một tư cách `許可` (仲卸) hoặc `承認` (売買参加者) vi phạm và cần tạm dừng/xét lại.
**Says:** `許可` vi phạm → `条件付き`: **vẫn giao dịch được** nhưng trong điều kiện; vượt điều kiện bị chặn bằng `P0010`. `承認` vi phạm → `再審査`: **dừng giao dịch** tới khi có người duyệt lại, chặn bằng `P0011`. Hết hạn của cả hai → `P0003`. Hai căn cứ có bảng, enum, hàm kiểm và mã lỗi riêng.
**Source artifact:** [architecture.md §2.3](architecture.md#23-xét-duyệt-tư-cách-tham-gia--hai-nhánh-không-gộp) · [ADR-006](../decisions/ADR-006-split-license-approval-state-machines.md) · [SCR036_QualificationReview](../screens/SCR036_QualificationReview/spec.md)

---

### BR-LOT-02 / FR-LOT-03 — Số lượng khả dụng của lô không bao giờ âm
**Trạng thái:** ĐÃ TRIỂN KHAI
**Applies when:** chốt giao dịch trên một lô; ghi lần giao cho một giao dịch.
**Says:** số lượng ban đầu của lô (`lots.quantity`) **không đổi** sau khi tạo; số còn lại luôn được **tính** (ban đầu − tổng giao dịch đã chốt), không lưu thành cột. Giao dịch đòi nhiều hơn số còn lại bị từ chối `P0004`, kèm số còn lại và các giao dịch đã mua. Tương tự, một lần giao vượt số còn lại của giao dịch bị `P0006`. Hai người chốt cùng lô cùng lúc không thể cùng thấy "còn đủ" vì dòng lô bị khoá trong transaction.
**Source artifact:** [ADR-003](../decisions/ADR-003-derived-remaining-quantity-views.md) · [F003_LotsAndInspection](../features/F003_LotsAndInspection/technical-spec.md) · [F004_TradeRecording](../features/F004_TradeRecording/technical-spec.md)

---

### FR-AITAI-02 — Từ chối chốt phải nêu lý do và không tạo giao dịch
**Trạng thái:** ĐÃ TRIỂN KHAI
**Applies when:** `trading` bấm chốt một 相対取引 hoặc ghi kết quả せり.
**Says:** hai phép kiểm (tư cách người mua, số lượng lô) chạy **trong một transaction phía server**; thiếu một trong hai thì từ chối, hiện lý do cụ thể và không ghi gì. Giao diện phân biệt hai lý do (hết hạn: liên hệ ai; thiếu hàng: còn bao nhiêu, ai đã mua) và rẽ nhánh theo **mã lỗi**, không theo câu chữ. Bước chốt **không thể chạy offline**.
**Source artifact:** [FL-01](../flows/FL-01-lot-trade-delivery-reconciliation.md#cổng-chốt-giao-dịch--sc-10--sc-11) · [SCR010_NegotiatedTradeConfirm](../screens/SCR010_NegotiatedTradeConfirm/spec.md) · [ADR-002](../decisions/ADR-002-no-write-policy-on-trades-deliveries.md)

---

### FR-LOT-01 — Mã lô là duy nhất
**Trạng thái:** ĐÃ TRIỂN KHAI
**Applies when:** tạo lô (SC-07).
**Says:** mã lô dạng `LOT-YYYYMMDD-0001` do server sinh; trùng mã bị chặn bằng ràng buộc UNIQUE trong database, không kiểm ở trình duyệt. Mã giao dịch `TRD-YYYYMMDD-0001` sinh tương tự.
**Source artifact:** [entities.md § lots](../generated/entities.md#lots) · [F003_LotsAndInspection](../features/F003_LotsAndInspection/technical-spec.md)

---

### BR-CLOSE-01 / FR-CORR-03 / FR-SETTLE-02 — Không sửa trực tiếp ngày đã chốt
**Trạng thái:** ĐÃ TRIỂN KHAI (chặn + ghi lần thử); đường sửa hợp lệ là ĐỀ XUẤT
**Applies when:** ngày nghiệp vụ đã `closed` và ai đó thử thêm/sửa/xoá lô, giao dịch hoặc lần giao của ngày đó.
**Says:** mọi thao tác ghi trực tiếp bị chặn (`P0009`; qua RPC thì `P0002`) và **lần thử được ghi vào nhật ký** — chặn *và* ghi log, vì thử sửa ngày đã chốt là sự kiện nghiệp vụ. Sau chốt chỉ còn đường điều chỉnh có kiểm soát.
**Source artifact:** [FL-01](../flows/FL-01-lot-trade-delivery-reconciliation.md) · [FL-02](../flows/FL-02-post-close-correction.md) · [architecture.md §3.1](architecture.md#31-append-only--temporal-br-close-01-fr-corr-02-06-04)

---

### Chốt sổ cảnh báo chứ không chặn khi còn giao dịch chưa giao xong
**Trạng thái:** ĐÃ TRIỂN KHAI — sai lệch có ý thức so với nghĩa đen `US-11`; mã gốc: —
**Applies when:** `reconcile` chốt ngày (SC-18) khi còn giao dịch giao chưa đủ.
**Says:** hệ thống liệt kê các giao dịch còn tồn (`pending_conditions`) để cảnh báo nhưng **vẫn cho chốt**, vì giao hàng kéo dài qua ngày là bình thường (lịch xe gom). Số liệu chốt có thể chứa giao dịch chưa giao xong; sửa sau qua F07. Hướng này đã chốt (PREMISE C-07); chỉ cần khách xác nhận nếu họ hiểu "phải đạt" là chặn.
**Source artifact:** [ADR-005](../decisions/ADR-005-close-day-warn-not-block.md) · [SCR018_DailyReconciliation](../screens/SCR018_DailyReconciliation/spec.md)

---

### FR-DEL-05 / FR-DEL-03 — Một giao dịch giao nhiều lần; ngoại lệ phải có loại
**Trạng thái:** ĐÃ TRIỂN KHAI một phần
**Applies when:** ghi lần giao hàng (SC-14).
**Says:** mỗi lần giao có số thứ tự riêng trong giao dịch (không trùng); hiển thị đã giao / còn lại; không giao vượt phần còn lại. Ngoại lệ (`short`, `over`, `returned`, `partial_cancel`) chỉ hợp lệ khi lần giao ở trạng thái ngoại lệ, và **chỉ ghi lại phán quyết của người**, không tự kết luận. Giao thừa (`over`) hiện chưa ghi được (B-03). Giới hạn số lần chia giao và thời điểm phản ánh vào 精算 chưa chốt (Q-02).
**Source artifact:** [F005_DeliveryTracking](../features/F005_DeliveryTracking/technical-spec.md) · [FL-01](../flows/FL-01-lot-trade-delivery-reconciliation.md)

---

### BR-DEL-03 / FR-DEL-02 — Giao hàng chỉ hoàn tất khi khớp quy tắc 精算
**Trạng thái:** ĐỀ XUẤT — chưa định nghĩa được "khớp rule 精算" (câu hỏi khách #2 ở mục *Điểm chưa chốt trước LAB-5*); cấp hoàn tất đã chốt (PREMISE G3)
**Applies when:** chốt một giao dịch ở trạng thái hoàn tất giao hàng.
**Says:** thiếu trường bắt buộc hoặc số lượng sai thì **không** được chốt hoàn tất. Hoàn tất giao hàng ghi ở **cấp giao dịch** (`trades.delivery_completed_at`, chốt ở SCR016); lô chỉ chuyển `引渡し完了` khi **mọi** giao dịch `confirmed` của lô đã hoàn tất (PREMISE G3). Prototype hiện chỉ lật trạng thái lô khi số còn lại về 0 (không đọc quy tắc 精算) — đó là B-02/B-04.
**Source artifact:** [F005_DeliveryTracking](../features/F005_DeliveryTracking/technical-spec.md) · [SCR016_DeliveryDetail](../screens/SCR016_DeliveryDetail/spec.md)

---

### FR-SERI-01 / CAP-HUMAN-01 — Kết quả せり do người nhập, hệ thống không suy luận
**Trạng thái:** ĐÃ TRIỂN KHAI
**Applies when:** ghi kết quả phiên đấu giá (SC-11).
**Says:** chỉ ghi kết quả cuối do người vận hành nhập; **không** nhận dạng 手やり/符丁, **không** OCR, **không** gợi ý tự động. Phải có đủ năm thông tin: người thắng, số lượng, đơn giá, thời điểm phiên, người xác nhận tại hiện trường. Thời điểm phiên và người xác nhận (nhập tay) khác với dấu vết hệ thống (ai gọi API, lúc nào) — vì máy dùng chung nên người chịu trách nhiệm có thể không phải người gõ. Dòng `auction` thiếu hai trường tay bị từ chối lúc commit (`P0008`); dòng `negotiated` không được mang hai trường này.
**Source artifact:** [ADR-004](../decisions/ADR-004-invariants-at-database-layer.md) · [SCR011_AuctionResultEntry](../screens/SCR011_AuctionResultEntry/spec.md)

---

### Đổi chức danh phải có lưu vết và không được tự đổi quyền của chính mình
**Trạng thái:** ĐÃ TRIỂN KHAI; mã gốc: FR-IAM-02, F01-04
**Applies when:** `sys_admin` đổi chức danh một tài khoản (SC-04).
**Says:** đổi quyền có hiệu lực ngay (không có độ trễ token); mỗi lần đổi ghi lịch sử (quyền cũ → mới, ai, lúc nào, ghi chú) và nhật ký kiểm toán. Quản trị viên **không** đổi được quyền của chính mình (`P0007`) để không tự khoá mình khỏi màn quản trị. Tài khoản bị tạm dừng bị đăng xuất ngay ở request kế tiếp.
**Source artifact:** [ADR-001](../decisions/ADR-001-role-from-table-not-jwt.md) · [F001_AdminAndPermissions](../features/F001_AdminAndPermissions/technical-spec.md) · [permissions.md](permissions.md)

---

### Quy tắc đang đứng sau cổng "hai người" (maker-checker)
**Trạng thái:** ĐỀ XUẤT; mã gốc: GOV-RULE-01 (起票・承認分離), FR-CORR-01/02, CAP-HUMAN-01 điểm 3, `US-17`
**Applies when:** lập/duyệt phiếu điều chỉnh sau chốt; phiếu chuyển trạng thái tư cách; phiên bản quy chế/料率.
**Says:** người lập và người duyệt **luôn khác nhau**, cưỡng chế **trong thân RPC** chứ không chỉ ẩn nút; hệ thống không tự duyệt. Khi duyệt điều chỉnh, bản ghi gốc **không đổi** — hệ thống sinh bản ghi reverse/delta ở ngày nghiệp vụ đang mở, truy ngược được về phiếu. Phiếu `pending` không làm đổi gì cho tới khi được duyệt.
**Source artifact:** [FL-02](../flows/FL-02-post-close-correction.md) · [FL-03](../flows/FL-03-participant-qualification-lifecycle.md) · [F007_PostCloseCorrection](../features/F007_PostCloseCorrection/technical-spec.md)

---

### GOV-RULE-01 / FR-INC-02 / FR-RULE-01 — Quy chế và 料率 là dữ liệu có phiên bản
**Trạng thái:** ĐỀ XUẤT (MOCK-4)
**Applies when:** thay đổi quy chế vận hành hoặc bảng 料率.
**Says:** tối đa **4 lần/năm**; người lập ≠ người duyệt; phải có **発効日** (có thể ở tương lai) và đường **quay lui** truy vết được; bản nháp không có hiệu lực; thay đổi không được biến dạng dữ liệu đã chốt trước 発効日. Lưu đủ người tạo, người duyệt, 発効日, trạng thái cho **mọi** phiên bản. Tra quy tắc luôn theo **ngày nghiệp vụ**, không theo giờ hệ thống.
**Source artifact:** [ADR-007](../decisions/ADR-007-market-rules-as-versioned-data.md) · [F008_IncentiveAndRateVersion](../features/F008_IncentiveAndRateVersion/technical-spec.md) · [F012_MarketManagement](../features/F012_MarketManagement/technical-spec.md)

---

### BR-INC-01 / FR-INC-01 / FR-INC-03 — Tính 完納奨励金
**Trạng thái:** ĐỀ XUẤT (MOCK-4; test vector Appendix C chưa có, điểm chưa chốt #9)
**Applies when:** thanh toán trong 支払期日 và tính tiền khuyến khích.
**Says:** hệ số thuế **110/100**, sau đó **làm tròn xuống (切り捨て) theo đơn vị JPY**; **không có 猶予期間**; dùng **JST và 業務日**; quá hạn thì tiền khuyến khích = 0. Điều chỉnh sau chốt kỳ sinh **差分 ở kỳ kế tiếp** — báo cáo kỳ trước giữ nguyên. Hệ số thuế và làm tròn là quy tắc **quốc gia**, không phải quy tắc chợ.
**Source artifact:** [F008_IncentiveAndRateVersion](../features/F008_IncentiveAndRateVersion/technical-spec.md) · [SCR025_IncentiveResult](../screens/SCR025_IncentiveResult/spec.md)

---

### BR-NOTIFY-01 / FR-NOTIFY-01…03 — Thông báo
**Trạng thái:** ĐỀ XUẤT (MOCK-7)
**Applies when:** có sự kiện baseline (hồ sơ gần hết hạn, tranh chấp mở, lỗi xuất dữ liệu, giao hàng ứ đọng).
**Says:** kênh baseline là **アプリ内 + 電子メール** (không SMS/FAX); loại Critical gửi xong trong 5 phút, email lỗi thì thử lại tối đa **3 lần**, vẫn thất bại thì đẩy vào hàng cảnh báo vận hành; đổi cấu hình không làm mất lịch sử gửi.
**Source artifact:** [F010_Notifications](../features/F010_Notifications/technical-spec.md)

---

### Truy vấn báo cáo luôn tham số hoá theo ngày nghiệp vụ và phiên bản quy tắc
**Trạng thái:** ĐỀ XUẤT (nguyên tắc kiến trúc); mã gốc: 06-04 #3, BR-CLOSE-01
**Applies when:** mọi truy vấn báo cáo và tính toán nghiệp vụ.
**Says:** dùng `business_date` + phiên bản quy tắc tại `as_of`, **không bao giờ** `now()`; in lại báo cáo kỳ cũ phải ra đúng con số cũ. Một `now()` lọt vào tầng báo cáo phá BR-CLOSE-01 mà không test nào đỏ.
**Source artifact:** [architecture.md §3.1](architecture.md#31-append-only--temporal-br-close-01-fr-corr-02-06-04)

---

### NFR-USE-01 / INV-DEVICE-01 — Keyboard-first và máy dùng chung theo ca
**Trạng thái:** ĐÃ TRIỂN KHAI ở SC-07, SC-10, SC-14
**Applies when:** nhập lô, chốt giao dịch, ghi giao hàng.
**Says:** làm được thuần bàn phím; `Enter` chỉ submit ở ô cuối, `Ctrl+Enter` chốt từ vùng form, `Escape` xoá form về ô 1; chốt xong focus quay về ô mã lô; banner lỗi tự nhận focus; form **xoá sạch sau submit**, không autofill người mua lần trước; không hover-only, không kéo-thả, không menu chuột phải.
**Source artifact:** [architecture.md §3.4](architecture.md#34-keyboard-first-ở-ba-màn-hiện-trường-sc-07--sc-10--sc-14) · [SCR007_LotEntry](../screens/SCR007_LotEntry/spec.md) · [SCR014_DeliveryEntry](../screens/SCR014_DeliveryEntry/spec.md)

---

### NFR-AVL-02 / PP-NET-01 — Biên offline do nghiệp vụ vẽ
**Trạng thái:** ĐỀ XUẤT (prototype không có hàng đợi, MOCK-2)
**Applies when:** mất mạng ngắn ở cầu cảng/khu tiếp nhận.
**Says:** nhập nháp lô và ghi lần giao **được** làm offline rồi đồng bộ (chống trùng bằng khoá idempotency); chốt giao dịch và chốt sổ **không** — vì cần trạng thái máy chủ. Dữ liệu đồng bộ về sau khi ngày đã chốt phải đi qua đường điều chỉnh, không chèn lén vào ngày đã khoá.
**Source artifact:** [architecture.md §3.5](architecture.md#35-mất-mạng-ở-cầu-cảng--35-lầntuần-cao-điểm-pp-net-01) · [SCR035_SyncQueue](../screens/SCR035_SyncQueue/spec.md)

---

## B. Quy tắc kỹ thuật bắt buộc cho mọi thay đổi schema

Rút từ sự cố thật (tài liệu gốc trong repo, không nộp: `db-schema` §7, §11; [ADR-002](../decisions/ADR-002-no-write-policy-on-trades-deliveries.md), [ADR-004](../decisions/ADR-004-invariants-at-database-layer.md)). **ĐÃ TRIỂN KHAI** làm thực hành chuẩn của prototype.

| # | Quy tắc | Vì sao | Nguồn |
|---|---|---|---|
| K1 | Mọi RPC `SECURITY DEFINER` + `set search_path = ''` + tự kiểm `current_app_role()` ở dòng đầu (`P0005`) | `SECURITY DEFINER` bỏ qua RLS | `0007`, `0008` |
| K2 | Hàm mới: `REVOKE EXECUTE … FROM PUBLIC` **và** `FROM anon`, rồi kiểm `has_function_privilege('anon', …)` = false | PUBLIC được cấp EXECUTE mặc định; project còn có default privilege cấp thẳng cho `anon` | `0011`, `0040` |
| K3 | Đổi chữ ký hàm (thêm/bớt tham số) = hàm mới → revoke lại | `REVOKE` cũ chỉ áp chữ ký cũ | `0041` |
| K4 | Bất biến dữ liệu nằm ở tầng database; `DEFERRABLE` chưa đủ — thân trigger phải đọc lại trạng thái, không tin `NEW` | ADR-004 | `0035` |
| K5 | Client rẽ nhánh theo `error.code` (SQLSTATE `P00xx`); `error.details` là chuỗi JSON cần parse; không so khớp `error.message` | câu chữ có thể đổi | tài liệu gốc `db-schema` §4 (trong repo, không nộp) |
| K6 | Không sửa migration đã áp (`0001…0011` khoá); thay đổi đi qua migration mới | hợp đồng dữ liệu | tài liệu gốc `db-schema` (trong repo, không nộp) |
| K7 | Sequence sinh mã mới: gọi hàm sinh mã thật một lần sau khi seed | `lot_code_seq`, `trade_code_seq` từng lệch với seed (`0021`, `0032`) | `0021`, `0032` |
| K8 | Comment trong migration có thể lỗi thời ngay tại chỗ — kiểm bằng truy vấn/đếm, không bằng đọc comment (vd `P0007` ghi "dự phòng" nhưng chính `0012` chiếm nó) | | `0012` |

## Bảng mã lỗi đã triển khai

`USING ERRCODE` tự định nghĩa, đọc ổn định qua PostgREST. **ĐÃ TRIỂN KHAI** — `P0001`–`P0009` đã dùng hết (`P0007` ở `0012`, `P0008` ở `0034/0035`; tài liệu gốc `db-schema` §4, không nộp, ghi hai mã này là "dự phòng" — đã lỗi thời).

| Mã | Ý nghĩa | `details` (sau `JSON.parse`) | Nơi phát sinh |
|---|---|---|---|
| `P0001` | Không tìm thấy bản ghi (lô/giao dịch/ngày nghiệp vụ/tài khoản) | `{id}` | các RPC (`0007:60,133`; `0008:19,58`) |
| `P0002` | Ngày nghiệp vụ đã chốt sổ | `{business_date, closed_at}` (chỉ `{business_date}` khi `close_business_day` chốt lần 2) | `_confirm_trade`, `record_delivery`, `close_business_day` |
| `P0003` | Tư cách tham gia của người mua hết hiệu lực | `{expired_on, basis_type, basis_label, contact_office}` | `_assert_buyer_qualified` (`0007:33`) |
| `P0004` | Lô không đủ số lượng còn lại | `{remaining, requested, sold_to[]}` | `_confirm_trade` (`0007:78`) |
| `P0005` | Sai chức danh | `{required_role, actual_role}` | mọi RPC, kể cả `log_blocked_write_attempt` (`0040:33`) |
| `P0006` | Giao vượt số còn lại của giao dịch | `{delivered, remaining, requested}` | `record_delivery` (`0007:151`) |
| `P0007` | Quản trị viên tự đổi quyền của chính mình | `{user_id}` | `change_user_role` (`0012:22`) |
| `P0008` | Giao dịch `auction` thiếu thời điểm phiên hoặc người xác nhận lúc commit | `{id}` | trigger `check_auction_fields_complete` (`0035:33`) |
| `P0009` | Chặn ghi trực tiếp vào ngày đã chốt | `{table, business_date}` | trigger `guard_closed_day` (`0010:34`) |

Mã chuẩn Postgres gặp trong tài liệu: `42501` (anon bị từ chối EXECUTE), `23505` (UNIQUE), `23514` (CHECK).

## Mã lỗi đề xuất (chưa cài)

**ĐỀ XUẤT.** Mỗi nhóm spec được cấp một dải riêng để không đụng nhau. Sau lượt nộp lại 2026-10-06: **68 mã distinct** (9 đã cài + 59 đề xuất: `P0010`–`P0027`, `P0030`–`P0038`, `P0040`–`P0071`), **không mã nào mang hai nghĩa**. Mã từ `P0010` trở lên chưa tồn tại trong database.

Đợt nộp lại phát hiện và sửa **3 chỗ trùng nghĩa** mà lượt 1 đã bỏ sót: `P0005` (đã cài, "sai chức danh") bị tái dùng cho "mã đã tồn tại" (SCR006, SCR037) và "trạng thái đã đổi từ lúc mở phiếu" (SCR036); `P0017` bị dùng đôi ("không có quy chế hiệu lực" và "không còn phiên bản trước để quay lui" ở SCR037). Cấp lại: "mã đã tồn tại" → `P0069`, "trạng thái đã đổi" → `P0070`, "không còn phiên bản trước" → `P0071`; `P0005` và `P0017` giữ nghĩa gốc.

| Dải | Nhóm | Dùng cho |
|---|---|---|
| `P0010`–`P0011` | architecture §2.3 | Vượt điều kiện `条件付き` · đang `再審査` |
| `P0012`–`P0019` | SCR005, SCR006, SCR036, SCR037 | Sai máy trạng thái (`P0012`) · maker=checker/thiếu `can_approve` (`P0013`) · thiếu điều kiện `条件付き` (`P0014`) · vượt 4 lần/năm · `発効日` sai · không có quy tắc tại `as_of` (`P0017`) · căn cứ không khớp loại đối tác |
| `P0020`–`P0027` | F07 — điều chỉnh sau chốt | Ngày đích chưa chốt (`P0020`) · trùng phiếu pending (`P0021`) · delta = 0 (`P0022`) · maker=checker (`P0023`) · số gốc đã đổi, khoá lạc quan (`P0024`) · phiếu không còn pending (`P0025`) · không có ngày mở (`P0026`) · thiếu `can_approve` (`P0027`) |
| `P0030`–`P0038` | F08 — tiền khuyến khích | Trần 4 lần/năm · `発効日` không hợp lệ · quay lui không còn bản trước · không có `料率` tại `as_of` · kỳ đã chốt |
| `P0040`–`P0049` | F09 — báo cáo và xuất | Mã báo cáo ngoài catalog · tham số sai · **thiếu `as_of` (chống `now()`)** · trùng lô `full` đã gửi · vượt 3 lần thử |
| `P0050`–`P0059` | F10 + F06 | Ngưỡng ngoài khoảng · bật loại sự kiện không người nhận · chuyển trạng thái tranh chấp sai · ngày là `休市日` · lịch do nguồn ngoài quản |
| `P0060`–`P0068` | Chi tiết và giao hàng | Lô sai trạng thái để ghi đánh giá · giảm `quantity` dưới số đã bán · huỷ giao dịch đã có lần giao · chốt hoàn tất chưa đủ điều kiện · ghi thẳng ngoại lệ lệch số lượng · duyệt phiếu ngoại lệ trùng người lập · trùng `client_generated_id` · đồng bộ về ngày đã chốt · tra cứu nhật ký quá rộng |
| `P0069`–`P0071` | mã cấp lại cho các màn mới (xem ghi chú dưới Bảng mã lỗi đề xuất) | Mã đã tồn tại (SCR006, SCR037) · trạng thái đã đổi từ lúc mở phiếu (SCR036) · không còn phiên bản trước để quay lui (SCR037) |

### Bảng toàn cục mã lỗi → nghĩa (đối chiếu với từng spec màn)

Nguồn nghĩa là **Error Catalogue của spec màn** (spec màn thắng khi lệch). Một mã có nhiều nhánh (`P0054`, `P0063`) vẫn là một nghĩa: cùng một thao tác bị từ chối vì điều kiện tiên quyết, nhánh phân biệt qua `details`. Mã dùng ở nhiều màn cùng nghĩa ghi đủ các màn.

| Mã | Nghĩa | Màn dùng |
|---|---|---|
| `P0001`–`P0009` | xem bảng đã triển khai ở trên (`P0005` = sai chức danh, ở mọi RPC ghi; `P0009` = ghi vào ngày đã chốt, cũng là lưới an toàn của SCR035) | nhiều màn |
| `P0010` | vượt điều kiện `条件付き` | SCR010, SCR011 (hệ quả từ SCR036) |
| `P0011` | đang `再審査` | SCR010, SCR011 (hệ quả từ SCR036) |
| `P0012` | bước chuyển không thuộc máy trạng thái của `basis_type` | SCR036 |
| `P0013` | maker = checker, hoặc người duyệt thiếu `can_approve` | SCR036, SCR037 |
| `P0014` | `条件付き` thiếu nội dung hoặc hạn điều kiện | SCR036 |
| `P0015` | đã đủ 4 bản `approved` trong cùng năm dương lịch | SCR037 |
| `P0016` | `effective_from` không ở tương lai, hoặc trùng bản `approved` khác | SCR037 |
| `P0017` | không có quy chế/quy tắc hiệu lực tại ngày tra (`resolve_market_rule`) | SCR005, SCR006 (biến thể D), SCR036, SCR037 |
| `P0018` | `basis_type` không khớp `party_type` | SCR006 |
| `P0019` | thiếu `market_id` với căn cứ `approval`/`market_registration` | SCR006 |
| `P0020` | ngày của bản ghi đích chưa chốt | SCR022 |
| `P0021` | đã có phiếu `pending` trên cùng bản ghi | SCR022 |
| `P0022` | delta = 0 | SCR022 |
| `P0023` | người duyệt trùng người lập phiếu điều chỉnh | SCR023, SCR024 |
| `P0024` | số gốc đã đổi so với `snapshot_before` | SCR024 |
| `P0025` | phiếu không còn `pending` lúc duyệt | SCR024 |
| `P0026` | không có ngày nghiệp vụ mở để ghi điều chỉnh | SCR024 |
| `P0027` | người duyệt thiếu `can_approve` (phiếu điều chỉnh) | SCR023, SCR024 |
| `P0030` | đã đủ 4 bản `approved` trong năm (料率) | SCR026, SCR027 |
| `P0031` | `発効日` không ở tương lai, hoặc trùng bản `approved` khác (料率) | SCR026, SCR027 |
| `P0032` | thiếu thẩm quyền duyệt/quay lui 料率 (không `can_approve`, hoặc người duyệt = người lập) | SCR026, SCR027 |
| `P0033` | quay lui 料率 mà không còn bản trước | SCR026 |
| `P0034` | không có 料率 `approved` hiệu lực tại ngày nghiệp vụ | SCR025 |
| `P0035` | tính lại kỳ đã `closed` | SCR025 |
| `P0036` | bản 料率 không còn `pending` lúc duyệt | SCR027 |
| `P0037` | `grant_rate` ngoài khoảng khi server kiểm lại | SCR027 |
| `P0038` | kỳ còn ngày `open` (cảnh báo, vẫn cho tính) | SCR025 |
| `P0040` | `report_code` ngoài 12 mã seed | SCR028, SCR029 |
| `P0041` | tham số báo cáo sai, lạ hoặc ngày không tồn tại | SCR029 |
| `P0042` | báo cáo chỉ-sau-chốt trên ngày chưa chốt | SCR029 |
| `P0043` | thiếu `as_of` ở báo cáo chỉ-sau-chốt (chống `now()`) | SCR029 |
| `P0044` | chức danh không nằm trong `allowed_roles` của báo cáo | SCR028, SCR029 |
| `P0045` | tạo lô kế toán khi ngày chưa chốt | SCR030 |
| `P0046` | đã có lô `full` ở trạng thái `sent` cho ngày | SCR030 |
| `P0047` | "Chạy lại" khi batch không ở `failed` | SCR030 |
| `P0048` | batch đã thử `attempt ≥ 3` | SCR030 |
| `P0049` | xuất CSV quá 50.000 dòng (chuyển chạy nền, không chặn) | SCR029 |
| `P0050` | đánh dấu đã đọc thông báo của người khác hoặc không tồn tại | SCR031 |
| `P0051` | `threshold_value` ngoài khoảng cho phép | SCR032 |
| `P0052` | bật loại sự kiện mà không có người nhận | SCR032 |
| `P0053` | cấu hình đã bị người khác sửa (khoá lạc quan) | SCR032 |
| `P0054` | không thử lại được cảnh báo vận hành (đã xử lý xong, `hard_fail`, hoặc đang có lần thử chạy dở) | SCR033 |
| `P0055` | chuyển trạng thái tranh chấp ngoài 6 chuyển hợp lệ | SCR020 |
| `P0056` | `resolved` thiếu `resolution_type`/`resolution_note` | SCR020 |
| `P0057` | ngày là `休市日`, không dùng làm `業務日` | SCR020, SCR021 |
| `P0058` | đặt nghỉ cho ngày đã sinh `業務日` mà `closed` hoặc đã có dữ liệu | SCR021 |
| `P0059` | ghi lịch nghỉ khi `calendar_mode = external_sync` | SCR021 |
| `P0060` | lô sai trạng thái để ghi đánh giá (không còn `受付`/`下見済`) | SCR009 |
| `P0061` | giảm `quantity` dưới số đã bán | SCR009 |
| `P0062` | huỷ giao dịch đã có lần giao (kiểm `exists(select 1 from deliveries where trade_id = …)`, không dùng `v_trade_delivery_status.delivery_count` vì view chỉ đếm `completed`/`exception`) | SCR013 |
| `P0063` | chưa đủ điều kiện chốt hoàn tất giao hàng (số lượng chưa khớp `精算`; hoặc còn lần giao `pending`/`in_transit`) | SCR016 |
| `P0064` | ghi thẳng ngoại lệ lệch số lượng ở `record_delivery`, chưa có phiếu duyệt (ghi từ SCR014, dẫn sang SCR017) | SCR017 |
| `P0065` | người duyệt phiếu ngoại lệ giao hàng trùng người lập, hoặc không có cờ `can_approve` (duyệt bởi `reconcile` + `can_approve`; cùng cách `P0013` ở SCR036). Sai role vẫn là `P0005` | SCR017 |
| `P0066` | cùng `client_generated_id` nhưng nội dung khác | SCR035 |
| `P0067` | đồng bộ về ngày nghiệp vụ đã chốt | SCR035 |
| `P0068` | khoảng tra nhật ký `> 31` ngày mà không có chiều lọc hẹp | SCR034 |
| `P0069` | mã đã tồn tại (mã người tham gia, mã chợ) | SCR006, SCR037 |
| `P0070` | trạng thái đã đổi từ lúc mở phiếu | SCR036 |
| `P0071` | không còn phiên bản trước để quay lui | SCR037 |

Ghi chú mã: `P0069`–`P0071` được cấp thêm cho các màn SCR006/SCR036/SCR037 (thiết kế đề xuất, chưa cài); các mã còn lại giữ như ở bảng.

Cách kiểm: script quét mọi dòng `| \`P00xx\` |` trong Error Catalogue của 37 spec màn, gom theo mã và so nghĩa; không mã nào ra hai nghĩa khác nhau. Các spec feature, flow, wireframe và bảng này được đối chiếu lại theo kết quả đó.

## Phủ yêu cầu LAB-1

39 yêu cầu truy vết của RFP (24 × P0, 15 × P1; `product-a-rfp-business-summary.md` §13) đối chiếu với prototype. "Một phần" = có xương nhưng thiếu thịt (xem MOCK-1…9 ở tài liệu gốc trong repo, không nộp: `lab3-scope-lock`).

| Yêu cầu | Prio | Prototype | Tính năng |
|---|:-:|---|---|
| FR-IAM-01 xác thực + gán role | P0 | Có | [F001](../features/F001_AdminAndPermissions/technical-spec.md) |
| FR-IAM-02 cấp/dừng/mở lại quyền + lịch sử | P0 | Một phần (đổi role + lịch sử có; đổi `status` chưa có RPC riêng) | F001 |
| FR-PARTY-01 profile 参加者 theo 4 loại | P0 | Không (có bảng, không màn) | [F002](../features/F002_ParticipantsAndQualification/technical-spec.md) |
| FR-PARTY-02 kiểm hiệu lực khi chốt | P0 | Một phần (một luật chung, D-01) | F002 |
| FR-PARTY-03 cảnh báo gần hết hạn | P1 | Không | F002 |
| FR-LOT-01 mã lô + thuộc tính + chứng từ | P0 | Có | [F003](../features/F003_LotsAndInspection/technical-spec.md) |
| FR-LOT-02 kết quả đánh giá + người + thời điểm | P0 | Không (`下見済` chưa có đường ghi) | F003 |
| FR-LOT-03 kiểm số lượng khả dụng | P0 | Có | F003 |
| FR-LOT-04 điều chỉnh lô có kiểm soát | P1 | Không | F003 |
| FR-AITAI-01 tạo giao dịch trực tiếp | P0 | Có | [F004](../features/F004_TradeRecording/technical-spec.md) |
| FR-AITAI-02 từ chối chốt + lý do | P0 | Có | F004 |
| FR-AITAI-03 huỷ giao dịch chưa lock | P1 | Không (`cancelled` có trong enum) | F004 |
| FR-SERI-01…02 ghi kết quả せり nhập tay | P0 | Có | F004 |
| FR-SERI-03 tra lại + lịch sử sửa | P1 | Không | F004 |
| FR-CORR-01…02 phiếu + reverse/delta | P0 | Không (MOCK-6) | [F007](../features/F007_PostCloseCorrection/technical-spec.md) |
| FR-CORR-03 chặn sửa ngày đã lock + ghi log | P0 | Một phần (chặn đủ; ghi log từ đường SC-14) | F007 / [F006](../features/F006_ReconciliationAndClosing/technical-spec.md) |
| FR-SETTLE-01…02 bảng đối chiếu + lock ngày | P0 | Có | F006 |
| FR-SETTLE-03 tranh chấp + SLA | P1 | Không | F006 |
| FR-INC-01…03 tiền khuyến khích | P0 | Không (MOCK-4) | [F008](../features/F008_IncentiveAndRateVersion/technical-spec.md) |
| FR-RULE-01 phiên bản quy chế | P0 | Không | F008 / [F012](../features/F012_MarketManagement/technical-spec.md) |
| FR-DEL-01 trạng thái từng lần giao | P0 | Một phần | [F005](../features/F005_DeliveryTracking/technical-spec.md) |
| FR-DEL-02 chốt hoàn tất khớp 精算 | P0 | Không (chưa định nghĩa) | F005 |
| FR-DEL-03 lý do ngoại lệ | P1 | Một phần (`over` chưa ghi được) | F005 |
| FR-DEL-04 liên kết giao hàng ↔ 取引/精算 | P1 | Một phần (liên kết trade; chưa có 精算) | F005 |
| FR-DEL-05 nhiều lần giao + 累積/残数量 | P1 | Có | F005 |
| FR-NOTIFY-01…03 thông báo | P1 | Không (MOCK-7) | [F010](../features/F010_Notifications/technical-spec.md) |
| FR-RPT-01…03 báo cáo + xuất | P0/P1 | Không | [F009](../features/F009_ReportsAndExport/technical-spec.md) |
| FR-AUDIT-01 logical audit | P0 | Một phần (đổi quyền, chốt sổ, thử ghi bị chặn; thiếu `reason`) | [F011](../features/F011_AuditLog/technical-spec.md) |
| FR-AUDIT-02…03 tra cứu; hiển thị version | P1 | Không (MOCK-8) | F011 |

Yêu cầu phi chức năng (RFP 09): NFR-AVL-01 99,5%/tháng 02:00–10:00 JST · NFR-AVL-02 mất mạng ngắn · NFR-AVL-03 RTO ≤ 4h/RPO ≤ 15 phút · NFR-PERF-01 p95 ≤ 2s · NFR-PERF-02 load FIG-LOAD-01 · NFR-SEC-01 MFA phân tầng (đã chốt PREMISE C-04; prototype chưa có, MOCK-1) · NFR-SEC-03 session timeout/khoá · NFR-SEC-04 TLS, secret tách khỏi code · DR-RET-01 lưu 7 năm/3 năm · NFR-OPS-01 monitor · NFR-OPS-02 đổi quy tắc không cần deploy (ĐỀ XUẤT, ADR-007) · NFR-USE-01 keyboard (có) · NFR-COMP-01 trình duyệt + tablet. Trong prototype chỉ NFR-USE-01 và một phần NFR-SEC-04 được hiện thực/kiểm chứng; các NFR còn lại **chưa đo**.

## Năm lỗi tầng dữ liệu đã phát hiện

Lỗi **thật** trong code LAB-3; B-01…B-03 và B-05 đã xác minh trên database đang chạy, B-04 mới đọc code (chưa tái hiện). Phải xử lý trước LAB-5 vì spec của các màn giả định hành vi đúng. **Code không sửa trong đợt nộp này** (đề không yêu cầu nộp code). Chi tiết và bằng chứng: [entities.md](../generated/entities.md#năm-lỗi-tầng-dữ-liệu-phát-hiện-khi-viết-spec).

| # | Lỗi | Ảnh hưởng |
|---|---|---|
| B-01 | `v_trade_delivery_status` cộng cả `returned`/`partial_cancel` vào "đã giao" | trả hàng làm "đã giao" **tăng**; chạm SC-14, 15, 16, 18 |
| B-02 | `record_delivery` lật `lots.status = 引渡し完了` khi **một** giao dịch giao đủ | trạng thái lô sai cho giao dịch còn lại; vô hiệu hoá phép kiểm tư cách của `F05-04` |
| B-03 | `over` không ghi được vì `P0006` | enum có giá trị không bao giờ dùng; thực ra thiếu SC-17 |
| B-04 | *(đọc code, chưa tái hiện)* `record_delivery` tính `v_delivered` chỉ từ `completed`/`exception` (`0007:146-147`) nhưng lật `lots.status = 引渡し完了` khi `v_remaining − p_quantity = 0` bất kể `p_status` (`0007:159-161`); form cho chọn `pending`/`in_transit` (`lib/domain/deliveries.ts:11`) | ghi một lần giao "Chờ giao" bằng đúng số còn lại → lô thành đã giao xong trong khi chưa giao gì. Chạm SC-14 |
| B-05 | **ĐÃ XÁC NHẬN trên DB thật 2026-10-06.** `v_daily_reconciliation` `LEFT JOIN deliveries` rồi `sum(tr.amount)` (`0006:48-62`) → tiền giao dịch nhân theo số lần giao | ngày 2026-09-23 view = 53250.00, tổng thật `sum(trades.amount where confirmed)` = 38250.00, lệch +15000 vì `TRD-20260923-0001` có 2 lần giao; ngày 2026-09-24 khớp 69550.00. SC-18 hiển thị sai tổng tiền; số này còn dùng cho chốt sổ và xuất kế toán. **Chưa sửa; đã có bản vá nháp `0042_fix_daily_reconciliation_amount.sql`, chưa áp.** |

Checklist LAB-3 không bắt được B-05 vì đối chiếu số với chính view. Câu SQL kiểm: so `v_daily_reconciliation.total_amount` với `sum(amount)` từ `trades` cùng ngày, `status = 'confirmed'` (V-F006-04 ở [F006](../features/F006_ReconciliationAndClosing/technical-spec.md)).

Hướng sửa đề xuất, chưa làm: B-05 tính `sum` trên `trades` trước rồi mới join đếm `deliveries` (hoặc subquery tương quan); B-04 chỉ lật trạng thái khi `p_status in ('completed','exception')`; B-02 ghi hoàn tất ở cấp giao dịch (`trades.delivery_completed_at`) và chỉ lật lô sang `引渡し完了` khi mọi giao dịch `confirmed` của lô đã hoàn tất (PREMISE G3). B-05 đã có bản vá nháp `0042_fix_daily_reconciliation_amount.sql`, chưa áp.

## Điểm chưa chốt trước LAB-5

Gom từ mục 10 của 37 screen spec (xếp theo mức chặn).

### Cần hỏi khách

| # | Câu hỏi | Chặn gì |
|---|---|---|
| 1 | **`条件付き` cụ thể hạn chế cái gì?** | `P0010` không có gì để kiểm. Ưu tiên cao nhất |
| 2 | **"Khớp quy tắc `精算`" (`BR-DEL-03`) nghĩa là gì?** | `P0063` và nửa hộp thoại SC-16. RFP xếp vào 4 điểm chặn estimate |
| 3 | **Phạm vi degraded operation** (câu 1 gửi khách) | Toàn bộ SC-35, và có cần thêm màn điều hoà mâu thuẫn không |
| 4 | **Số cấp duyệt** cho điều chỉnh sau chốt (câu 3 gửi khách) | **Đã chốt một cấp (PREMISE C-03b)**; nếu khách đòi phân cấp → một luồng; phân cấp theo tiền → `F07-02` nở 3–5 luồng + thêm màn định nghĩa ngưỡng |
| 5 | **Lịch nghỉ chợ** nhập tay hay đồng bộ nguồn ngoài (câu 16) | **Đã chốt nhập tay (PREMISE C-08)**; giữ cờ `calendar_mode` để đổi nếu khách chọn khác; khách chưa xác nhận giả định. Hướng đồng bộ phát sinh hạng mục chưa có trong ước lượng |
| 6 | **Một chợ hay nhiều chợ?** | `SC-N2`, `SC-21`, và câu "`料率` thuộc chợ hay quốc gia" |
| 7 | Ai có quyền duyệt `再審査` | Maker-checker của `SC-N1` (hướng đã chọn ở PREMISE G8) |
| 8 | Hạn hiệu lực theo căn cứ | **Đã chốt (PREMISE G9; khách chưa xác nhận):** `valid_to` nullable (NULL = không thời hạn) với `市場登録` (`registration`) và `卸売場登録` (`market_registration`); `許可`, `承認` bắt buộc có hạn. Prototype `valid_to NOT NULL` → nới nullable là thay đổi ràng buộc đề xuất, migration ở LAB-5 |
| 9 | **Test vector Appendix C** cho công thức tiền khuyến khích | `FR-INC-01` và `US-13` đòi khớp test vector, LAB-1 chỉ mô tả 3 ca biên, không có bộ số |
| 10 | SLA tranh chấp — `RPT-09` nhắc nhưng không cho số ngày | `due_date` hiện là ô nhập tay, không so được giữa các vụ |

### Cần quyết nội bộ

| # | Vấn đề | Ghi chú |
|---|---|---|
| 11 | **Chức danh:** thêm `party_admin`, `rule_admin`, `ops_admin`, `market_manager`, `internal_audit` hay dùng cờ `can_approve`? | **Đã chốt (PREMISE C-09):** 9 role key + cờ `can_approve`. Prototype có 4, LAB-1 có 7; giữ 4 thì maker-checker không hiện thực hoá được |
| 12 | **Sửa `B-01`, `B-02`** | Đổi view là thay đổi phá tương thích, chạm 4 màn |
| 13 | **RLS `app_users` chỉ cho tự xem mình + `sys_admin`** | Người duyệt thấy "Người 起票 = —" → duyệt mà không biết phiếu của ai. Hỏng nghiệp vụ, không chỉ hỏng hiển thị |
| 14 | **Seed chỉ có 1 tài khoản `reconcile`** | Maker-checker không demo được. Cần tài khoản thứ hai |
| 15 | **`audit_log` thiếu cột `reason`** và RLS chỉ mở cho `sys_admin` | `FR-AUDIT-01` đòi đủ 5 trường. Không vá thì `SC-34` trả rỗng |
| 16 | **`correction_entries` phải cho số âm ngay từ migration đầu** | `trades.amount` là cột `GENERATED`; sao khuôn đó thì không ghi được số âm cho `reverse` |
| 17 | Token màu cho badge trạng thái | `条件付き` và `再審査` mà cùng màu thì mất nghĩa D-01 |
| 18 | Thứ tự prefix trong `ROUTE_POLICY` | `/notifications/settings` phải đặt **trước** `/notifications`; `/deliveries/new` phải khớp **chính xác** trước prefix `/deliveries` — nếu không lớp chặn bị nuốt |

### Phụ thuộc thứ tự triển khai

| Màn | Cần có trước |
|---|---|
| `SC-30` batch kế toán | **`SC-22`/F07** — đường khắc phục duy nhất sau khi gửi lô dẫn tới màn chưa tồn tại |
| `SC-17` ngoại lệ giao hàng | RPC duyệt phải bỏ qua trần `P0006` tường minh, nếu không `over` vẫn không ghi được |
| `SC-34` audit | Migration thêm `reason` + nới RLS |

### Câu hỏi còn mở từ các ADR

1. **ADR-005** — "điều kiện phải đạt" trong `US-11` nên hiểu là chặn hay cảnh báo? Hiện chọn cảnh báo (đã chốt, PREMISE C-07); khách chưa xác nhận.
2. **ADR-006** — hai căn cứ còn lại (`市場登録`, đăng ký tại điểm giao dịch) có luồng vi phạm riêng không, hay hoàn toàn theo quy chế chợ? Tài liệu khách để trống.
3. **ADR-007** — hệ thống thực tế phục vụ một chợ hay nhiều chợ? Nếu một chợ thì ADR-007 hạ từ bắt buộc xuống nên có.
4. **ADR-001** — ngưỡng tải nào thì round-trip mỗi request thành vấn đề? Chưa đo, chỉ ước lượng từ 60 người dùng đồng thời.
