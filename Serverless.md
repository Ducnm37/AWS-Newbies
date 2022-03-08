# Giới thiệu về các dịch vụ liên quan tới serverless

- Serverless là một nền tảng cho phép chạy ứng dụng mà không cần quản lý về mặt vật lý hay hạ tầng, nó giống như 1 môi trường dùng để chạy các ứng dụng
- Các dịch vụ chạy trên serverless của AWS
  - Lambda
  - DynamoDB
  - Cognito
  - API Gateway
  - S3
  - SNS, SQS
  - Kinesis Data Firehose
  - Aurora Serverless
  - Step Functions
  - Fargate

---------------------------------------

## Lambda

- Virtual function (không có server để quản lý)
- Giới hạn bởi thời gian (tối đa 15 phút cho lần khởi chạy thực thi)
- Tự động mở rộng 
- Chi phí sẽ tính theo số request và thời gian tính toán
  - Gói free tier là  1,000,000 Lambda request và 400,000 GB thời gian tính toán
- Có thể monitor với Cloudwatch 
- Tích hợp với nhiều ngôn ngữ lập trình
  - Nodejs (JavaScript)
  - Python
  - Java (Java 8 Compatible)
  - C# (.Net Core)
  - Ruby
  - Custom Runtime API 
  - Lambda Container Image
    - Container Image phải thực hiênj lambda runetime API
    - ECS / Fargate được ưu tiên chạy các docker images tùy ý 
- Tích hợp với nhiều services
- Dễ dàng lấy được nhiều resource cho mỗi function (lên đến 10 GB RAM)
- Tăng RAM cũng sẽ đồng nghĩa với việc cải thiện CPU và network
- API gateway để tạo ra 1 REST API sẽ gọi tới Lambda functiopn
- Kinesis sử dụng Lambda để truyền tải 1 số dữ liệu 
- DynamoDB sẽ sử dụng để tạo 1 số trigger khi database gặp sự cố sẽ trigger tới 1 Lambda function
- S3 sẽ được Lambda function trigger mọi lúc ví dụ như 1 file được tạo trong S3
- CloudFront là Lambda Edge
- CloudWatch Events hoặc EventBridge để phản ứng và đưa ra yêu cầu thực thi đối với sự thay đổi trong hạ tầng sử dụng Lambda Function
- Cloudwatch Logs dùng để bắn logs
- SNS dùng để thông báo topic
- SQS thực hiện truyền messages
- Cognito phản ứng với các lệnh thực thi, ví dụ như user login vào database
- Serverless Cron Job được chạy trên EC2
- Giá Lambda function là 0,02$ cho 1 triệu request (với 1 triệu request đầu free) 
- 400000 giây thực thi cho Lambda function với 1GB RAM
- 3200000 giây thực thi cho Lambda function với 128MB RAM
- 1$ cho 600000 GB thời gian cho tính toán
- Số lượng execute cho phép đồng thời là 1000 và có thể tăng lên
- Disk capacity trong function container mục /tmp là 512 MB
- Lambda function deploy size (nén .zip) là 50MB
- kích thước giải nén (code + dependencies) là 250 MB
- Có thể sử dụng /tmp để load các file khác khi startup
- Size enviroment variables là 4 KB

### Lambda Edge

- Deplot CDN sử dụng Cloudfront
- run AWS Lambda global
- Deploy Lambda function với Cloudfront CDN
  - Xây dựng các ứng dụng responsive
  - Lambda được triển khai cho global
  - Customize cho CDN content
- Có thể sử dụng Lambda để thay đổi Cloudfront request và response

  <img src="https://i.imgur.com/ZFtBsOl.png">

  - Thay đổi viewer request,origin request, origin response, viewer response
  - Có thể generate response để viewer mà không cần gửi request tới origin
- Global application

  <img src="https://i.imgur.com/NCk62xO.png">

  - User gửi API request tới Cloudfront, Cloudfront trigger Lambda Edge running global, Lambda Edge query DynamoDB (global database)
- Sử dụng Lambda Edge cho các trường hợp
  - Dynamic Web Application tại Edge
  - Search Engine Optimization (SEO)
  - Intelligently Route Across Origin và Data center
  - Bot Mitigation tại Edge
  - Real-time Image Transformation
  - A/B Testing
  - User Authentication và Authorization
  - User Prioritization
  - User Tracking và Analytics

