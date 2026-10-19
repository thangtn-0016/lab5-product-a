# Business Context — F003_LotsAndInspection

## Why It Matters

Một lô cá là **vật thể duy nhất đã lên bờ** — không phải hàng tồn kho nhập thêm được. Hôm nay cùng một lô xuất hiện trên ba loại phiếu giấy và có người phải nhập lại; khách đặt mục tiêu giảm số lần nhập lại từ 2,4 xuống tối đa 1,2 lần mỗi giao dịch. Tính năng này đưa lô vào hệ thống ngay khi hàng cập chợ, theo dõi nó qua năm trạng thái, và bảo đảm số lượng còn lại **không bao giờ âm** — nếu bán vượt lô thì đó là tranh chấp giữa hai người mua đang đứng ngay tại sàn, không sửa được bằng một cuộc gọi cho nhà cung cấp.

Một điểm khách hàng cố ý giữ trong tay con người: **đánh giá chất lượng lô** (độ tươi, loài, hạng). Hệ thống chỉ ghi lại kết quả do người có kinh nghiệm nhập — kèm tên người ghi và thời điểm — chứ không gợi ý, không tự phân loại, không đọc ảnh. Tự động hoá chỗ này là điều kiện bị loại.

Tính năng này mở đầu [chuỗi lô → giao dịch → giao hàng → đối chiếu](../../flows/FL-01-lot-trade-delivery-reconciliation.md).

## Who Uses It

- **Nhân viên tiếp nhận** (`receiving`) — nhập lô ngay tại khu tiếp nhận từ 02:00 sáng, đeo găng tay, trên máy dùng chung theo ca; chuyển lô sang "đã mở bán". Cần làm xong **không dùng chuột**.
- **Người đánh giá** — LAB-1 tách "tiếp nhận" và "đánh giá"; hệ thống đang chạy gộp cả hai vào chức danh tiếp nhận; đã chốt giữ gộp (PREMISE C-10), khách chưa xác nhận.
- **Nhân viên giao dịch** (`trading`) — xem danh sách lô để chọn lô trước khi chốt giao dịch; chỉ xem.
- **Nhân viên đối chiếu** (`reconcile`) — cần xem chi tiết lô để giải thích số liệu (đặc tả; quyền đọc lô chưa được mở).

## What They Do

1. Nhân viên tiếp nhận nhập mã chủ hàng, mã mặt hàng, số lượng, ngày nghiệp vụ, số chứng từ tiếp nhận và ghi chú — hệ thống cấp **mã lô duy nhất** và lưu lô ở trạng thái "đã tiếp nhận".
2. Nhân viên lưu rồi nhập tiếp lô kế mà không rời bàn phím — form xoá sạch để người ca sau không thấy dữ liệu của ca trước.
3. Người đánh giá ghi kết quả nhìn hàng vào lô — hệ thống lưu kèm tên người đánh giá và thời điểm; lô chuyển sang "đã xem hàng" (đặc tả).
4. Nhân viên tiếp nhận bấm "Mở bán" cho lô — lô sang trạng thái "đã mở bán" và hiện trong danh sách của nhân viên giao dịch.
5. Khi giao dịch được chốt, lô trừ số lượng; khi bán hết, lô sang "đã bán"; khi giao đủ, sang "đã giao xong". Số **còn lại** luôn được tính từ các giao dịch đã chốt, không có ô nào để ai đó sửa tay.
6. Cần sửa thuộc tính lô (ví dụ số lượng) — nhân viên nêu lý do, hệ thống lưu số cũ, số mới và người sửa; từ chối nếu số lượng mới thấp hơn phần đã bán hoặc ngày đã chốt sổ (đặc tả).
7. Khi mạng khu tiếp nhận mất (3–5 lần mỗi tuần giờ cao điểm), nhân viên vẫn ghi được lô vào máy; bản ghi tự gửi khi có mạng, không tạo bản trùng (đặc tả).

## Unresolved Questions

- **Số lượng ban đầu của lô có được sửa không?** Tài liệu LAB-3 coi là "bất biến sau khi tạo", nhưng khách yêu cầu điều chỉnh thuộc tính lô có kiểm soát. Hai điều này mâu thuẫn trừ khi "bất biến" hiểu là "không bị trừ tự động". Chặn thiết kế.
- **Người đánh giá có là một chức danh riêng không?** Nếu có thì phát sinh chức danh thứ năm và ma trận quyền phải làm lại.
- **Kết quả đánh giá có cấu trúc (hạng, độ tươi, cỡ) hay chỉ văn bản tự do?** Nghiêng về văn bản tự do vì không cho hệ thống tự phân loại; khách cần xác nhận.
- **Một lô được đánh giá nhiều lần thì lần nào là chính thức?**
- **Phạm vi làm việc khi mất mạng**: chỉ nhập lô và ghi lần giao, hay rộng hơn? Nếu rộng hơn, chi phí tăng khoảng 80–120 ngày công (ước lượng LAB-1).
- **Chứng từ tiếp nhận** là một số tham chiếu hay một tệp đính kèm? Hệ thống đang chạy chỉ lưu số tham chiếu và ghi chú.
