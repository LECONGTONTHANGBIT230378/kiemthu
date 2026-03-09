# kiemthu
Báo cáo Kết quả Kiểm thử Hiệu năng (JMeter)
1. Tổng quan Test Plan
Kịch bản kiểm thử được chia thành 3 Thread Group chính để đánh giá các khía cạnh khác nhau của hệ thống:

TG01_Basics: Kiểm tra kết nối cơ bản đến trang chủ.

TG02_HeavyLoad: Giả lập tải nặng (Heavy Load) đan xen giữa Home Page và About Page.

TG03_Custom: Đánh giá hiệu năng của tính năng tìm kiếm (Search_JMeter).

2. Phân tích chi tiết từng Thread Group
2.1. Nhóm 1: TG01_Basics (Truy cập cơ bản)
Thành phần theo dõi: View Results Tree.

Tình trạng hiển thị: Request báo thành công (icon tick xanh).

 Cảnh báo cấu hình (Quan trọng):
Mặc dù JMeter hiển thị tick xanh, nhưng khi xem chi tiết phần Response data, hệ thống đang trả về lỗi nội bộ của máy khách:
Response code: Non HTTP response code: org.apache.http.client.ClientProtocolException
Response message: Non HTTP response message: URI does not specify a valid host name: http:/

Phân tích: Kịch bản này chưa thực sự gửi request thành công đến server. Lỗi URI does not specify a valid host name chỉ ra rằng bạn đang để trống hoặc cấu hình sai trường Server Name or IP (có thể do thừa dấu / hoặc thiếu tên miền thực tế) trong HTTP Request hoặc HTTP Request Defaults. Thời gian Load time (1ms) và Latency (0ms) chứng tỏ request chưa hề đi ra ngoài mạng.

2.2. Nhóm 2: TG02_HeavyLoad (Tải nặng)
Thành phần theo dõi: View Results in Table.

Luồng xử lý: Gọi luân phiên GET Home Page và GET About Page.

Dữ liệu thống kê:

Tổng số mẫu quan sát: 48.

Thời gian phản hồi trung bình (Average): 481 ms.

Độ lệch chuẩn (Deviation): 255 ms.

Phân tích:

Mức trung bình 481ms là một con số khá tốt cho việc tải trang cơ bản. Cả hai trang Home và About đều trả về lượng byte tương đối ổn định (Home ~131KB, About ~161KB).

Tuy nhiên, hệ thống xuất hiện các điểm nghẽn đột biến (spikes). Điển hình là Sample #9 (GET About Page) mất 1691 ms, hay Sample #14 (GET Home Page) mất 1527 ms. Với độ lệch chuẩn khá cao (255ms), có thể thấy dưới áp lực tải nặng, server xử lý chưa thực sự đồng đều và thỉnh thoảng bị quá tải nhẹ.

2.3. Nhóm 3: TG03_Custom (Chức năng Tìm kiếm)
Thành phần theo dõi: View Results in Table.

Luồng xử lý: Search_JMeter.

Dữ liệu thống kê:

Tổng số mẫu quan sát: 20.

Thời gian phản hồi trung bình (Average): 1178 ms.

Độ lệch chuẩn (Deviation): 415 ms.

Phân tích:

Đây là tác vụ tốn kém tài nguyên nhất trong toàn bộ kịch bản. Thời gian phản hồi trung bình vượt quá 1 giây.

Sample #7 có tổng thời gian xử lý lên tới 2198 ms, trong đó chỉ riêng Latency (thời gian chờ phản hồi byte đầu tiên từ server) đã chiếm tới 1500 ms. Điều này cho thấy backend/database đang mất rất nhiều thời gian để thực thi logic tìm kiếm trước khi kịp trả data về cho client.

3. Kết luận & Khuyến nghị hành động
Khắc phục lỗi cấu hình TG01: Kiểm tra lại toàn bộ URL, IP, và Port trong phần cấu hình của TG01_Basics để đảm bảo request được gửi đến đúng máy chủ thực tế.

Tối ưu hóa thuật toán/Database cho chức năng Search: Phân tích lại API tương ứng với TG03_Custom. Cần xem xét việc đánh Index cho Database, áp dụng Caching, hoặc tối ưu hóa lại câu truy vấn tìm kiếm để giảm thiểu Latency (hiện đang rất cao, lên tới 1.5s).

Giám sát Server (Monitoring): Để giải quyết các điểm nghẽn đột biến (>1.5s) trong TG02_HeavyLoad, cần kết hợp theo dõi các thông số phần cứng của Server (CPU, RAM, Disk I/O) trong quá trình test để tìm ra giới hạn thắt cổ chai thực sự.
