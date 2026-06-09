# APP MONITOR + ALERT DATA REALTIME
MÔN HỌC: PHÁT TRIỂN ỨNG DỤNG TRÊN THIẾT BỊ DI ĐỘNG   
Môn học: Phát triển ứng dụng với mã nguồn mở   
Họ và tên: Nguyễn Thị Kim Huệ  
MSSV: K225480106026  
YÊU CẦU BÀI TẬP  
## LÝ THUYẾT  
+ docker là gì?   
+ các keyword được sử dụng trong docker-compose.yml  
  để mô tả 1 service, network, volume,...  
  liệt kê + ý nghĩa của từ khoá đó + ví dụ minh hoạ  
+ ưu điểm khi triển app sử dụng docker là gì?  
+ dùng docker: tạo app, test app OK trên laptop cá nhân  
  giờ muốn triển khai app này trên máy chủ thật ko có internet  
  thì các bước cần làm là?
  
THỰC HÀNH ÁP DỤNG  
sử dụng docker compose có nhiều serivce   
và các thành phần cần thiết để tạo thành ứng dụng:  
 + nodered liên tục lấy dữ liệu từ nguồn nào đó (chứng khoán, thời tiết, giá vàng,...)  
   nguồn thực tế, số liệu luôn động sau thời gian ngắn  
 + nodered lưu trữ dữ liệu vào 2 database: mariadb để lưu giá trị tức thời  
   lưu lịch sử vào influxdb  
 + sử dụng grafana để trực quan hoá dữ liệu: vẽ biểu đồ  
 + sử dụng nginx để làm webserver  
   chạy 1 trang web html+js+css làm front-end  
   js: lấy dữ liệu tức thời trong mariadb qua (ajax | socket)   
       gọi api (api tự build bằng Flask giống bt1)  
       api trả về giá trị tức thời trong mariadb  
       hiển thị lên web, auto hiển thị số mới khi thay đổi  
   sử dụng iframe để gọi grafana  
   hiển thị biểu đồ dữ liệu lịch sử của thông số đã lưu  
 + QUAN SÁT DỮ LIỆU LỊCH SỬ => GIÁ TRỊ BẤT THƯỜNG  
   (VD MIỀN A..B: OK, DƯỚI A: ALERT LOW, TRÊN B: ALERT HIGH)  
 + nodered: kết hợp bot Telegram  
   khi dữ liệu not OK, thì gửi tin nhắn từ bot => group trên telegram  
   group đã add bot vào: (nhóm đã có 2 người), add thêm 1875746636 thành 3 người  
   mỗi khi bot gửi dữ liệu vào nhóm: mọi member of group đều nhận đc  
   nội dung alert: tường minh, có value gây alert  

 xuất tất cả các container ra file nén.  
 xoá mọi container đang chạy  
 load lại các container  từ file nén để khôi phục các container đã xoá   
 
### BÀI LÀM  
### PHẦN 1: LÝ THUYẾT  
## 1. Docker là gì?
   Docker là một nền tảng phần mềm giúp bạn building, deploying và running ứng dụng dễ dàng hơn bằng cách sử dụng các containers (trên nền tảng ảo hóa).
  Docker đóng gói phần mềm thành các container tiêu chuẩn hóa, chứa đựng tất cả những thứ cần thiết để phần mềm hoạt động như thư viện, công cụ hệ thống, mã nguồn và thời gian chạy. Khi cần deploy app lên bất kỳ server nào, bạn chỉ cần run container của Docker thì app của bạn sẽ được khởi chạy ngay lập tức.  

Khi sử dụng Docker, bạn có thể dễ dàng triển khai và mở rộng quy mô ứng dụng trong bất kỳ môi trường nào, đồng thời đảm bảo rằng mã nguồn của bạn sẽ luôn chạy được một cách ổn định.    
## 2. Các từ khóa thường dùng trong docker-compose.yml  
<img width="782" height="298" alt="image" src="https://github.com/user-attachments/assets/685a2db6-9519-4758-8826-70b34e4419f6" />  

