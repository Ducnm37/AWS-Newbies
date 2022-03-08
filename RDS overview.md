# Renational Database Service

- Postgre
- Mysql
- Mariadb
- Oracle
- Microsoft SQL Server
- Aurora (AWS Proprietary database)

```
Database port:

PostgreSQL: 5432
MySQL: 3306
Oracle RDS: 1521
MSSQL Server: 1433
MariaDB: 3306 (same as MySQL)
Aurora: 5432 (if PostgreSQL compatible) or 3306 (if MySQL compatible)
```

Read replica sử dụng synchronous replica đồng bộ dữ liệu (có nghĩa là mỗi khi ứng dụng ghi là gần như ngay lập tức data sẽ replicate sang bản dự phòng)

multi AZ sử dụng asynchronous replica không đồng bộ dữ liệu (có nghĩa sau khi ứng dụng ghi xong dữ liệu vào database primary thì dữ liệu mới được replica sang database standby, trong trường hợp failover khả năng sẽ mất dữ liệu vào các lần ghi cuối)

## RDS

- RDS là một dịch vụ quản lý database
  - Tự động backup
    - Daily full backup db
    - Backup Transaction Log mỗi 5 phút 1 lần bởi RDS và restore mọi lúc
    - Bản backup sẽ được giữ trong 7 ngày có tăng lên tới 35 ngày
  - Snapshot database
  - Tự động cung cấp bản vá lỗi
  - Giám sát database
  - Đọc bản sao để cải thiện hiệu suất 
  - Multi AZ setup DR (disaster recovery)
  - Không thể SSH vào RDS instance

- RDS auto scaling storage
  - Tự động tăng storage khi hết dung lượng
  - Có thể set ngưỡng tối đa cho database
  - Automatic modify storage khi:
    - Khi ngưỡng storage dưới 10%
    - Low-storage kéo ít nhất 5 phút
    - Qua 6 giờ kể từ lần sửa đổi cuối cùng
  - Hữu ích cho các ứng dụng không thể dự đoán
  - Hỗ trợ tất cả RDS database engine 
- Rds read replicas
  - replica database only read and Main database can read and write
  - Up to 5 read replicas
  - Replication là ASYNC
  - Replicas có thể đặt làm database riêng
  - Lưu lượng data replica từ db ở cùng region (có thể khác AZ) sẽ không tính phí, nếu khác region sẽ bị tính phí

  <img src="https://i.imgur.com/oWgTY9q.png">

- RDS multi-AZ (DR)
  - App từ bên ngoài sẽ kết nối với database thông qua DNS name, và mỗi khi node master chết thì tự động dữ liệu sẽ failover sang node standby và standby database sẽ trở thành master, qua có các app không cần phải chỉnh sửa cấu hình mà sẽ chỉ cần kết nối qua DNS name

  <img src="https://i.imgur.com/Fbo3rcJ.png">


  - Chuyển từ single AZ sang multi AZ sẽ không downtime mà chỉ cần click "modify" cho database
  - Data base mới sẽ được restore từ snapshot và dữ liệu từ master database cũng sẽ được đồng bộ sang database mới 
  <img src="https://i.imgur.com/hbYdvzB.png">

------------------------------------------------------------------------------------------------------------------
- RDS security:
  - Sử dụng KMS (key management service) để replica data và dùng AES-256 để mã hóa
  - Có thể enable Transparent Data Encryption (TDE) cho Oracle và SQL server để mã hóa cho database
  - in-flight encryption:
    - Dùng SSL để mã hóa data tới RDS trong flight khi client gửi data đến databse
    - Để thi hành SSL trên postgres thực hiện lệnh: rds.force_ssl=1 (thực hiện trên AWS RDS console phần parameter group)
    - Để thi hành SSL trên mysql thực hiện lệnh: grant usage on *.* to 'mysqluser'@'%' require ssl;
- RDS encrypted:
  - Nếu database encrypted thì bản snapshot của nó sẽ encrypted và ngược lại nếu database không encrypted thì snapshot cũng không encrypted
  - Khi có một bản snapshot database un-encrypted ta chỉ cần enable encrypted nó lên sau đó restore bản snapshot đã encrypted và ta sẽ nhận được một database encrypted
- IAM kiểm soát quyền trên database, nó cho phép user nào được hoặc không được phép tác động vào database
- RDS - IAM authentication
  - Sử dụng cho MySQL và PostgreSQL
  - Không cần password authentication mà xác thực qua token thông qua IAM và RDS API
  - Authen token có thời gian sống là 15 phút 

  <img src="https://i.imgur.com/pdAKctV.png">
  
- AMAZON Aurora là (AWS cloud optimized) công nghệ độc quyền của Amazon
  - Postgre và mysql được hỗ trợ cho Aurora
  - Aurora cải thiện gấp 5 lần cho mysql và gấp 3 lần cho postgre trên RDS
  - Aurora storage tự động tăng trưởng từ 10GB có thể lên đến 128 TB
  - Aurora có thể có 15 bản replicas trong khi mysql chỉ có 5 bản replicas  và tiến trình replicas của Aurora nhanh hơn
  - Failover trên Aurora là ngay lập tức( thấp hơn 30s) từ multi AZ tới RDS, HA native
  - Aurora hoạt động hiệu quả hơn, và cũng đắt hơn RDS 20%
  - Aurora lưu 6 bản copies data qua 3 AZ
    - 4 bản copy cho write
    - 3 bản copy cho read
    - Tự động khôi phục (self healing) peer to peer replication
    - storage có thể vượt qua 100 volume
- Aurora hỗ trợ replicate qua các region khác và chỉ cần 1 master còn các node khác là read
- Aurora DB Cluster có thể tự động mở rộng từ 10GB tới 64TB
- Aurora chỉ cần 1 node writer endpoint các node khác sẽ là reader endpoint, do cơ chế failover rất nhanh lên khi writer node fail thì 1 node reader sẽ lên làm writer, và các node writer được load balancing khi request connect

<img src="https://i.imgur.com/JRRuBJt.png">

- Aurora mã hóa sử dụng KMS, cà các bản replicas, snapshot hay backup đều tự động được mã hóa
- Aurora sử dụng SSL để mã hóa đường truyền và cũng có thể sử dụng IAM auth token
- Tier Priority: cái này là độ ưu tiên cho node reader để trở thành master khi node master fail, tier càng nhỏ thì độ ưu tiên trở thành master càng cao
- Aurora Provisioned Mode: Phù hợp cho các ứng dụng chạy liên tục và lâu dài
- Aurora Serverless Mode: phù hợp cho các ứng dụng chạy tạm thời, không liên tục và ngắn hạn
- Aurora hỗ trợ service SageMaker (machine learning)
- Aurora hỗ trợ service Comprehend (analyze Unstructured Text)
- Aurora Global Databases cho phép có các bản replica database ở các region khác  (max 5 secondary regions)

<img src="https://i.imgur.com/1a9NjXc.png">

