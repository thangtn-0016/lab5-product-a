# Business Context — F008_IncentiveAndRateVersion

> Nhóm F08 của LAB-1 · 完納奨励金・料率バージョン · Tiền khuyến khích & quản lý phiên bản quy chế. Trạng thái: **đặc tả, chưa xây** (prototype LAB-3 không có phần tính tiền khuyến khích; đây là hạng mục bị lược bỏ có khai báo).

## Why It Matters

Tiền khuyến khích thanh toán đúng hạn (完納奨励金) hiện được tính bằng bảng tính cá nhân, tồn tại nhiều bản với ngày hiệu lực khác nhau. Hệ quả là khó kiểm chứng một con số, khó sửa có kiểm soát, và khi có thắc mắc phải đi tìm xem lúc đó đã dùng bản nào; sai lệch đối chiếu hiện ở mức 1,8% số dòng, mục tiêu của khách là không quá 0,3%.

Công thức do tài liệu khách quy định cứng, không đàm phán: hệ số thuế 110/100, làm tròn xuống theo đơn vị yên, không có thời gian gia hạn, quá hạn thanh toán thì bằng 0. Phần còn lại — tỷ lệ áp dụng — thay đổi theo quyết định của chợ, tối đa 4 lần mỗi năm, nên phải là **dữ liệu có phiên bản và ngày hiệu lực**, không phải hằng số trong phần mềm: in lại báo cáo kỳ cũ phải ra đúng con số cũ.

Đây là phép tính sâu nhất của hệ thống và là hạng mục khách kiểm bằng bộ số mẫu khi nghiệm thu; sai một đồng cũng là sai. Tính năng này nằm cuối luồng ngày nghiệp vụ ([FL-01](../../flows/FL-01-lot-trade-delivery-reconciliation.md)) và dùng chung nguyên tắc điều chỉnh sau chốt với luồng [FL-02](../../flows/FL-02-post-close-correction.md).

## Who Uses It

- **Nhân viên bộ phận đối chiếu (照合事務)** — chạy tính tiền khuyến khích cho kỳ đang mở, xem kết quả theo kỳ và theo người tham gia, giải thích một con số cũ bằng cách mở đúng phiên bản tỷ lệ đã dùng.
- **Quản trị quy chế (`rule_admin` — role đề xuất)** — lập phiên bản tỷ lệ mới với ngày hiệu lực trong tương lai, gửi duyệt, quay lui về phiên bản trước khi cần. Đây là người lập.
- **Người duyệt quy chế** (có cờ `can_approve` trên tài khoản — đã chốt PREMISE C-09) — một người khác người lập, đọc bản mới đặt cạnh bản đang hiệu lực rồi duyệt hoặc trả lại.
- **Kiểm toán nội bộ** — mở một bản ghi tiền khuyến khích là thấy ngay phiên bản tỷ lệ nào đã áp dụng, hiệu lực từ ngày nào, ai lập, ai duyệt.
- **Ban quản lý chợ** — hưởng lợi từ số liệu tháng đúng và từ việc đổi tỷ lệ không phải chờ bên đề xuất sửa phần mềm.

## What They Do