2.1 Các keyword mô tả service  
`image`  

Chỉ định image có sẵn (từ Docker Hub hoặc local) để chạy container.  
```yaml
  services:   
    database:     
      image: mariadb:latest   # dùng image mariadb phiên bản mới nhất
```

`build`  

Thay vì dùng image có sẵn, tự build image từ Dockerfile.  
```yaml
services:  
  flask_api:  
    build:  
      context: ./flask_app    # thư mục chứa Dockerfile  
      dockerfile: Dockerfile  # tên file (mặc định là "Dockerfile")
```

`container_name`   

Đặt tên cụ thể cho container thay vì để Docker tự sinh tên ngẫu nhiên.  
```yaml
services:  
  web:  
    image: nginx  
    container_name: my_nginx
```
`ports`  

Map cổng theo cú pháp HOST:CONTAINER — cho phép truy cập từ bên ngoài máy host.  
```yaml
services:  
  grafana:  
    image: grafana/grafana  
    ports:  
      - "3000:3000"   # truy cập http://localhost:3000 → grafana bên trong container  
  ```
 
`environment`

Truyền biến môi trường vào trong container  
```yaml
services:  
  mariadb:  
    image: mariadb:latest  
    environment:  
      MYSQL_ROOT_PASSWORD: root123  
      MYSQL_DATABASE: alert_db  
      MYSQL_USER: admin  
      MYSQL_PASSWORD: admin123
```

`env_file ` 

Đọc biến môi trường từ file .env bên ngoài — tránh lộ thông tin nhạy cảm trong compose file.  
```yaml
services:  
  mariadb:    
    env_file:   
      - .env
```

`volumes`  

Mount dữ liệu giữa host và container. Có 2 loại:  
```yaml
Named volume: Docker tự quản lý vị trí lưu trữ.  
Bind mount: Mount trực tiếp thư mục/file từ máy host.  
services:  
  mariadb:  
    volumes:  
      - mariadb_data:/var/lib/mysql        # named volume  
      - ./config/my.cnf:/etc/mysql/my.cnf  # bind mount (file cụ thể)  
```

`networks`

Chỉ định container tham gia vào mạng nào. Một container có thể thuộc nhiều mạng.  
```yaml
services:  
  flask_api:  
    networks:  
      - frontend_net  
      - backend_net
```
   
`depend_on`  

Xác định thứ tự khởi động — service này chỉ start sau khi các service phụ thuộc đã sẵn sàng.  
```yaml
services:  
  flask_api:  
    depends_on:  
      - mariadb    # mariadb phải khởi động trước flask_api  
      - influxdb
```

`restart`

Chính sách tự khởi động lại container khi bị crash hoặc khi Docker daemon restart  
```yaml
services:  
  nodered:  
    restart: always          # luôn luôn restart  
    # restart: unless-stopped  # restart trừ khi bị dừng thủ công bằng docker stop  
    # restart: on-failure      # chỉ restart khi thoát với mã lỗi khác 0  
    # restart: no              # không bao giờ tự restart
```

`command`

Ghi đè lệnh mặc định (CMD) được định nghĩa trong image khi container khởi động.  
```yaml
services:  
  flask_api:  
    command: python app.py --port 5000
```

`healthcheck`  

Định nghĩa câu lệnh kiểm tra định kỳ xem service có hoạt động đúng không.  
```yaml
services:  
  mariadb:  
    healthcheck:  
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]  
      interval: 10s   # kiểm tra mỗi 10 giây  
      timeout: 5s     # timeout sau 5 giây  
      retries: 5      # thử lại 5 lần trước khi đánh dấu "unhealthy"
```

`expose`

Mở cổng cho các container khác trong cùng network — không mở ra ngoài máy host.  
```yaml
services:  
  flask_api:  
    expose:  
      - "5000"   # container khác trong cùng network có thể gọi :5000  
                 # nhưng máy host bên ngoài không truy cập được
```

`entrypoint`

Ghi đè lệnh ENTRYPOINT mặc định của image.  
```yaml
services:  
  flask_api:  
    entrypoint: ["python", "-m", "flask", "run"]
```