--------------------------------------

## Amazon DynamoDB

- Quản lý đầy đủ, High Availability với replication across multiple AZ
- NoSQL database
- Scale lớn, distributed databse
- Hàng triệu request mỗi giây, có thể có hàng nghìn tỉ row, có thể có 100TB storage
- Hiệu suất nhanh và ổn định (độ trễ thấp và retrieval)
- Tích hợp với IAM cho security và xác thực phân quyền
- có thể enable event driven programming với DynamoDB Stream
- DynamoDB được làm từ các Bảng
  - Mỗi bảng chứ 1 Primary Key (phải được quyết định bởi thời gian tạo)
  - Mỗi bảng có thể có 1 row vô hạn
  - Mỗi items là 1 thuộc tính attribute (có thể add thêm mọi lúc hoặc có thể là null)
  - Max item size là 400 KB
  - Data type hỗ trợ
    - Scalar Type - String, number, binary, boolean, null
    - Document type - List, Map
    - Set Types - String Set, Number Set, Binary Set 

<img src="https://i.imgur.com/qFdwnnq.png">

- Quản lý kích thước tables (read/write throughput)
- Provisioned Mode (default)
  - Chỉ định số lượng read/write mỗi giây
  - Cần tính toán kích thước trước
  - Chi phí tính theo đơn vị Read Capacity (RCU) và Write Capacity (WCU)
  - Có thể thêm auto-scaling mode cho RCU và WCU
- On-demand Mode
  - Tự động scale read/write tăng hoặc giảm theo workload
  - Không cần tính toán capacity trước
  - Chi phí tínhg theo dung lượn sử dụng

### DynamoDB Accelerator (DAX)

- in-memory Cache cho  DynamoDB
- Độ trễ thấp đơn bị microsecond
- Tương thích với DynamoDB API
- Giải quyết vấn đề nghẽn read bởi cache
- 5 minutes TTL cho cache (default) và sẽ refresh 

<img src="https://i.imgur.com/XIw0uFo.png">

- Sự khác nhau giữa Elastic Cache và DynamoDB

<img src="https://i.imgur.com/7NHeEZT.png">

- DynamoDB Stream có thể create/update/delete  trong 1 table
- Stream record có thể:
  - Gửi tới Kinesis Data Streams
  - Read bởi Lambda, Kinesis Client Library Application
- Data tồn tại lên tới 24 giờ
- Trường hợp sử dụng
  - Phản ứng với sự thay đổi tại thời gian thực
  - Analytics
  - insert vào derivative table
  - insert vào ElasticSearch
  - Triển khai replicate cross region
  
  <img src="https://i.imgur.com/BLjP5jd.png">
  
  - DynamoDB Global Table
  
    <img src="https://i.imgur.com/dvFLuHd.png">
  
    - Active - Active replicate
    - Application có thể read và write vào table trong mọi region
    - Phải enable DynamoDB Stream giống như 1 điều kiện tiên quyết
    - Truy cập với độ trễ thấp ở multi region

- DynamoDB Time To Live (TTL)
  - Tự động xóa item sau khi hết hạn timestamp
  - Khi hết hạn các items sẽ được gán flag expire

  <img src="https://i.imgur.com/4IqXqwt.png">.
  
- DynamoDB indexes
  - Global Secondary indexes  (GSI) và Local Secondary indexes (LSI)
  - Cho phép query các thuộc tính khác với Primary key chỉ cần tạo indexes on top table thay thì query primary key
  
  <img src="https://i.imgur.com/4c4j53Q.png">
  
- DynamoDB Transaction
  - 1 transaction có thể write vào multi table 
  
  <img src="https://i.imgur.com/FjVrFLV.png">
  
--------------------------------------------------
 
## AWS API Gateway

<img src="https://i.imgur.com/jGE0PPT.png">

- Lambda + API gateway không có hạ tầng quản lý
- Hỗ trợ websocket protocol
- Có thể real-time streaming thông qua API gateway
- API version (v1, v2...)
- Xử lý môi trường khác nhau (dev, test, prod...)
- Xử lý bảo mật (authentication, authorization)
- Tạo API key xử lý điều tiết request
-  Sử dụng Swagger/Open API import nhanh chóng xác định APIs
-  Transform và xác thực request và response
-  Generate chỉ định SDK và API 
- Cache API response
- Lambda Function
  - Gọi tới lambdafunction
  - Dễ dàng expose REST API lưu bởi Lambda
