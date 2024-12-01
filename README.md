```markdown
# Ứng dụng Chia sẻ Tệp Ngang hàng (P2P)

## Tổng quan

Ứng dụng chia sẻ tệp P2P này sử dụng một server trung gian để điều phối và giao tiếp giữa các peer thông qua giao thức TCP.

### Tác giả

**Trần Ngọc Khánh Huy**

---

## Hướng dẫn sử dụng

### 1. Thiết lập chương trình
Chạy lệnh sau để xóa cấu hình chương trình và cơ sở dữ liệu của các client:
```bash
make
```

### 2. Khởi động server
Chạy script server:
```bash
./run_server.sh
```

### 3. Khởi động client
Mở ít nhất hai client từ các terminal khác nhau. Ví dụ:
```bash
./run_u1.sh
./run_u2.sh
```

### 4. Sử dụng chương trình
Làm theo hướng dẫn trên giao diện menu của client để kiểm tra và sử dụng ứng dụng.

---

## Thiết kế giao thức

Giao thức hoạt động ở tầng ứng dụng, sử dụng các kết nối TCP tạm thời (ephemeral connections).

### Danh sách lệnh
- **OK**: Phản hồi xác nhận chung.
- **ERROR**: Phản hồi lỗi chung.

### Chi tiết giao tiếp

#### CLIENT ↔ SERVER

**1. Đăng ký kết nối**

- **1.1 Kết nối lần đầu**

  Client gửi yêu cầu chào để đăng ký:

  **Trình tự:**
  ```
  Client: HELLO
  Server: AVAILABLE <username>
  Client: IWANT <username>
  Server: WELCOME <username>
  ```

  **Mô tả:**
  - Server tạo một username duy nhất theo mẫu `u#` (ví dụ: `u1`, `u2`) và gửi cho client.
  - Client có thể chấp nhận username được đề xuất hoặc yêu cầu một username khác.
  - Server kiểm tra username:
    - Nếu hợp lệ: trả về `WELCOME`.
    - Nếu không hợp lệ: gửi lại `AVAILABLE` với username mới. Quá trình tiếp tục cho đến khi chọn được username hợp lệ.

- **1.2 Đã đăng ký username trước đó**

  **Trình tự:**
  ```
  Client: HELLO <username>
  Server: WELCOME
  ```

  Nếu username không hợp lệ:
  ```
  Server: ERROR
  ```

**2. Gửi địa chỉ IP và cổng**

```
Client: LISTENING <listening_ip> <listening_port>
Server: OK
```
- Client gửi địa chỉ IP và cổng mà nó đang lắng nghe. Server xác nhận bằng `OK`.

**3. Gửi danh sách tệp chia sẻ**

```
Client: LIST <number_of_files>\n<file_a>\n<file_b>
Server: OK
```
- Client gửi danh sách tệp chia sẻ:
  - Phần tiêu đề: `LIST <số lượng tệp>`.
  - Phần nội dung: Danh sách tệp, mỗi tệp trên một dòng.
- Server xác minh và lưu trữ danh sách để chia sẻ với các client khác.

**4. Yêu cầu danh sách tệp chia sẻ**

```
Client: SENDLIST
Server: FULLLIST <number_of_files>\n<client_a> <file_a>\n<client_b> <file_b>
```
- Client yêu cầu danh sách đầy đủ các tệp được chia sẻ.
- Server trả về danh sách chi tiết các client và tệp của họ.

**5. Yêu cầu IP và cổng của peer**

```
Client: WHERE <peer_username>
Server: AT <peer_IP_address> <peer_port>
```
- Nếu username không tồn tại:
  ```
  Server: UNKNOWN
  ```
- Nếu yêu cầu không hợp lệ:
  ```
  Server: ERROR
  ```

#### CLIENT ↔ CLIENT

**Tải tệp từ peer**

- **Trình tự:**
  ```
  Client: GIVE <file_name>
  Peer: TAKE <file_size>
  <Dữ liệu tệp>
  Client: THANKS
  ```
  - Nếu tệp không tồn tại:
    ```
    Peer: ERROR
    ```

---

## Lưu ý

### Ký hiệu
- `c`: Client
- `s`: Server
- `p`: Peer
- Mọi lệnh đều kết thúc bằng ký tự xuống dòng (`\n`) và ký tự null (`\0`).
```