1. Quản trị quy chế lập một phiên bản tỷ lệ mới: nhập tỷ lệ, ngày hiệu lực trong tương lai, các tham số hạn thanh toán và lý do thay đổi — bản nháp **chưa có tác dụng gì**.
2. Quản trị quy chế gửi duyệt; một người khác đọc bản mới đặt cạnh bản đang hiệu lực rồi duyệt hoặc trả lại kèm lý do.
3. Nếu năm đó đã đổi đủ 4 lần, hệ thống từ chối bản mới — mức tối đa theo quy chế.
4. Khi duyệt xong, bản mới chờ tới ngày hiệu lực; trước ngày đó, mọi phép tính vẫn dùng bản cũ, và bản cũ chỉ chuyển sang đã thay thế đúng vào ngày hiệu lực của bản mới.
5. Nhân viên đối chiếu chạy tính cho kỳ đang mở: hệ thống áp tỷ lệ của phiên bản hiệu lực tại ngày nghiệp vụ, nhân hệ số thuế, làm tròn xuống theo yên; thanh toán quá hạn thì tiền khuyến khích bằng 0 chứ không giảm dần.
6. Mỗi dòng kết quả giữ nguyên phiên bản và ngày hiệu lực đã dùng; nhân viên bấm vào phiên bản để xem ai lập, ai duyệt, vì sao đổi.
7. Khi một điều chỉnh sau chốt kỳ được duyệt (nhóm điều chỉnh sau chốt), kỳ kế tiếp sinh thêm dòng chênh lệch có ghi rõ bù cho kỳ nào và vì sao; kỳ cũ không đổi một con số.
8. Muốn quay lui về phiên bản cũ, quản trị quy chế chọn ngày hiệu lực tương lai; hệ thống tạo một phiên bản mới sao nội dung bản cũ (không hồi sinh dòng cũ) và bản mới vẫn phải qua duyệt như mọi bản khác.

## Unresolved Questions

- **Bộ số mẫu của khách (Appendix C) — chưa có (Q-17)**: nghiệm thu đòi kết quả khớp bộ số mẫu, nhưng LAB-1 mới chỉ nhận được mô tả ba tình huống biên (đúng ngưỡng tối thiểu, điều chỉnh sau chốt kỳ, số thập phân sau hệ số thuế), không có số đầu vào và đầu ra. Không kiểm chứng được công thức nếu thiếu; đây là điểm chặn nghiệm thu cần lấy trước khi làm.
- **"Hạn thanh toán" tính từ mốc nào, và "đã thanh toán" là mốc nào (Q-09)**: mặc định đề xuất là một tham số của phiên bản quy chế, N ngày từ ngày nghiệp vụ; "đã thanh toán" là lúc bộ phận đối chiếu xác nhận trong hệ thống vì kết nối ngân hàng ngoài phạm vi. Công thức đúng mà mốc sai vẫn cho số sai — đây là rủi ro nghiệm thu cao nhất của nhóm.
- **"Kỳ" dài bao nhiêu**: tháng, nửa tháng hay theo lịch chợ; quyết định này đổi nghĩa của "kỳ kế tiếp" khi sinh chênh lệch. Liên quan lịch nghỉ chợ (Q-14).
- **Có ngưỡng tối thiểu để được hưởng không**: tình huống mẫu nhắc "đúng ngưỡng tối thiểu" nhưng không nói ngưỡng là gì.
- **Tỷ lệ thuộc chợ hay thuộc quốc gia**: đã chốt — thuộc quốc gia, một dòng thời gian duy nhất do F008 quản lý; `incentive_rate` không nằm trong quy tắc theo chợ của [F012](../F012_MarketManagement/business-context.md). Khách chưa xác nhận giả định này.
- **Tỷ lệ có khác nhau theo nhóm người tham gia không**: tài liệu chỉ nói một tỷ lệ.
- **Bản tỷ lệ đầu tiên lấy ở đâu**: cần khách chỉ định đúng một bản làm gốc kèm ngày hiệu lực, nếu không mọi phép tính lại dữ liệu cũ sẽ lệch.
- **Quay lui có tính vào trần 4 lần/năm không, và trần tính theo năm dương lịch hay năm tài chính**: giả định hiện tại là có tính và theo năm dương lịch; nếu không tính thì quay lui thành đường lách trần.
- **"Hoàn tất thanh toán" đo trên toàn phần hay từng lần giao**: giao nhiều lần làm câu này không hiển nhiên; RFP cũng để ngỏ thời điểm phản ánh vào quyết toán (Q-02).
- **Chạy tính thủ công hay theo lịch**: bản thiết kế chọn thủ công cho kỳ đang mở để người chịu trách nhiệm bấm nút.
