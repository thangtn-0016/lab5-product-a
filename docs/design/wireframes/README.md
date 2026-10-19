# Wireframe LAB-4 — bốn màn hình mới

Wireframe HTML tĩnh cho bốn màn hình **chưa từng được xây** của LAB-4. Đây là **tài liệu thiết kế**, không phải code sản phẩm: không framework, không build, không gọi dữ liệu. Mọi số liệu trên trang là dữ liệu mẫu nhúng cứng.

Nguồn quyền lực là các screen spec trong `docs/screens/` (SCR005, SCR006, SCR036, SCR037). Wireframe bám theo spec; chỗ nào spec chưa đủ thì ghi rõ ở mục "Chỗ tự quyết" bên dưới.

## Cách mở

Mở thẳng bằng trình duyệt — không cần server, không cần Supabase:

```
open docs/design/wireframes/sc-05-danh-sach-nguoi-tham-gia.html
```

Hoặc kéo file thả vào cửa sổ trình duyệt. Bốn file liên kết chéo với nhau qua đường dẫn tương đối nên điều hướng giữa các màn chạy được ngay trên `file://`.

## Chỉ mục

| File | Màn | Thể hiện điều gì |
|---|---|---|
| `sc-05-danh-sach-nguoi-tham-gia.html` | `SC-05` Danh sách người tham gia | Bảng 10 cột + 7 ô lọc (mã/tên, loại đối tác, căn cứ, trạng thái hiệu lực, sắp hết hạn, ngưỡng cảnh báo, chợ). Cột trạng thái dùng **badge năm màu phân biệt**: `有効` xanh · `条件付き` hổ phách · `再審査` đỏ · `失効` xám nhạt · `取消` xám đậm. Kèm banner `V-02`, bộ đếm số dòng đang `条件付き`/`再審査`, chú giải màu, và mô tả các trạng thái màn hình còn lại. |
| `sc-06-ho-so-nguoi-tham-gia.html` | `SC-06` Hồ sơ người tham gia | **Bốn biến thể form**. Khối chung đặt một lần ở trên; bốn khối riêng nằm trong bốn thẻ (tab thuần CSS, không JS): A `市場登録` (số đăng ký, bên cấp là 開設者) · B `許可` (số giấy phép, **cơ quan cấp phép ngoài chợ**, ngày cấp, hạn, khối `条件付き` chỉ đọc) · C `承認` (số chấp thuận, **người duyệt là chính chợ**, hạn mức tham gia, khối `再審査` chỉ đọc) · D `卸売場登録` (mã điểm, tên điểm, phiên bản quy chế chợ đang áp). Đóng bằng bảng đối chiếu 8 dòng "vì sao bốn biến thể không dùng chung một form". |
| `sc-n1-xet-duyet-tu-cach.html` | `SC-N1` Xét duyệt / tạm dừng tư cách | **Hai sơ đồ máy trạng thái vẽ cạnh nhau** bằng div + ký tự mũi tên: `許可` (`有効 → 条件付き → 有効 \| 取消`, **vẫn giao dịch được, có trần**) và `承認` (`有効 → 再審査 → 有効 \| 取消`, **chặn giao dịch tới khi duyệt lại**), mỗi sơ đồ có một khối "khác biệt cốt lõi". Kèm danh sách phiếu, phiếu maker-checker (khối lập / khối duyệt với `V-05` ẩn nút khi người duyệt trùng người đề nghị), và bảng dấu vết hai người của các phiếu đã xử lý. |
| `sc-n2-quan-ly-cho.html` | `SC-N2` Quản lý chợ & quy tắc vận hành | Khối chợ `markets`, danh mục 5 `rule_key`, **bảng phiên bản quy tắc** (版, 発効日, trạng thái `draft`/`approved`/`superseded`/`rolled_back`, 起案者, 承認者, 旧版, 変更理由), thanh đếm **3/4 lần đổi trong năm** kèm cảnh báo `P0015`, khối lập nháp + khối duyệt đối chiếu v4↔v5 với `V-06`, và hộp thoại xác nhận **quay lui** nêu rõ bản nào sống lại, từ ngày nào, bản nào bị huỷ hiệu lực. |

## Quy ước chung của bốn file