- HTTP
  - Expose HTTP endpoint trong backend
  - Add rate limiting, caching, user authentication, API key...
- AWS service
  - Expose mọi AWS API thông qua API gateway
  - Start 1 AWS Step Function workflow, post 1 message tới SQS fron API gateway
  - Add authentication, deplay public API, rate control...
- Có 3 cách deploy API
  - Edge-Optimized (default): cho global client
    - Request được định tuyến thông qua Cloudfront Edge location (cải thiện độ trễ)
    - API gateway còn sống chỉ trong 1 region nơi được tạo ra
  - Regional
    - Cho client cùng region
    - Dùng cho platform distribution như tích hợp với Cloudfront (kiểm soát caching và distribute)
  - Private
    - Có thể sử dụng truy cập từ VPC sử dụng interface VPC endpoint (ENI)
    - Sử dụng 1 resource policy để định nghĩa việc truy cập resource

- API Gateway Security 
  -  IAM permission sử dụng Signature v4 cho IAM credential trong header, client call tới API gateway với Sig v4 và IAM policy verify và chuyển tới backend
  
  <img src="https://i.imgur.com/rxCYkcq.png">
  
- Lambda authorizer
  - Sử dụng Lambda để xác thực token trong header là pass hay chưa
  - Option cache kết quả xác thực
  - Hỗ trợ sử dụng OAuth/SAML/ hỗ trợ xác thực bên thứ 3
  - Lambda phải return 1 IAM policy cho user
  
  <img src="https://i.imgur.com/GKwSlPi.png">

- Cognito quản lý lifecycle user
- API gateway tự động xác minh danh tính từ AWS Cognito
- Cognito chỉ giúp authentication, không hỗ trợ authorization

<img src="https://i.imgur.com/qXKTQlB.png">

- Tổng kết

<img src="https://i.imgur.com/IPr4x9N.png">

-------------------------------------------------

## AWS Cognito

- Quản lý nhóm người dùng, nhóm nhận dạng, đăng ký, đăng nhập cho dịch vụ của AWS
- Cognito user pool
  - Hàm chức năng đăng nhập cho app user 
  - Kết hợp với API gateway để xác thực
  - Tạo 1 serverless database của user cho mobile app
  - Simple login: username, email, password
  - Có thể xác thực email, phone number, thêm MFA
  - Có thể enable xác thực danh tính liên hợp từ facebook, google, SAML (login bằng facebook, google...)
  - Gửi lại 1 Json web token để verify ai đó
  
  <img src="https://i.imgur.com/4CQSKB3.png">
  
  
- Cognito Identity pool
  - Cung cấp AWS credential tới user để có thể truy cập trưc tiếp tới resource AWS
  - Tích hợp với Conito user pool giống như 1 nhà cung cấp danh tính

- Conito Sync
  - Đồng bộ dữ liệu từ các thiết bị tới Cogito (mọi platform, ios, androi...)
  - Có thể thay thế bằng Appsync
  - Lưu trữ cấu hình, trạng thái của app
  - Đồng bộ khi online
  - Cần federate identity pool trong Conito, không cần user pool
  - Lưu data trong dataset (upto 1MB)
  - Lên tới 20 dataset đồng bộ hóa

- Cognito federate identity pool (xác thực danh tính liên hợp)
  - Cho phép truy cập trực tiếp tới AWS resource từ client
  - login với federate identity pool để lấy AWS credential temporary tạm thời từ federate identity pool, credential xác định IAM policy và bắt đầu phân quyền (ví dụ truy cập và write vào S3 bucket thông qua facebook)
  
  <img src="https://i.imgur.com/DlE0epZ.png">
  
  
--------------------------------------------------------------

## Tổng kết

- AWS SAM (serverless application model)
- Framework cho developing và deploy serverless application
- Tất cả config là YAML code
  - Lambda function
  - DynamoDB tables
  - API Gateway
  - Conito User Pool

- SAM có thể giúp run Lambda, API Gateway, DynamoDB locally
- SAM có thể sử dụng CodeDeploy để deploy Lambda functions

