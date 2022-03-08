# Database của AWS

- Database type (SQL/OLTP online transaction processing): RDS (Postgres, Mysql, MyRDB, MySQL Server, Oracle), Aurora
- Nosql: DynamoDB lưu trữ kiểu như file JSON, Elastic Cache (key, value pair), Neptune (Biểu đồ)
- Object store: S3 ( RDB có thể lưu trữ 400 KB data mỗi row, còn S3 có thể lưu lên đến 5TB data ), Glacier (backup/archives)
- Data warehouse = (SQL Analytics/BI): Redshift (OLAP online analytical processing), Athena
- Search: ElasticSearch (JSON) - Free text, unstructured search
- Graph: Neptune hiển thị mối quan hệ giữa các data


## RDS

- Tổng quan:
  - Cần cung cấp EC2 và EBS volume type, size
  - Hỗ trợ read replicate, multi AZ
  - Tính năng Backup/Snapshot đúng lúc
  - Quản lý và lập lịch bảo trì
  - Monitor với CloudWatch

- Hoạt động:
  - Thời gian downtime thấp khi failover, bảo trì
  - Mở rộng read replicas/EC2/restore EBS bằng manual
- Bảo mật:
  - AWS có trách nhiệm bảo mật OS
  - Setup KMS, security group, IAM policy
  - Authorizing user trong DB
  - Sử dụng SSL
- Khả năng thực tế:
  - Tính năng Multi AZ
  - Failover trong trường hợp lỗi
- Hiệu năng:
  - Phục thuộc vào EC2 instance type, EBS volume type
  - Có thể add read replicas
  - Storage auto-scaling và manual scaling instance
- Chi phí:
  - Trả mỗi giờ dựa trên tài nguyên EC2 và EBS được cung cấp

## Aurora

- Overview:
  - Tương thích với Postgres và Mysql
  - Data với 6 replicas và across 3 AZ
  - Khả năng tự sửa lỗi
  - Multi AZ, Auto scaling Read Replicas
  - Read replicas có thể Global
  - Aurora database có thể Global cho DR hoặc giảm latency
  - Auto scaling storage từ 10GB tới 128TB
  - Define EC2 instance type cho aurora instance
  - Same security/monitoring/maintenance feature như RDS
  - Aurora serverless cho workloads không dự đoán được
  - Aurora Multi-Master cho write failover
- Hoạt động: auto scaling storage
- Security: 
  - AWS chịu trách nhiệm bảo mật OS
  - Setup KMS, security group, IAM policy
  - Authorizing user trong DB
  - Sử dụng SSL
- Thực tế:
  - Multi AZ, HA, khả năng hơn RDS
  - Aurora serverless option
  - Aurora Multi-Master option
- Hiệu năng:  
  - Gấp 5 lần hiệu suất do tối ưu thiết kế
  - Lên tới 15 Read replicas (chỉ có 5 cho RDS)
- Chi phí:
  - Trả theo giờ dựa trên EC2 và Storage sử dụng, giá rẻ hơn so với Enterprise database như Oracle
  - Giống như RDS nhưng bảo trì ít hơn, linh hoạt hơn, hiệu suất cao hơn


## ElastiCache

- Overview:
  -  Quản lý bởi Redis/Memcached
  -  In-memory data store, độ trễ dưới mili giây
  -  Phải cung cấp 1 EC2 instance type
  -  Hỗ trợ Clustering (redis) và Multi AZ, Read replicas (sharding phan tán databse)
  -  Backup/Snapshot
  -  Quản lý và lập lịch bảo trì
  -  Monitor với Cloudwatch
  -  Sử dụng cho : Key/value store, thường xuyên Read, ít write, cache result cho DB queries, lưu sesion data cho website, không thể sử dụng SQL
- Hoạt động:
  - Giống như RDS
  - Bảo mật: 
  - AWS có trách nhiệm bảo mật OS
  - Setup KMS, security group, IAM policy
  - User (Redis Auth)
  - Sử dụng SSL
