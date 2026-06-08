# APP MONITOR + ALERT DATA REALTIME
MÔN HỌC: PHÁT TRIỂN ỨNG DỤNG TRÊN THIẾT BỊ DI ĐỘNG   
Môn học: Phát triển ứng dụng với mã nguồn mở   
Họ và tên: Nguyễn Thị Kim Huệ  
MSSV: K225480106026  
YÊU CẦU BÀI TẬP  
LÝ THUYẾT  
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
 
BÀI LÀM  
PHẦN 1: LÝ THUYẾT  
1. Docker là gì?
   Docker là một nền tảng phần mềm giúp bạn building, deploying và running ứng dụng dễ dàng hơn bằng cách sử dụng các containers (trên nền tảng ảo hóa).
  Docker đóng gói phần mềm thành các container tiêu chuẩn hóa, chứa đựng tất cả những thứ cần thiết để phần mềm hoạt động như thư viện, công cụ hệ thống, mã nguồn và thời gian chạy. Khi cần deploy app lên bất kỳ server nào, bạn chỉ cần run container của Docker thì app của bạn sẽ được khởi chạy ngay lập tức.  

Khi sử dụng Docker, bạn có thể dễ dàng triển khai và mở rộng quy mô ứng dụng trong bất kỳ môi trường nào, đồng thời đảm bảo rằng mã nguồn của bạn sẽ luôn chạy được một cách ổn định.    
2. Các từ khóa thường dùng trong docker-compose.yml  
<img width="782" height="298" alt="image" src="https://github.com/user-attachments/assets/685a2db6-9519-4758-8826-70b34e4419f6" />  

2.1 Các keyword mô tả service  
`image`  

Chỉ định image có sẵn (từ Docker Hub hoặc local) để chạy container.  

```services:   
  database:   
   image: mariadb:latest   # dùng image mariadb phiên bản mới nhất  ` ` `
   


