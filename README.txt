#Hướng dẫn chi tiết: Ứng dụng chia sẻ tệp ngang hàng (P2P) với Python
##Tổng quan
Ứng dụng chia sẻ tệp P2P này sử dụng một server trung gian để điều phối, giao tiếp thông qua giao thức TCP.

##Tác giả: Trần Ngọc Khánh Huy

##Hướng dẫn sử dụng
###1. Thiết lập chương trình
Chạy lệnh make để xóa cấu hình chương trình và cơ sở dữ liệu của các client.
###2. Khởi động server
Chạy script: ./run_server.sh
###3. Khởi động client
Mở ít nhất hai client từ các terminal khác nhau.
Ví dụ: chạy ./run_u1.sh và ./run_u2.sh.
###4. Sử dụng chương trình
Làm theo hướng dẫn từ menu giao diện để kiểm tra và sử dụng chương trình.
Thiết kế giao thức
Giao thức hoạt động ở tầng ứng dụng, truyền thông qua các kết nối TCP ngẫu nhiên (ephemeral connections).

Danh sách lệnh
OK: Xác nhận chung.
ERROR: Phản hồi lỗi chung.
Giao tiếp giữa CLIENT -> SERVER
###1. Đăng ký kết nối

####1.1. Kết nối lần đầu
Client gửi yêu cầu chào (greeting message) để đăng ký.

Trình tự giao tiếp:

Client: HELLO
Server: AVAILABLE <username>
Client: IWANT <username>
Server: WELCOME <username>
Mô tả:

Server tạo một username duy nhất theo mẫu "u#", bắt đầu từ 1, và gửi cho client.
Client có thể chọn username được đề xuất hoặc yêu cầu username khác.
Server kiểm tra username, nếu hợp lệ sẽ gửi WELCOME. Nếu không, server gửi lại AVAILABLE với một username mới. Quá trình tiếp tục đến khi chọn được username hợp lệ.
####1.2. Đã đăng ký username trước đó
Trình tự giao tiếp:

Client: HELLO <username>
Server: WELCOME
Nếu username không hợp lệ, server trả về:

Server: ERROR
###2. Gửi địa chỉ IP và cổng

Client: LISTENING <listening_ip> <listening_port>
Server: OK
Client gửi địa chỉ IP và cổng mà nó lắng nghe kết nối. Server xác nhận bằng lệnh OK.

###3. Gửi danh sách tệp được chia sẻ

Client: LIST <number_of_files>\n<file_a>\n<file_b>
Server: OK
Client gửi danh sách tệp dưới dạng:

Phần tiêu đề: LIST <số lượng tệp>.
Phần nội dung: Danh sách tệp, mỗi tệp trên một dòng.
Server xác minh danh sách và lưu trữ để chia sẻ với các client khác.

###4. Yêu cầu danh sách tệp chia sẻ

Client: SENDLIST
Server: FULLLIST <number_of_files>\n<client_a> <file_a>\n<client_b> <file_b>
Client yêu cầu danh sách đầy đủ các client và tệp của họ. Server trả về danh sách theo định dạng FULLLIST.

###5. Yêu cầu IP và cổng của peer để kết nối

Client: WHERE <peer_username>
Server: AT <peer_IP_address> <peer_port>
Nếu username không tồn tại:

Server: UNKNOWN
Nếu yêu cầu không hợp lệ:

Server: ERROR
Giao tiếp giữa CLIENT -> CLIENT
Tải xuống tệp từ peer

Client nhận: GIVE <file_name>
Peer gửi: TAKE <file_size>
Dữ liệu tệp được gửi.
Client nhận: THANKS
Nếu tệp không tồn tại:

Peer: ERROR
Lưu ý
Ký hiệu:
c: Client
s: Server
p: Peer
Mọi lệnh đều kết thúc bằng ký tự xuống dòng \n và ký tự null \0.