- Thực tế:
  - Clustering
  - Multi AZ
- Hiệu suất:
  - Độ trễ thấp dưới mili giây
  - In-memory
  - Read replicas cho sharding phân tán database
- Chi phí:
  - Giống như RDS

## DynamoDB

- Overview:
  - Công nghệ do AWS phát minh, quản lý bởi NoSQL
  - Là 1 loại serverless, cung cấp dung lượng theo nhu cầu, tự động mở rộng
  - Có thể thay thế ElastiCache giống như lưu key/value (lưu sesion data)
  - HA, Multi AZ mặc định, Read và Write decoupled tách riêng, sử dụng DAX cho read cache
  - Read có thể *eventually consistent* hoặc *strong consistent*
  - Bảo mật, xác thực, phân quyền bằng IAM
  - DynamoDB Streams tương thích với Lambda
  - Backup/restore, tính năng Global table
  - Monitor với CloudWatch
  - Có thể chỉ query trên primary key, sort key, hoặc index rất linh hoạt
  - Sử dụng cho trường hợp: Phát triển ứng dụng trên serverless (small document row nhỏ hơn 400KB), phân phối serverless cache, không có query SQL, có thể chọn số lượng transaction 
- Hoạt động:
  - Không cần vận hành, tự động mở rộng, không cần server
- Bảo mật:
  - Bảo mật đầy đủ với IAM policy
  - Mã hóa KMS
  - SSL
- Thực tế:
  - Multi AZ, Backup
- Hiệu suất:
  - Độ trễ mili giây 1 chữ số
  - DAX cho cache read
  - Hiệu suất không giảm cả khi app scale
- Chi phí:
  - Trả theo mức dung lượng sử dụng (có tể tự động scaling)

## Athena

- Overview:
  - Serverless database với ngôn ngữ SQL
  - Sử dụng để query data trong S3
  - Trả phí cho mỗi query
  - Kết quả trả ra sẽ lưu ở S3
  - Bảo mật thông qua IAM
  - Sử dụng cho SQL query, Serverless queries trên S3, phân tích log
- Hoạt động:
  - Không cần server, không cần vận hành
- Bảo mật:
  - IAM, S3 security
- Thực tế:
  - Quản lý service sử dụng Presto engine hiệu suất cao, HA
- Hiệu năng:
  - Queries mở rộng dựa trên data size
- Chi phí:
  - Dựa trên mỗi query hoặc mỗi TB data scanned

## Redshift

- Overview:
  - Xây dựng dựa trên Postgres, nhưng không sử dụng cho OLTP
  - Redshift là OLAP - online analytical processing (analytics và data warehouse)
  - Hiệu suất gấp 10 lần các data warehouse khác, mở rộng tới hàng PB data
  - Lưu trữ dữ liệu dạng cột
  - Sử dụng Massively Parallel Query Execution (MPP)
  - Trả phí dựa trên mỗi instance được cung cấp
  - Có SQL interface để queries
  - BI tool như AWS Quicksight hoặc Tableau tích hợp với Redshift
- Hoạt động:
  - Data load từ S3, DynamoDB, DMS, và các DB khác
  - Kích thước từ 1 tới 128 node, lên đế 128TB cho mỗi node
  - Leader node: cho query tổng hợp kết quả
  - Compute node: sử dụng để thực hiện query và gửi kết quả tới leader
  - Redshift: sử dụng query trực tiếp tới S3
  - Backup và Restore, security VPC/IAM/KMS, monitoring, copy/upload command thông qua VPC
- Snapshot và DR
  - Redshift không có Multi-AZ mode
  - Snapshot là backup 1 cluster đúng thời điểm và lưu internal tại S3
  - Snapshot incremental (lưu những thay đổi)
  - Có thể restore 1 snapshot vào 1 cluster mới
  - Tự động: mỗi 8 tiếng và mỗi 5GB sẽ thực hiện snapshot
  - Snapshot được giữ lại cho đến khi thực hiện xóa
  - Có thể cấu hình Redshift tự động copy snapshot (auto/manual)  của 1 cluster tới 1 cluster khác region
