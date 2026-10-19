# ADR-006 — Tách hai luồng trạng thái riêng cho `許可` và `承認`

| | |
|---|---|
| **Trạng thái** | **ĐỀ XUẤT — chưa xây.** Prototype LAB-3 đang làm khác (sai lệch D-01) |
| **Ngày ghi** | 2026-10-04, **đã sửa** sau khi viết architecture §2.3 và bộ screen spec (xem khung "Bản ghi đã được sửa") |
| **Phạm vi** | Mô hình tư cách tham gia, `_assert_buyer_qualified`, SC-06, màn xét duyệt mới |
| **Liên quan** | [architecture.md §2.3, §3.3](../system/architecture.md) · [FL-03](../flows/FL-03-participant-qualification-lifecycle.md) · [entities.md D-01](../generated/entities.md#d-01--bốn-căn-cứ-tham-gia-bị-gộp-thành-một-luật-chung) · [F002_ParticipantsAndQualification](../features/F002_ParticipantsAndQualification/technical-spec.md) · [SCR006_ParticipantProfile](../screens/SCR006_ParticipantProfile/spec.md) · [SCR036_QualificationReview](../screens/SCR036_QualificationReview/spec.md) |

## Bối cảnh

Bốn căn cứ tham gia của tài liệu khách: `市場登録` · `許可` (giấy phép) · `承認` (chấp thuận) · đăng ký tại điểm giao dịch. LAB-1 §7.2 ghi ma trận xử lý khi tạm dừng hoặc gỡ:

| | 市場登録 | **許可** | **承認** | Đăng ký tại điểm GD |
|---|---|---|---|---|
| Tạm dừng / gỡ | 承認 | **条件付き** hạn chế có điều kiện | **再審査** phải xét lại | — |

Và LAB-1 ghi thẳng: *"許可 ≠ 承認 — hai căn cứ khác bản chất, tuyệt đối không gộp thành một rule chung. Đây là điều kiện trực tiếp bị loại (RFP 12-06)."*

Prototype hiện lưu đủ bốn căn cứ nhưng dùng **một máy trạng thái ba bước chung** (`active` `expired` `revoked`) và **một phép kiểm đồng nhất** (`0007_rpc_functions.sql:18-22`). Cột `violation_action` khai khác nhau cho từng loại nhưng không dòng logic nào đọc tới.

## Lựa chọn

Tách thành hai máy trạng thái, mỗi căn cứ có đường xử lý vi phạm riêng:

```
許可 (license):   active ──vi phạm──> 条件付き ──khắc phục──> active
                                       └──không khắc phục──> revoked

承認 (approval):  active ──vi phạm──> 再審査  ──duyệt lại──> active
                                       └──từ chối────────> revoked
```

Cần: **bảng riêng và enum riêng cho mỗi căn cứ có luồng vi phạm** — `qualification_license` (許可) và `qualification_approval` (承認); thêm bảng phiếu chuyển trạng thái có maker-checker; `_assert_buyer_qualified` rẽ nhánh theo `basis_type` thay vì kiểm đồng nhất.

> **Bản ghi này đã được sửa.** Bản đầu viết "mở rộng enum `qualification_status`" — tức giữ một bảng, thêm giá trị. Khi viết `architecture.md` §2.3 và bộ screen spec `SC-06`/`SC-N1`, cách đó lộ ra là chưa đủ: LAB-1 nói rõ `許可` và `承認` là *"hai **mô hình dữ liệu** riêng, hai luồng trạng thái riêng"* — mô hình dữ liệu riêng nghĩa là bảng riêng, không phải cùng một bảng với enum rộng hơn. Hai căn cứ có **trường riêng không chia sẻ được**: `許可` cần cơ quan cấp phép ngoài chợ, `承認` cần hạn mức tham gia và người duyệt là chính chợ. Nhồi vào một bảng thì quá nửa số cột luôn `NULL`.
>
> Giữ `qualification_status` cũ cho hai căn cứ không có luồng vi phạm riêng (`市場登録`, đăng ký tại điểm giao dịch) để migration nhẹ nhất.

Chi tiết hai máy trạng thái (sơ đồ, khác biệt, mã lỗi `P0010`/`P0011`): [architecture.md §2.3](../system/architecture.md) và [FL-03](../flows/FL-03-participant-qualification-lifecycle.md).

## Phương án đã bỏ và vì sao

**Giữ một luồng chung, phân biệt bằng cột `violation_action` như hiện tại.** Bỏ vì đó chính là "gộp thành một rule chung" mà tài liệu khách xếp vào điều kiện bị loại. Nguy hiểm hơn: cột khai báo khác nhau **tạo ảo giác** quy tắc đã được mô hình hoá, trong khi nó chỉ là dữ liệu chết. Người đọc schema sẽ tưởng đã phân nhánh.

**Bảng riêng cho từng căn cứ (cả bốn).** Bỏ vì bốn bảng gần giống nhau, vi phạm DRY, và hai căn cứ còn lại (`市場登録`, đăng ký tại điểm giao dịch) không có luồng vi phạm riêng — chúng theo quy chế chợ, thuộc [ADR-007](./ADR-007-market-rules-as-versioned-data.md).

## Hệ quả

**Được:** luồng rẽ nhánh đặc thù Nhật được mô hình hoá đúng, không bị làm phẳng thành luồng tuyến tính. Đóng được điều kiện bị loại.

**Phải chịu:** `SC-06` phải có **bốn biến thể form**, không phải một form dùng chung. LAB-1 để ngỏ một màn hay bốn màn; PREMISE C-05 đã chốt một màn SCR006 với bốn biến thể form. Chi phí giao diện tăng.

**Phải chịu:** thêm một màn hình mới — xét duyệt và tạm dừng tư cách — với luồng duyệt người-khác-người-đề-nghị. Không có trong 35 màn của LAB-1.

**Phải chịu:** di trú dữ liệu từ một bảng sang ba. Dòng đang ở `expired`/`revoked` phải tách về đúng bảng theo `basis_type`, và `_assert_buyer_qualified` phải đọc ba nguồn thay vì một — truy vấn phức tạp hơn hẳn.

**Phải chịu:** cột `valid_to` hiện là `NOT NULL` (`0003:47`), nhưng LAB-1 F02-01 (đăng ký chợ) và F02-04 (đăng ký tại điểm giao dịch) không ghi ngày hết hạn. Người làm bài đã chốt (PREMISE G9; khách chưa xác nhận): `valid_to` nullable với hai căn cứ đó, `許可`/`承認` vẫn bắt buộc. Nới nullable là thay đổi phá tương thích, migration ở LAB-5.

## Câu hỏi còn mở

- Hai căn cứ còn lại (`市場登録`, đăng ký tại điểm giao dịch) có luồng vi phạm riêng không, hay hoàn toàn theo quy chế chợ? Tài liệu khách để trống.
- `条件付き` cụ thể hạn chế cái gì (trần số lượng, trần giá trị, cấm せり, hay do chợ tự định)? Định hình `market_rules.payload` và `P0010`.
- `再審査` — ai có thẩm quyền duyệt lại, trong bao lâu? (xem [architecture.md §6](../system/architecture.md))
