# Business Context — F007_PostCloseCorrection

> Nhóm F07 của LAB-1 · 修正申請・承認 · Điều chỉnh sau chốt. Trạng thái: **đặc tả, chưa xây** (prototype LAB-3 dừng ở chỗ chặn sửa trực tiếp; đường sửa hợp lệ là hạng mục bị lược bỏ có khai báo).

## Why It Matters

Khách đặt một nguyên tắc duy nhất cho nhóm này: **lịch sử không được sửa đè**. Hiện nay việc sửa số liệu sau khi đã xác nhận được làm thẳng trên bảng tính, nên chỉ khoảng 35% giao dịch còn đủ dấu vết ai sửa, sửa gì, vì sao; mục tiêu của khách là đưa con số này lên từ 98% trở lên. Sau khi sổ ngày đã chốt, hệ thống chặn mọi lần sửa trực tiếp — nên cần một con đường duy nhất, có người chịu trách nhiệm, để sửa cái sai mà không làm đổi con số đã công bố.

Nhóm này nằm trên đường găng của dự án (nhóm khó nhất về kỹ thuật theo function list), vì cách lưu dữ liệu phải theo nguyên tắc chỉ thêm, không sửa ngay từ đầu — chọn sai thì về sau không sửa lại được.

Tính năng này là một phần của luồng điều chỉnh sau chốt ([FL-02](../../flows/FL-02-post-close-correction.md), maker-checker) và nối tiếp luồng ngày nghiệp vụ ([FL-01](../../flows/FL-01-lot-trade-delivery-reconciliation.md)) ở bước chốt sổ.

## Who Uses It

- **Nhân viên bộ phận đối chiếu (照合事務局)** — phát hiện sai sau khi sổ đã chốt; lập phiếu xin điều chỉnh kèm lý do và chứng cứ. Đây là người lập phiếu.
- **Người có thẩm quyền duyệt (承認者)** — đọc lý do, chứng cứ, so số trước với số sau rồi duyệt hoặc từ chối. Phải là một người khác người lập phiếu. Theo giả định đang dùng của LAB-1 (một cấp duyệt), đây là trưởng bộ phận đối chiếu.
- **Kiểm toán nội bộ** — không thao tác trên tính năng này nhưng đọc lại dấu vết: bản điều chỉnh sinh ra từ phiếu nào, ai duyệt, vì sao.
- **Ban quản lý** — hưởng lợi gián tiếp: số liệu đã công bố không bị đổi âm thầm sau lưng.

## What They Do

1. Nhân viên đối chiếu phát hiện một số liệu sai trên ngày nghiệp vụ đã chốt — hệ thống từ chối sửa trực tiếp và dẫn họ sang màn xin điều chỉnh.
2. Nhân viên đối chiếu chọn bản ghi cần sửa (giao dịch, lần giao hàng hoặc kết quả tiền khuyến khích), chọn kiểu điều chỉnh (đảo toàn bộ hoặc ghi phần chênh lệch), nhập số sau điều chỉnh, lý do và chứng cứ — phiếu được gửi đi, **số liệu gốc chưa đổi một ký tự nào**.
3. Nếu bản ghi đang có một phiếu chờ duyệt khác, hoặc ngày của bản ghi chưa chốt — hệ thống không nhận phiếu và nói rõ lý do (chưa chốt thì sửa trực tiếp, không cần phiếu).
4. Người duyệt mở danh sách phiếu chờ, thấy phiếu nào chờ lâu nhất; phiếu do chính mình lập thì không có nút duyệt.
5. Người duyệt đọc lý do, mở chứng cứ, xem bảng so sánh trước/sau (số trước được đọc lại từ bản ghi hiện tại, không dùng ảnh chụp cũ), rồi duyệt hoặc từ chối — từ chối bắt buộc nêu lý do.
6. Khi duyệt, hệ thống **tạo bản ghi điều chỉnh mới** ở ngày nghiệp vụ đang mở; bản ghi gốc và báo cáo của ngày đã chốt vẫn nguyên. Bản điều chỉnh truy ngược được về phiếu đã sinh ra nó.
7. Với tiền khuyến khích, điều chỉnh sau khi chốt kỳ sinh chênh lệch ở kỳ kế tiếp; in lại báo cáo kỳ cũ vẫn ra đúng số cũ.
8. Nếu người lập muốn rút phiếu khi chưa ai duyệt thì phiếu chuyển sang trạng thái đã rút nhưng vẫn lưu dấu vết; duyệt nhầm thì cách sửa duy nhất là lập một phiếu điều chỉnh ngược.

Hệ thống **không bao giờ tự duyệt**: đây là một trong bốn điểm khách cố ý giữ quyết định cho con người.

## Unresolved Questions

- **Số cấp duyệt (Q-03)**: đã chốt một cấp (PREMISE C-03b) — trưởng bộ phận đối chiếu duyệt, người xin khác người duyệt, không có cấp thứ ba; quyền duyệt là cờ `can_approve` trên tài khoản (PREMISE C-09). Khách chưa xác nhận giả định này. Nếu khách phân cấp theo số tiền thì số luồng duyệt tăng từ một lên ba đến năm và cần thêm một màn định nghĩa ngưỡng; LAB-1 định lượng +10…14 MD.
- **Bảng phân quyền theo trạng thái nghiệp vụ (業務状態マトリクス)**: tài liệu khách nhắc tới nhưng chưa gửi; thiếu nó thì ai được xin, ai được duyệt theo từng trạng thái chỉ là giả định.
- **"Đính kèm chứng cứ" là đường dẫn hay file thật**: hiện thiết kế là ô văn bản (đường dẫn hoặc số hiệu văn bản); nếu khách hiểu là tải file lên thì phát sinh kho lưu trữ, phân quyền và quét mã độc.
- **Điều chỉnh được những trường nào**: hiện giả định số lượng và đơn giá; đổi người mua hoặc lô chỉ qua đảo toàn bộ rồi lập giao dịch mới. Khách chưa nói gì.
- **Một bản ghi được có mấy phiếu chờ duyệt cùng lúc**: thiết kế chặn ở một phiếu để đơn giản; chưa có yêu cầu nào quyết.
- **Phiếu treo quá lâu có cần hạn xử lý không** (đề xuất: cảnh báo sau 3 ngày làm việc, không tự huỷ) và có cần thông báo cho người duyệt khi có phiếu mới không.
- **Điều chỉnh lần giao hàng có kéo theo điều chỉnh giao dịch không**: hai bản ghi liên đới nhau nhưng tài liệu khách không nói.
- **Bản ghi điều chỉnh có đi sang hệ thống kế toán không, dưới trạng thái nào**: trạng thái dòng "調整済み" nhiều khả năng là dòng sinh từ đây nhưng tài liệu khách chưa nối hai chỗ lại với nhau (liên quan Q-04).
