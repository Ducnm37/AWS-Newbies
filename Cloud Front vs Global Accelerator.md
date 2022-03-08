# Cloud Front

<img src="https://i.imgur.com/YAKt6Hj.png">

- Giống như Content delivery network (CDN)
- Cải thiện hiệu xuất đọc và caching content ở biên
- 216 khu vực biên trên toàn cầu
- Ngăn chặn DDos tích hợp bảo vệ và AWS web apps firewall
- Có thể expose external HTTPS giao tiếp được với internal HTTPS backends nếu cần mã hóa lưu lượng
- File cos thể cache cho TTL khoảng 1 ngày
- Sử dụng Signed Cookies để truy cập nhiều file
- S3 bucket
  - Phân phối file tại local và caching chúng ở biên (Edge)
  - Bảo mật với Cloudfront Origin Access Identity (OAI)
  - Cloudfront có thể sử dụng giống như ingress upload file lên S3

- Tạo Cloudfront cho S3
  - bucket ở đây là **athena-ducnm5**


  <img src="https://i.imgur.com/0J1Y71x.png">
  
  
  - Tạo cloudfront


  <img src="https://i.imgur.com/c51i0np.png">


  - Chọn bucket
  
  
  <img src="https://i.imgur.com/MC5v4h8.png">
  

  - Tạo rule OAI để có quyền truy cập vào resource của bucket, ban đầu bucket policy trong mục **permission** của bucket không có gì, sau khi tạo vào add OAI sẽ có
  
  <img src="https://i.imgur.com/sBvgrJ3.png">
  
  - Policy của bucket ban đầu trống không và sau khi tạo OAI trên cloudfront
  
  <img src="https://i.imgur.com/4PgJbZ8.png">

  - Chỉ định file mặc định khi load vào link Cloudfront (nếu ko đặt thì mặc định sẽ là root của bucket )

  <img src="https://i.imgur.com/Do1X6vR.png">
  
  - Chờ Cloudfront khởi tạo xong click vào sẽ có link truy cập
  
  <img src="https://i.imgur.com/Nenc7x7.png">

  <img src="https://i.imgur.com/bcM1fgs.png">
  
  <img src="https://i.imgur.com/gqDmNGD.png">

  - Để xóa cloudfront thì cần disable cloudfront sau đó chờ quá trình deploy xong mới xóa được

  <img src="https://i.imgur.com/Aavrh3P.png">

- Cloudfront Signed URL
  - Cho phép truy cập đường dẫn mà không ảnh hưởng tới thư mục gốc
  - Chỉ có root mới được phép key-pair trên tài khoản đó
  - Có thể lọc IP, path, date, expire 
  - Có thể tận dụng tính năng cache
  
  <img src="https://i.imgur.com/j5ZGkv4.png">

- S3 Pre-Signed URL
  - Đưa ra 1 yêu cầu cho người nào đó Pre-Signed vào URL
  - Sử dụng AIM key để đăng ký IAM principal
  - Thời gian hoạt động bị giới hạn

  <img src="https://i.imgur.com/NEH0hz4.png">
  
- Giá thuê dịch vụ Cloudfront 

<img src="https://i.imgur.com/fE4XNEu.png">

- Cloudfront field level encryption

<img src="https://i.imgur.com/atYcyOZ.jpg">

----------------------------------------------------------------------
----------------------------------------------------------------------


# Glocal Accelerator

- Làm việc với ElasticIP, EC2, Application loadbalancer, Network Loadbalancer, Public hoặc private
- Sử dụng anycast IP: Anycast IP là các server đều có cùng 1 ip giống nhau client sẽ truy cập vào server nào gần nó nhất nhằm truy cập nhanh chóng và giảm độ trễ đường truyền
- Anycast IP gửi trực tiếp request đến các Edge location và Edge location gửi request đến Apps
- Cải thiện hiệu suất
  - Định tuyến thông minh với độ trễ nhỏ nhất và fast region failover
  - Client cache hoạt động ổn định vì IP không thay đổi
- Health check
  - Glocal Accelerator thực hiện health check cho apps
  - Failover ít hơn 1 phút cho unhealthy
- Security
  - Chỉ 2 external IP cần cho whitelisted
  - Bảo vệ khỏi DDos bở AWS Shield

----------------------------------------------------------------------------
----------------------------------------------------------------------------

# So sánh Cloud Front và Glocal Accelerator

- Giống nhau:
  - Đều sử dụng global network và Edge location
  - Cả 2 service đều được bảo vệ bởi AWS Shield và DDos protection

- Khác nhau
  - Cloudfront:
    - Cải thiện hiệu suất bằng việc có thể caching content (images, video)
    - Dynamic content ( giống như API acceleration và dynamic site delivery)
    - Content được cung cấp ở Edge 

  - Global Accelerator
    - Cải thiện hiệu suất cho nhiều loại ứng dụng qua TCP hoặc UDP
    - Packets được ủy quyền tại Edge location tới 1 hoặc nhiều App đang chạy ở các region
    - Phù hợp với các giao thức ko phải HTTP như UDP (gaming), IoT (MQTT) hoặc Voice over IP
    - Phù hợp với HTTP trong trường hợp cần static IP global
    - Phù hợp với HTTP trong các hệ thống deterministic (hệ thống ko thay đổi) và fast region failover
