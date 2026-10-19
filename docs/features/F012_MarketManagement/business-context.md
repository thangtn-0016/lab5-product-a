# Business Context — F012_MarketManagement

## Why It Matters

Đề bài LAB-4 nêu đích danh "quy tắc vận hành riêng của từng chợ". Bản prototype không có khái niệm chợ nào; căn cứ tham gia loại đăng ký tại điểm giao dịch ghi hành động xử lý vi phạm là "theo quy chế chợ" — trỏ tới một thứ không tồn tại. Đây là sai lệch nghiệp vụ D-02, hạng phải sửa; bản phân tích LAB-1 cũng không có màn nào cho việc này.

Quy chế chợ đổi theo quyết định hành chính, tối đa 4 lần mỗi năm, cần người lập khác người duyệt, có ngày hiệu lực và quay lui được. Thứ như vậy phải là dữ liệu có phiên bản, không phải hằng số trong code; nếu không, mỗi lần chợ đổi quy chế là một lần phát hành phần mềm và báo cáo kỳ cũ in lại sẽ ra số mới.

Tính năng này **không có trong 35 màn của LAB-1** và không có user story riêng; nó là thiết kế mới ở LAB-4 (đề xuất, rủi ro Trung bình: nếu khách thực tế chỉ vận hành một chợ thì phần lớn là chi phí thừa).

Feature gồm bốn tính năng con F12-01…F12-04, tất cả P0, và là gốc của các luồng khác: tư cách tham gia ([FL-03](../../flows/FL-03-participant-qualification-lifecycle.md)), chốt đấu giá, cảnh báo hết hạn đều đọc quy tắc từ đây.

## Who Uses It

- **Quản trị quy chế** (`rule_admin`, role đề xuất — chưa có trong hệ thống) — lập bản nháp quy tắc mới; là người lập.
- **Người duyệt quy chế** (`rule_admin` có cờ `can_approve` trên tài khoản — đã chốt PREMISE C-09) — duyệt hoặc quay lui; phải là người khác người lập.
- **Bộ phận đối chiếu** (`reconcile`) — chỉ xem để giải thích số liệu theo phiên bản quy tắc nào.
- **Quản lý người tham gia** (`party_admin`, role đề xuất) — xem quy chế khi lập hồ sơ người mua đăng ký tại điểm giao dịch.
- **Quản trị hệ thống** (`sys_admin`) — xem để hỗ trợ, không quyết định nghiệp vụ.

## What They Do

1. Quản trị quy chế khai báo một chợ: mã, tên Nhật/Việt, đơn vị mở chợ, múi giờ, khung giờ vận hành.
2. Chọn loại quy tắc (ví dụ điều kiện đăng ký người mua, điều kiện hạn chế có điều kiện, giờ cắt phiên đấu giá, số ngày cảnh báo hết hạn) và lập bản nháp mới, sao nội dung từ bản đang hiệu lực.
3. Nhập ngày hiệu lực trong tương lai và lý do thay đổi; bản nháp chưa có tác dụng gì.
4. Nếu năm đó đã có đủ 4 lần đổi được duyệt — hệ thống từ chối lập thêm.
5. Người có quyền duyệt (khác người lập) xem bản mới đặt cạnh bản đang hiệu lực rồi duyệt.
6. Đến ngày hiệu lực, bản mới thay bản cũ; trước ngày đó mọi tra cứu theo ngày cũ vẫn thấy bản cũ.
7. Cần quay lại: chọn một bản cũ, hệ thống tạo bản mới sao nội dung đó (không hồi sinh dòng cũ), vẫn qua duyệt.
8. Mọi màn nghiệp vụ tra quy tắc theo ngày nghiệp vụ của giao dịch, không theo giờ máy.

Hệ số thuế 110/100, làm tròn xuống theo yên và tỷ lệ khuyến khích 料率 là quy tắc quốc gia, không thuộc quy tắc chợ — không đưa vào đây (料率 có phiên bản riêng ở [F008](../F008_IncentiveAndRateVersion/business-context.md)).

## Unresolved Questions

- **Một chợ hay nhiều chợ?** Quyết định này quyết định tính năng có cần thiết hay chỉ là một trang cấu hình (câu hỏi gốc của kiến trúc; chưa có trong danh sách Q của LAB-1).
- **Điều kiện hạn chế có điều kiện cụ thể gồm những gì** (trần số lượng, trần giá trị, cấm đấu giá, hạn)? Chưa có trong tài liệu khách.
- **Thẩm quyền duyệt quy chế chợ** đã chốt dùng chung cờ `can_approve` với duyệt tư cách (PREMISE C-09); khách chưa xác nhận giả định đó, và nếu khách tách thì cần hai cờ.
- **Trần 4 lần mỗi năm tính theo năm dương lịch hay năm tài chính Nhật** (tháng 4 đến tháng 3), và quay lui có bị tính vào trần không?
- **Chợ có bao giờ bị xoá không?** Đề xuất: chỉ ngừng hoạt động vì dữ liệu lịch sử vẫn trỏ về.