- Loading data vào Redshift

  <img src="https://i.imgur.com/JaimDbP.png">
  
  - Kinesis data firehose thu thập data từ các nguồn khác nhau và sau đó gửi tới Redshift , và để làm việc này nó cần write data vào S3 bucket trước và sử dụng S3 copy command để load data vào Redshift
  - Copy data từ bucket vào Redshift thông qua IAM role, data copy thông qua public hoặc private VPC network
  - Insert data Sử dụng JDBC driver để insert data vào Redshift cluster
- Redshift spectrum: 
  - Query data trong S3 mà không cần load
  - Phải có 1 Redshift cluster để start query
  - Mỗi khi query submit tới 5000 Redshift spectrum node sau đó sẽ query data trong S3

  <img src="https://i.imgur.com/d1Kxday.png">
  
- Hoạt động : giống RDS
- Bảo mật:
  - IAM, VPC, KMS, SSL
- Thực tế: 
  - Tự động sửa lỗi, cross-region snapshot copy
- Hiệu năng:
  - Gấp 10 lần hiệu suất các data warehouse khác, nén
- Chi phí: 
  - Trả chi phí mỗi node được cung cấp
- Redshift = Analytics/BI/Data warehouse

## AWS Glue

- Overview:
  - Quản lý giải nén, transform and load (ETL) serrvice
  - Hữu ích để chuẩn bị và vận chuyển data cho analytics
  - serverless service

  <img src="https://i.imgur.com/S6Yx2JL.png">

- Glue Data Catalog

  <img src="https://i.imgur.com/ZtZxVOL.png">
  
  - AWS Glue Data Crawler thu thập data từ các DB và write metadata vào AWS Glue Data Catalog thành các database (table, data type, column name...) và Glue Data Catalog có thể sử dụng bởi Glue job thực hiện ETL chắc chắn hoàn thành chính xác.
  - Glue data catalog có thể tận dụng cho Athena, Redshift Spectrum, EMR để analytics
  
## Neptune

- Overview
  - Quản lý đồ thị database 
  - Xem relationship data
  - Knowledge graph (wikipedia)
  - Mạng xã hội: user friend với user, reply to comment on post user và like user comment
  - Tất cả các relationship được link với nhau sử dụng 1 đồ thị lớn
  - HA across 3 AZ, lên tới 15 read replica
  - Recovery tại thời điểm hiện tại, backup tới Amazon S3
  - Hỗ trợ mã hóa KMS với REST + HTTPS
- Hoạt động:
  - Tương tự RDS
- Bảo mật:
  - IAM,VPC, KMS, SSL + IAM authen
- Thực tế
  - Multi AZ, clustering
- Hiệu năng:
  - Phù hợp cho đồ thị, cluster cải thiện hiệu suất
- Chi phí:
  - Tính theo mỗi node được cung cấp (giống RDS)

## Elastic Search

- Overview:
  - Trong DynamoDB có thể tìm primary key và index
  - Search mọi trường
  - Có thể tạo cluster
  - Built-in tích hợp: Kinesis data firehose, AWS IoT, Amazon CloudWatch Logs cho data
  - Bảo mật thông qua Cognito và IAM, mã hóa KMS, SSL và VPC
  - Kết hợp với Kibana và logstash thành ELK stack
- Hoạt động:
  - Giống RDS
- Bảo mật:
  - Cognito, IAM, VPC, KMS, SSL
- Thực tế:
  - Multi-AZ, clustering
- Hiệu suất:
  - Trên cơ sở opensource có thể scale tới hàng PB data
- Chi phí:
  - Dựa trên mỗi node được cung cấp giống RDS
