# AWS storage

### AWS Snow

- Time to transfer

|     |100Mbps |1 Gbps  |10 Gbps |
|-----|--------|--------|--------|
|10TB |12 days |30 hours|3hours  |
|100TB|124 days|12 days |30 hours|
|1PB  |3 years |124 days|12 days |


- Snowball Edge

<img src="https://i.imgur.com/Z4GcVtj.png">

  - Giải pháp cho physical data transfer: di chuyển hàng TB hoặc PB data vào hoặc ra AWS
  - Tương thích với block và object
  - Snowball Edge Storage Optimized

- AWS Snow family for data migration

<img src="https://i.imgur.com/U171Dcl.png">

- Snowcone (smaller)
  - 2 CPUs, 4 GB RAM, wired or wireless access
  - USA-C sử dụng dây hoặc pin
- Snowball Edge Compute Optimized
  - 52 vCPUs, 208 GB RAM
  - Optional GPU
  - 42 TB storage HĐ cho block volume và s3
- Snowball Edge Storage Optimized
  - Up to 40 vCPUs, 80 GB RAM 
  - Object storage cluster
  - 80 TB storage HDD cho block volume và s3

- Có thể chạy EC2 instance với AWS Lambda functions ( sử dụng AWS IoT greengrass)
- Lựa chọn deploy dài hạn: được giảm giá từ 1 - 3 năm
- AWS OpsHub: Là 1 phần mềm cài trên máy tính để quản lý Snow Family Device
  - Unlock và config single hoặc cluster devices
  - Truyền files
  - Khởi tạo hoặc quản lý instance đang chạy trên Snow Family Devices
  - Khởi tạo AWS service trên devices (EC2, DataSync, NFS)
  
  <img src="https://i.imgur.com/daQHwcH.png">

- AWS Snow cho phép run theo object storage hoặc EC2 instance
- Snowball không thể import trực tiếp vào Glacier mà phải dùng S3 trước kết hợp với 1 S3 lifecycloe policy để vận chuyển object vào trong AWS Glacier

### AWS  FSX

- Là 3 party file system hiệu suất cao trên AWS
- Full manage services
- Có 3 loại là Lustre, window file server, Netapp ONTAP
- AWS FSx window file server
  - FSx quản lý Window file system share drive
  - Hỗ trợ SMB protocol và NTFS
  - Tích hợp Active directory, ACLs, User quotas
  - Build trên SSD tốc độ đạt tới 10 GB/s, hàng triệu IOPs, 100 PB storage
  - Có thể cấu hình tới Multi AZ
  - Data backup hằng ngày tới S3
- FSx for Lustre: là 1 dạng dữ liệu phân tán song song nguồn gốc từ Linux và cluster
  - Sử dụng cho Machine Learning, hiệu suất tính toán cao , video processing, financial medeling, electronic design automation
  - Scale up to 100GB/s, hàng triệu IOPs, độ trễ thấp ms
  - tích hợp với S3, có thể đọc S3 như 1 file system
  - có thể ghi đầu ra tính toán trở lại S3 thông qua FSx
  - Có thể sử dụng cho On-premise servers
- Lựa chọn Deploy FSx file system
  - Scrat File System
    -  Lưu trữ tạm thời
    -  Data không replicate
    -  High burst (6x faster, 200MBps mỗi TB)
    -  Tối ưu chi phí
  - Persistent File System
    - Lưu trữ dài hạn
    - Data được replicate cùng AZ
    - Thay thế file lỗi trong vài phút
    - Dữ liệu quan trọng


<img src="https://i.imgur.com/0rFzvEQ.png">

### AWS Storage Gateway

Block|File|Object
-----|----|------
Amazon EBS|Amazon EFS|Amazon S3
EC2 Instance store|Amazon FSx|Amazon Glacier

- 3 storage gateway
  - File Gateway
    - S3 bucket được cấu hình có thể truy cập sử dụng NDS hoặc SMB protocol
    - Hỗ trợ S3 standard, S3 IA, S3 One Zone IA
    - Truy cập bucket sử dụng IAM roles cho mỗi File gateway
    - File sử dụng gần nhất sẽ được cache trên File gateway
    - Có thể mount trên nhiều server
    - Tích hợp với Active Directory (AD) cho xác thực user

    <img src="https://i.imgur.com/iWZI5eQ.png">
    
  - Volume Gateway
    - Block storage sử dụng iCSI protocol được hỗ trợ bởi S3
    - Hỗ trợ EBS snapshot có thể restore on-premise volume
    - Cache volume: độ trễ truy cập thấp cho hầu hết data
    - Volume lưu trữ được lập lịch backup tới S3

    <img src="https://i.imgur.com/2lG303u.png">
    
  - Tape Gateway
    - Sử dụng Tape gateway để backup data lên cloud
    - Virtual Tape Library (VTL) được hỗ trợ bởi S3 và Glacier
    - Backup data sử dụng tape-base và iSCSI interface

    <img src="https://i.imgur.com/N8aad2o.png">
    
- Trong trường hợp không có server ảo trên amazon thì người dùng có thể mua Hardware Appliance trên Amazon.com để cài đặt nó trên server giống như 1 gateway

- FSx File Gate Way:
  - Native access tới AWS FSx cho Window File Server
  - Local cache cho data thường xuyên truy cập
  - Native với SMB, NTFS, Active Directory
  - Hữu ích cho việc chia sẻ file và home directory
  
  <img src="https://i.imgur.com/eb6i9Ck.png">

- AWS Transfer Family
  - Sử dụng FTP, TFTP, FTPS để truyền dữ liệu
  - Quản lý bởi hạ tầng, có thể mở rộng, HA (multi AZ) và đáng tin cậy
  - Chi phí tính theo giờ + data transfer (GB)
  - Lưu trữ và quản lý thông tin đăng nhập của user trong service
  - Tích hợp với hệ thống xác thực như LDAP, AD, Okta, Amazon Cognito
  - Chia sẻ file, public datasets, CRM, ERP...
  
  <img src="https://i.imgur.com/8HOnkA4.png">


### Bảng so sánh các AWS Storage 

<img src="https://i.imgur.com/4G7IhbM.png">



