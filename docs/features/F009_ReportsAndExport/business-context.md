# Business Context — F009_ReportsAndExport

## Why It Matters

Hiện nay báo cáo ngày và tháng do bộ phận đối chiếu ghép tay từ 5 nguồn dữ liệu, mất khoảng 90 phút mỗi ngày; tra một bản ghi cũ mất 15–30 phút. Mục tiêu khách đặt ra là báo cáo ngày còn không quá 15 phút và tra cứu không quá 2 phút. Nhóm này cũng là cửa ra duy nhất của số liệu đã chốt: 12 báo cáo cố định, xuất CSV, và gửi số liệu đối chiếu sang hệ thống kế toán của bộ phận tài chính.

Trạng thái hiện tại: chưa xây trong prototype LAB-3 (nằm trong phần được mock). Tài liệu này mô tả thiết kế theo bản đầy đủ của LAB-1.

Tính năng này nằm trong luồng "lô → giao dịch → giao hàng → đối chiếu" (flow [FL-01](../../flows/FL-01-lot-trade-delivery-reconciliation.md)) và là nơi đường điều chỉnh sau chốt (flow [FL-02](../../flows/FL-02-post-close-correction.md)) để lại hệ quả: sai lệch phát hiện sau khi đã gửi kế toán chỉ sửa được bằng phiếu điều chỉnh rồi gửi lô chênh lệch.

## Who Uses It

- **Bộ phận đối chiếu (reconcile)** — mở danh mục báo cáo sau khi sổ chốt khoảng 10:00 giờ Nhật, chạy báo cáo ngày, xuất CSV, tạo lô gửi kế toán và chạy lại khi lỗi.
- **Ban quản lý chợ (market_manager, đề xuất — chưa có trong hệ thống)** — xem các báo cáo tháng và một số báo cáo ngày; không xem hạ tầng batch.
- **Vận hành hệ thống (ops_admin, đề xuất — chưa có trong hệ thống)** — theo dõi batch trong cửa sổ một tiếng, chạy lại batch lỗi.
- **Quản trị hệ thống (sys_admin)** — chỉ xem trạng thái batch để hỗ trợ kỹ thuật, không chạy lại.
- **Bộ phận tài chính (bên ngoài hệ thống)** — nhận file số liệu theo định dạng cố định; không thao tác trong hệ thống này.

## What They Do

1. Người đối chiếu chốt sổ ngày nghiệp vụ — số liệu của ngày trở thành số chính thức.
2. Hệ thống chạy batch báo cáo ngày; batch phải xong trước 11:00 giờ Nhật, tức khoảng một tiếng sau khi chốt.
3. Người đối chiếu mở danh mục 12 báo cáo cố định (9 ngày, 3 tháng), chọn một báo cáo, nhập tham số (ngày nghiệp vụ hoặc tháng) rồi chạy — hệ thống luôn tính theo ngày nghiệp vụ và phiên bản quy chế đã hiệu lực tại thời điểm chốt, không theo giờ hiện tại.
4. Người đối chiếu xuất CSV; file mang theo thời điểm tạo, ngày nghiệp vụ, người kích hoạt và mã lô gửi.
5. Sau khi batch báo cáo xong, người đối chiếu hoặc vận hành tạo lô xuất kế toán cho ngày đã chốt; hộp thoại cho thấy trước số dòng, tổng tiền, thuế và bao nhiêu dòng đang ở trạng thái giữ lại.
6. Khi batch hoặc lô gửi lỗi, dòng lỗi hiện kèm hành động khắc phục; lỗi không được mất im lặng mà được đẩy sang hàng cảnh báo vận hành.
7. Phát hiện sai sau khi đã gửi: không có nút sửa; người dùng đi qua phiếu điều chỉnh, và hệ thống gửi lô chênh lệch thay thế lô gốc.

## Unresolved Questions

- **Cách gửi sang kế toán (Q-04)**: đã chốt file một chiều mỗi ngày qua thư mục do khách chỉ định (PREMISE C-03), thay vì kết nối tự động hai chiều (chênh lệch chi phí khoảng 4 lần). Khách chưa xác nhận — buổi họp về giao diện chưa diễn ra; còn mở: bên nào chỉnh sửa phía kế toán.
- **Trần ngân sách 60 triệu yên (Q-16)**: phủ chi phí một lần hay cả chi phí định kỳ — ảnh hưởng việc có nên làm tích hợp hai chiều hay không.
- **Cách hiểu "tra cứu 7 năm" (Q-10)**: toàn bộ 7 năm đều phải trả kết quả trong 2 giây hay chỉ 24 tháng gần nhất — quyết định chi phí hạ tầng và cửa sổ batch.
- **Ban quản lý được xem những báo cáo nào**: đặc biệt báo cáo log điều chỉnh sau chốt — là câu hỏi quản trị, chưa hỏi khách.
- **Khi nào dòng kế toán là "giữ lại", "đã xác nhận", "đã điều chỉnh"**: khách chốt ba giá trị nhưng không nói khi nào dùng giá trị nào.
- **Báo cáo trên ngày chưa chốt**: có cho xem số tạm hay chặn toàn bộ 12 báo cáo trước khi chốt?
- **Bộ cột của từng báo cáo**: khách chỉ cho tên và bộ lọc, chưa duyệt danh sách cột.