`working_dir`  

Đặt thư mục làm việc mặc định bên trong container.  
```yaml
services:  
  flask_api:  
    working_dir: /app
```
## 3. Ưu điểm khi triển khai ứng dụng bằng Docker
   <img width="782" height="596" alt="image" src="https://github.com/user-attachments/assets/1f41e697-a2bf-4e2f-8e46-65b835a1fee0" />

## 4. Triển khai lên máy chủ thật không có internet
Trong một số môi trường bảo mật cao, máy chủ triển khai không được phép kết nối Internet. Khi đó cần chuẩn bị toàn bộ Docker Image trên máy tính có Internet trước khi chuyển sang máy chủ đích.    

Bước 1: Trên Laptop - Tải và Build các Docker Image  

```yaml
Tải tất cả các Docker Image được khai báo trong file docker-compose.yml:  
docker compose pull  
Nếu ứng dụng sử dụng Dockerfile riêng, thực hiện build image:  
docker compose build  
Kiểm tra các image đã có trong máy:  
docker images  
```

Bước 2: Trên Laptop - Export Docker Image  
```yaml
Có thể export từng image riêng lẻ:  
docker save mariadb:10.11 -o mariadb.tar  
docker save influxdb:2.7 -o influxdb.tar  
docker save grafana/grafana -o grafana.tar  
docker save nodered/node-red -o nodered.tar  
docker save nginx:alpine -o nginx.tar  
docker save my_flask_api:latest -o flask_api.tar  

Hoặc export tất cả image vào một file duy nhất:  
docker save \  
mariadb:10.11 \  
influxdb:2.7 \  
grafana/grafana \  
nodered/node-red \  
nginx:alpine \  
my_flask_api:latest \  
-o project_images.tar  
```

Bước 3: Chuyển dữ liệu sang máy chủ  

```yaml
Sao chép các tệp sau sang máy chủ:    
docker-compose.yml  
file .env (nếu có)  
project_images.tar  
source code hoặc dữ liệu cần thiết  

Có thể sử dụng USB, ổ cứng di động hoặc mạng LAN nội bộ để sao chép.  
```

Bước 4: Trên máy chủ - Import Docker Image  

```yaml
Kiểm tra Docker đã được cài đặt:  
docker --version  
Import image vào máy chủ:  
docker load -i project_images.tar  
Kiểm tra danh sách image:  
docker images   
```

Bước 5: Khởi động hệ thống

