# MÔ HÌNH ĐE DỌA – CHỦ ĐỀ A

## 1. Bối cảnh và tài sản cần bảo vệ

Hệ thống phục vụ khoảng 200 người dùng nội bộ, gồm ba thành phần chính: nginx làm cổng vào, web application và PostgreSQL. nginx sử dụng TLS cho toàn bộ lưu lượng và mTLS riêng cho đường quản trị `/admin`.

Các tài sản cần bảo vệ chính gồm dữ liệu và thông tin xác thực lưu trong PostgreSQL, dữ liệu riêng của từng phòng ban, đường quản trị `/admin`, cùng Private Keys và Certificates phục vụ TLS/mTLS.

**Luồng dữ liệu chính:**  
Người dùng nội bộ → nginx → web app → PostgreSQL.  
Đường `/admin` đi qua nginx và yêu cầu mTLS; kết nối giữa web app và PostgreSQL được thiết kế buộc sử dụng SSL.

## 2. Các mối đe dọa và mức ưu tiên

Nhóm xác định ba mối đe dọa bắt buộc của Chủ đề A và xếp hạng theo công thức tác động × khả năng xảy ra:

| ID | Mối đe dọa | Tác động | Khả năng | Điểm ưu tiên |
|---|---|---:|---:|---:|
| T1 | Nhân viên phòng ban A dùng tài khoản hợp lệ truy cập dữ liệu phòng ban B | 5 | 4 | 20 |
| T3 | Kết nối web app – PostgreSQL cho phép truyền dữ liệu không mã hóa | 4 | 4 | 16 |
| T2 | Dữ liệu hoặc thông tin đăng nhập bị bắt trên mạng nội bộ do kết nối chưa mã hóa | 5 | 3 | 15 |

Thứ tự ưu tiên xử lý là **T1 → T3 → T2**.

## 3. Biện pháp kiểm soát dự kiến

T1 được xử lý bằng **Row Level Security của PostgreSQL**, nhằm giới hạn dữ liệu theo phạm vi phòng ban và thực hiện nguyên lý đặc quyền tối thiểu.

T2 được xử lý bằng **TLS và mTLS tại nginx**, trong đó TLS bảo vệ lưu lượng và mTLS được áp dụng riêng cho đường quản trị `/admin`.

T3 được xử lý bằng việc **bắt buộc kết nối SSL giữa web app và PostgreSQL**, không cho phép sử dụng kết nối cơ sở dữ liệu không mã hóa.

## 4. Phòng thủ nhiều lớp

Hai lớp phòng thủ độc lập chính của hệ thống là lớp bảo vệ tại nginx bằng TLS/mTLS và lớp kiểm soát dữ liệu tại PostgreSQL bằng Row Level Security.

Nếu lớp bảo vệ bên ngoài tại nginx bị vượt qua, tác nhân không mặc nhiên có quyền đọc toàn bộ dữ liệu. PostgreSQL vẫn duy trì RLS để giới hạn dữ liệu theo phạm vi phòng ban, vì vậy tài khoản của phòng ban A vẫn không được phép đọc các hàng dữ liệu thuộc phòng ban B. Kết nối giữa web app và PostgreSQL đồng thời được buộc sử dụng SSL, tiếp tục bảo vệ dữ liệu trên đường truyền nội bộ và giúp giảm phạm vi ảnh hưởng khi lớp ngoài thất bại.