- Mỗi file có **khối chú thích nền vàng ở đầu trang**: nói rõ đây là wireframe màn chưa xây, trỏ về file spec tương ứng, và nói rõ dữ liệu là mẫu.
- Khối nền xanh nhạt viền trái xanh là **ghi chú thiết kế** cho người đọc — không phải phần tử giao diện.
- Phong cách bám theo app hiện có: nền trắng, chữ `#18181b`, chữ phụ `#52525b`, viền `#e4e4e7`/`#d4d4d8`, nút chính nền đen chữ trắng, badge nền nhạt bo tròn, cỡ chữ gốc 18px, viền focus `3px solid #2563eb`.
- Thuật ngữ nghiệp vụ giữ nguyên chữ Hán: `許可` `承認` `条件付き` `再審査` `市場登録` `卸売場登録` `失効` `取消`.
- Chạm tối thiểu 44×44px, có `prefers-reduced-motion`, bố cục xếp dọc dưới 860px.

## Chỗ spec chưa đủ, wireframe phải tự quyết

Những chỗ này **không có trong spec** và chỉ là lựa chọn trình bày — không được hiểu là quyết định nghiệp vụ:

1. **Màu cụ thể của từng badge.** Spec chỉ ràng buộc `条件付き` và `再審査` không dùng chung màu. Wireframe chọn hổ phách cho `条件付き` (cảnh báo nhưng còn đi được) và đỏ cho `再審査` (chặn), xám cho `失効`/`取消`.
2. **Mã phiếu `T-0144`, mã chợ `TKY-TSUKI`, mã người tham gia `IM-0102`…** Spec không quy định định dạng mã phiếu hay dữ liệu mẫu. Toàn bộ là bịa để minh hoạ.
3. **SC-06 — vị trí của `basis_no`, `valid_from`, `valid_to`.** Spec xếp ba trường này vào khối chung (#7–#9) nhưng ràng buộc lại khác nhau theo biến thể và ánh xạ sang cột của bảng riêng. Wireframe đẩy chúng xuống khối riêng của từng biến thể để người xem thấy đúng ràng buộc (khớp G9 đã chốt: `valid_to` tuỳ chọn, trống = `無期限`, với `市場登録`/`卸売場登録`; bắt buộc với `許可`/`承認`); khối chung chỉ giữ phần thật sự dùng chung.
4. **SC-06 — tab chọn biến thể.** Ở sản phẩm thật người dùng **không chọn** biến thể (suy ra 1:1 từ `party_type`). Tab ở đây chỉ để người đọc tài liệu đối chiếu bốn biến thể cạnh nhau; wireframe đã ghi rõ điều này trong khối chú thích.
5. **SC-N2 — lược đồ `payload` của `buyer_registration`** (regex, `sites[]`, thời hạn) hiển thị dưới dạng ba ô cụ thể. Spec mục 10.8 ghi rõ lược đồ `payload` cho từng `rule_key` **chưa có bản nào từ khách** — ba ô này là minh hoạ.
6. **SC-N2 — hình thức bảng đối chiếu v4↔v5.** Spec chỉ yêu cầu "xem cạnh nhau"; cách bày thành bảng ba cột là lựa chọn của wireframe.
7. **SC-N1 — bố cục sơ đồ trạng thái.** Spec cho hai bảng bước chuyển, không cho hình. Cách vẽ (mỗi bước một dòng, chú thích dưới) là lựa chọn của wireframe.
8. **Nhãn tiếng Việt của các giá trị enum `party_type`.** Spec cho nhãn JA và tên kỹ thuật; wireframe hiển thị cả hai (`仲卸 (intermediary)`) vì chưa có từ điển nhãn VI chốt.

## Câu hỏi còn treo (lấy từ mục 10 của bốn spec, ảnh hưởng trực tiếp tới wireframe)

- **Một chợ hay nhiều chợ?** Nếu một chợ: bỏ ô lọc "Chợ" ở `SC-05`, bỏ ô chợ ở `SC-06` biến thể A/C/D, và `/markets` thu về một trang cấu hình.
- **`条件付き` cụ thể hạn chế cái gì?** Trần số lượng/ngày và cờ "chỉ 相対" đang là suy luận của spec, không phải tài liệu khách. Chưa chốt thì khối `条件付き` ở `SC-06`/`SC-N1` chỉ ghi được chữ tự do.
- **Ai có quyền duyệt?** Wireframe vẽ theo phương án cờ `can_approve`. Nếu khách tách thành chức danh riêng thì phần "ai mở được" của `SC-N1`/`SC-N2` đổi theo.
- **Trần 4 lần/năm tính theo năm dương lịch hay năm tài chính Nhật?** Thanh đếm ở `SC-N2` đang vẽ theo năm dương lịch.
- **Quay lui có tính vào trần 4 lần không?** Wireframe vẽ theo giả định **có tính**.