```yaml
Di chuyển đến thư mục chứa file docker-compose.yml:  
cd project  
Khởi động toàn bộ hệ thống:  
docker compose up -d  
Kiểm tra các container đang hoạt động:  
docker ps  
Xem log nếu cần:   
docker compose logs -f  

Bước 6: Kiểm tra ứng dụng

```yaml
Kiểm tra trạng thái các container:  
docker ps -a  
Kiểm tra các cổng dịch vụ:  
netstat -tulpn  
Hoặc truy cập ứng dụng từ trình duyệt:  
http://IP_MAY_CHU  
```
Kết luận
Quy trình triển khai Docker trên máy chủ không có Internet gồm các bước: tải và build image trên máy có Internet, export image thành file .tar, sao chép sang máy chủ, import image và khởi động hệ thống bằng Docker Compose. Phương pháp này giúp triển khai ứng dụng nhanh chóng, đảm bảo tính nhất quán và phù hợp với các môi trường có yêu cầu bảo mật cao.    

## PHẦN 2: THỰC HÀNH  
### 1 Mục tiêu

Xây dựng hệ thống giám sát dữ liệu thời gian thực (Realtime Monitoring System) bằng Docker Compose. Hệ thống có khả năng thu thập dữ liệu từ nguồn thực tế, lưu trữ, trực quan hóa dữ liệu, phát hiện bất thường và gửi cảnh báo tự động qua Telegram.

### 2 Kiến trúc hệ thống

Hệ thống gồm các thành phần sau:

-Node-RED: Thu thập dữ liệu thời gian thực từ nguồn bên ngoài (thời tiết, giá vàng, chứng khoán,...).  
-MariaDB: Lưu trữ dữ liệu hiện tại (giá trị tức thời).   
-InfluxDB: Lưu trữ dữ liệu lịch sử theo chuỗi thời gian.  
-Grafana: Hiển thị biểu đồ dữ liệu lịch sử.  
-Flask API: Cung cấp API truy vấn dữ liệu tức thời từ MariaDB.  
-Nginx: Web Server phục vụ giao diện người dùng.  
-Telegram Bot: Gửi cảnh báo khi phát hiện dữ liệu bất thường.    

1.Khởi động hệ thống  
Sau khi hoàn tất cấu hình, build và khởi động toàn bộ hệ thống:  
<img width="1103" height="317" alt="2" src="https://github.com/user-attachments/assets/80938220-c2e9-4ba4-8389-0db0d1d50c36" />  

Kiểm tra trạng thái các container  
![Uploading image.png…])  
2 Cấu hình NODERED để tự động hóa luồng dữ liệu  
Trong giai đoạn này, Node-RED sẽ đóng vai trò đầu não thực hiện 4 nhiệm vụ liên tục:  

Cào dữ liệu thời tiết thực tế từ API công khai. (mỗi 10-30 giây)  
Lưu trạng thái mới nhất vào MariaDB.  
Lưu lịch sử vào InfluxDB (để Grafana vẽ biểu đồ).  
Phân tích dị thường và kích hoạt Telegram Bot gửi tin nhắn cảnh báo vào Group.  
Bước 1: Chuẩn bị thư viện (nodes) trong Node-RED  
Truy cập Node-RED qua địa chỉ http://<IP_máy_chủ_Ubuntu>:1882  
`http://192.168.164.129:1882 ` 
Click vào Menu (3 dấu gạch ngang góc trên bên phải) -> Chọn Manage palette.Chuyển sang thẻ Install, tìm kiếm và nhấn Install lần lượt 3 thư viện sau:
```yaml
node-red-node-mysql (Kết nối MariaDB)
node-red-contrib-influxdb (Kết nối InfluxDB)
node-red-contrib-telegrambot (Kết nối Telegram)
```
<img width="1920" height="972" alt="3" src="https://github.com/user-attachments/assets/b3f6bfc6-09bb-4f04-9a0c-2877dffe0b77" />   
Bước 2: Chuẩn bị thông tin Telegram Bot  
Trước khi viết Flow, cần chuẩn bị thông tin từ Telegram:  
Bot Token: Chat với @BotFather trên Telegram, gõ lệnh /newbot, đặt tên cho bot. Sau khi tạo xong, @BotFather sẽ cấp một chuỗi Token. Copy token này để bước sau dán vào Nodered.  
<img width="893" height="904" alt="4" src="https://github.com/user-attachments/assets/8cdf5b1c-4653-44b5-80c6-56bd9fc9f4b3" />  
Tạo nhóm chat có bot để cảnh báo:  

Tạo một Group mới trên Telegram, thêm các thành viên vào (bao gồm cả tài khoản ID 1875746636 theo yêu cầu bài tập).  
Thêm cả con Bot vừa tạo ở trên vào nhóm này với quyền Admin (để nó có quyền gửi tin nhắn).  
<img width="968" height="907" alt="5" src="https://github.com/user-attachments/assets/a8b6ce02-01c5-4a08-8a76-368bcf4df4d7" />  
Bước 3:Deploy và kiểm tra  
Bấm nút Deploy màu đỏ trên góc phải màn hình để lưu và chạy.   
<img width="1559" height="969" alt="6" src="https://github.com/user-attachments/assets/0fa9e3d9-fc89-4f5f-9da9-80b9b812f33f" />   
Truy cập vào giao diện web để xem kết quả http://192.168.164.129  
<img width="1920" height="856" alt="7" src="https://github.com/user-attachments/assets/7fe550e8-a204-40de-9080-7974279f7be0" />  
Kết quả cảnh báo khi nhiệt độ vượt ngưỡng 30 độ:  
3 Cấu hình Grafana kết nối InfluxDB  
Bước 1: Đăng nhập grafana  
Truy cập http://192.168.164.129:3002 để vào Grafana  
Đăng nhập và đổi mật khẩu (nếu cần)  
<img width="1920" height="967" alt="9" src="https://github.com/user-attachments/assets/83ab1b9f-1961-4312-8899-9fe5893f2f9d" />  
Bước 2: Thêm datasource  
Tại thanh menu bên trái, chọn Connections -> Data sources -> Add data source.  
<img width="1920" height="677" alt="10" src="https://github.com/user-attachments/assets/ee06d623-740a-47f1-92b4-14c6af40c0f5" />  
Kéo xuống dưới cùng ấn Save & test. Nếu hiện thông báo màu xanh "Data source is working" là thành công!  

<img width="1920" height="958" alt="11" src="https://github.com/user-attachments/assets/004ca3e1-3208-492c-9f99-e7ef7e39ee40" />  

Bước 3: Tạo biểu đồ  
Nhấn vào biểu tượng + ở phía trên bên phải, chọn New Dashboard -> add Panel -> Configure Visualization  

<img width="1920" height="927" alt="12" src="https://github.com/user-attachments/assets/2af2f913-a80e-4889-8f2c-f9a8426f232d" />  
Bước 4: Lấy link nhúng Iframe  
Tại ô biểu đồ vừa vẽ, góc trên cùng bên phải của khung biểu đồ đó -> Xuất hiện dấu 3 chấm -> Chọn Share -> Chọn thẻ Embed.  
Copy đoạn link trong thuộc tính src="..." (đổi localhost thành 192.168.164.129) rồi dán vào file index.html của Nginx.  

<img width="1920" height="905" alt="13" src="https://github.com/user-attachments/assets/4c7c9788-0878-4844-9375-7bb2eda7fe94" />  

<img width="945" height="443" alt="14" src="https://github.com/user-attachments/assets/7f04c91a-a4b9-4876-9521-4baaf5835d8b" />   

<img width="1337" height="228" alt="15" src="https://github.com/user-attachments/assets/7f8187c5-c58c-4ce8-b8d5-6ef9dea888bb" /> 

4.Kết quả  
<img width="1920" height="976" alt="16" src="https://github.com/user-attachments/assets/7a1edff8-aca6-4bf0-a2b4-43ac1b0afb4e" />   

## Đóng gói và khôi phục hệ thống  
Bước 1: Xuất tất cả các Container ra file nén:  
```yaml
# Đứng tại thư mục người dùng, tiến hành nén thư mục dự án lại   
sudo tar -czvf weather_monitor_backup.tar.gz weather-monitor/
```  
Bước 2: Xóa mọi container đang chạy để nghiệm thu  
```yaml
cd weather-monitor  
docker compose down  
image  
```
<img width="614" height="220" alt="18" src="https://github.com/user-attachments/assets/6332067c-a5fc-4dfd-930b-78cb04f5eff2" />  
Lúc này vào trình duyệt cổng 8085 sẽ sập hoàn toàn, chứng minh hệ thống đã được dọn dẹp sạch  
<img width="1920" height="841" alt="17" src="https://github.com/user-attachments/assets/bd2a721a-ff87-4f98-bf87-8113ad61888f" />  
Bước 3: Khôi phục lại từ file nén    
```yaml
# Giải nén lại thư mục    
tar -xzvf weather_monitor_backup.tar.gz  
```
```yaml
# Truy cập và khởi động lại toàn bộ hệ thống như cũ  
cd weather-monitor  
docker compose up -d  
```
Hệ thống sẽ tự động khôi phục lại trạng thái đỉnh cao ban đầu, giữ nguyên lịch sử dữ liệu cũ trong DB mà không cần cấu hình lại từ đầu!  
Bước 4: Kết quả khôi phục   
<img width="1920" height="973" alt="19" src="https://github.com/user-attachments/assets/8021af30-afea-48fc-8dc0-c57f59c5a923" />

















  









