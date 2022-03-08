# Amazon Simple Queue Service và Kinesis

## SQS

<img src="https://i.imgur.com/xlM0m3x.png">

- Không giới hạn throughput, không giới hạn số lượng messages trong queue
- Messages mặc định được giữ lại trong 4 ngày và tối đa 14 ngày
- Đỗ trễ thấp < 10 ms (publish và receive)
- Mỗi messages gửi đi bị giới hạn 256KB
- Messages có thể duplicate (thỉnh thoảng ít nhất mỗi lần chuyển)
- Có thể messages không theo thứ tự
- Sau khi messages đến được đích thì consumer sẽ xóa messages đó trên queue 

<img src="https://i.imgur.com/AFul40j.png">

- Consumers có thể chạy trên EC2 instance, servers, AWS Lambda...
- Đôi khi message không được gửi đủ từ 1 consumer mà nó sẽ gửi thêm từ các consumer khác

<img src="https://i.imgur.com/wNNFPxY.png">

- Có thể sử dụng SQS cho backend hoặc frontend kết hợp với auto scaling group

<img src="https://i.imgur.com/BFg5xFT.png">

- SQS sử dụng HTTPS API, mã hóa sử dụng KMS keys
- Kiểm soát truy cập với IAM policy để điều tiết truy cập tới SQS API
- SQS access policy tương tự như S3 bucket policy
  - Hữu ích cho việc cross-account truy cập tới SQS
  - Hữu ích cho phép các dịch vụ khác (S3, SNS,...) ghi vào SQS
- SQS queue access policy

<img src="https://i.imgur.com/tJwdWJi.png">

- Access SQS policy : ví dụ dưới là template SQS policy access vào S3
  -   Account ID: 925663049766
  -   Object bucket name: testsqs12345423213124
  -   ARN SQS: arn:aws:sqs:ap-southeast-1:925663049766:myqueue


```
{
 "Version": "2008-10-17",
 "Id": "__default_policy_ID",
 "Statement": [
  {
   "Sid": "__owner_statement",
   "Effect": "Allow",
   "Principal": {
    "Service": "s3.amazonaws.com"  
   },
   "Action": [
    "SQS:SendMessage"
   ],
      "Resource": "arn:aws:sqs:ap-southeast-1:925663049766:myqueue",
   "Condition": {
      "ArnLike": { "aws:SourceArn": "arn:aws:s3:*:*:testsqs12345423213124" },
      "StringEquals": { "aws:SourceAccount": "925663049766" }
   }
  }
 ]
}
```

- Sau khi 1 messages được poll bởi consumer nó sẽ không hiển thị với các consumers còn lại (visible), có nghĩa khi 1 consumer poll thì các consumer khác sẽ chỉ poll được messages sau khi consumer đầu thực hiện xong tiến trình poll và vào visible timeout
- Mặc định thì mesages visible sẽ timeout sau 30 giây messages sẽ return lại, nếu sau 30s mà messages vẫn không xử lý kịp thì 1 consumer sẽ call API ChangeMessageVisibility để lấy thêm thời gian
- Nếu visibility timeout lâu hàng giờ thì consumer sẽ bị crashes và tiền trình  chạy lại sẽ mất thời gian
- Nếu visibility timeout rất nhanh vài giây thì chúng ta có thể nhận duplicates messages

<img src="https://i.imgur.com/kyCzywW.png">

- Khi queue bị quá tải maximum recieves thì messages sẽ được đưa vào Dead letter queue (DLQ) để chuyển sang queue khác, Dead letter queue giúp lọc ra các thông báo lỗi để ta debug dễ dàng hơn
  -  Cấu hình dead letter queue trong queue, giá trị **Maximum receives** là số lần tối đã queue nhận messages nếu quá thì sẽ put messages sang dead letter queue được cấu hình mục **choose queue** tức là queue A khi nhận quá tải messages thì sẽ put messages sang queue B
  
  <img src="https://i.imgur.com/gv4oj1i.png">

- Delay messages tức là khiến consumer không thể nhận messages ngay lập tức có thể delay tới 15 phút, mặc định là 0. Giá trị này khác với Visibility Timeout ở chỗ giá trị này chỉ được đếm đối với lần đầu tiên message được cho vào queue, còn Visibility Timeout sẽ đếm với mỗi lần thêm message vào queue.

<img src="https://i.imgur.com/soGRmhj.png">

  - Set up delay queue tại mục **configuration**
   
  <img src="https://i.imgur.com/6RW1KzQ.png">
  
- Long polling mục đích là giảm thiểu những response empty từ phía queue khi trong queue ko có messages thay vì connection bị timeout hoặc response trả về là empty,  consumer vẫn sẽ đợi để poll messages, khi có messages sẽ ngày lập tức poll đến consumer đồng thời cũng giảm độ trễ khi nhận messages, wait time của long polling từ 1 - 20 giây, trong khi shot polling thì giá trị wait time sẽ là 0
  
  <img src="https://i.imgur.com/Gx3SLPe.png">
  
- Request - response system

<img src="https://i.imgur.com/Wayj7Ee.png">

- FIFO queue (first in first out): messages sẽ vào ra ra theo đúng thứ tự vào trước ra trước 
  - Limit throughput: 300 messages/s  nếu không phân lô batching, còn nếu phân theo batching sẽ có thoughput là 3000 messages/s
  - Messages sẽ được xử lý theo thứ tự đến consumer
  
  <img src="https://i.imgur.com/C3XYYWK.png">
  
  - Khả năng gửi chính xác 1 lần (xóa bản duplicate)

  <img src="https://i.imgur.com/GD2WJkl.png">

  - Thứ tự sẽ đánh theo **Message deduplication ID** và **Message group ID**

  <img src="https://i.imgur.com/BJU5Mr0.png">

- SQS với auto scaling group
  - Khi nhận được nhiều messages, EC2 instance push metric tới cloudwatch custrom metric xem bao nhiêu messages đang đợi để xử lý xem có đủ instance để xử lý không nếu không đủ sẽ đẩy cảnh báo tới cloudwatch alarm để thông báo cho auto scaling group mở rộng instance theo policy đã đề ra

  <img src="https://i.imgur.com/nwpvOYn.png">

- SNS topic: Nó giống như 1 nơi trung chuyển messages, các subscriber nhận messages từ SNS (các subscriber là các thành phần như, http/https, SQS queue, lambda, email notification....) 
  - Mỗi subscriber sẽ nhận messages từ topic của nó
  - Lên tới 10.000.000 subscriber cho mỗi 1 topic và tối đa 100.000 topic
  - Topic Publish ( sử dụng SDK)
    - Tạo 1 topic
    - Tạo 1 subscription (hoặc nhiều)
    - Publish topic
  - Direct Publish ( dành cho mobile apps SDK)
    - Tạo 1 nền tảng ứng dụng
    - tạo 1 nền tảng endpoint
    - Publish trên nền tảng endpoint
    - Hoạt động với Google GCM, APNS, Amazon ADM...
    
    <img src="https://i.imgur.com/kDwPBew.png">

  - Mã hóa:
    - Sử dụng HTTPS API
    - Sử udngj KMS key mã hóa
    - Client có thể tự mã hóa và giải mã
    - Kiểm soát quyền truy cập sử dụng IAM policy để truy cập SNS API
    - SNS access policies cho S3 bucket cho phép các service write tới 1 SNS topic 
    - Cho phép cross-account truy cập SNS topic
- SNS kết hợp với SQS queue để dự phòng trong trường hợp SQS bị lỗi cần SQS queue khác để dự phòng
  - Push 1 topic hoặc event vào SNS thì tất cả các SQS sẽ nhận được ( fan-out )
  - SQS cho phép bảo toàn dữ liệu và làm chậm quá trình xử lý
  - Có thể thêm nhiều SQS subscriber mọi lúc
  - Phải cho phép SNS được phép write vào SQS queue qua access policy

  <img src="https://i.imgur.com/AG2OJGt.png">

-  Có thể sử dụng SNS fan-out với FIFO

<img src="https://i.imgur.com/Y4xzYEW.png">

- Create topic và create subscription trong topic để sử dụng
- Tính năng Message Filtering
  - Dùng JSON policy để filter mesages nếu đáp ứng được policy sẽ gửi tới SNS topic subscription, nếu không sẽ cancel order, hoặc decline...

  <img src="https://i.imgur.com/ky9FCuR.png">

-----------------------------------------------------

## Kinesis

- Kinesis là công cụ thu thập, xử lý và phân tích dữ liệu streaming ở real-time (App logs, website clickstreams, metrics, IoT telemetry data...)
- Kiến trức của Kinesis gồm
  - Producers: là nơi đưa data hay record vào trong Kinesis
  - Consumer: Là nơi lất data từ Kinesis và xử lý nó (EC2, Lambda...)
  - Shards: là một nhóm data records được xác định và duy nhất trên một stream, nó giống như 1 con đường chưa lưu lượng là data truyền vào càng nhiều shard thì càng truyền được nhiều data, có tốc độ truyền 1 MB/s đầu vào và 2 MB/s đầu ra
  - Data Record: Là 1 đơn vị trong Kinesis Streams no bao gồm
    - Sequence Number: Là số thứ tự hay id của Data Record
    - Partition Key: Được sử dụng để biết data record thuộc về Shard nào trong Kinesis, partition key thường đọc Unicode string có độ dài max là 256 bytes
    - Data Blob: Là dữ liệu của chúng ta, là một chuỗi bytes với max là 1MB, data blob không thể bị thay đổi

<img src="https://i.imgur.com/3DxOHn0.png">

- Các Kinesis là: Kinesis Data Streams, Kinesis Data Firehose, Kinesis Data Analytics, Kinesis Video Stream      
     
- Kinesis Data Streams: Capture, xử lý và lưu data stream
  - Thanh toán cho từng shard, có thể có nhiều shard 
  - Thời gian sống mặc định là 24h tới 346 ngày
  - Có thể xử lý lại data
  - Khi data được đưa vào Kinesis thì nó không thể xóa
  - Data chia sẻ cùng partition trên shard, khi producer sử dụng cùng partition key thì data sẽ cùng shard
  - Producer là AWS SDK, Kinesis Producer Library (KPL), Kinesis Agent
  - Consumer: AWS SDK, Kinesis Client Library, được quản lý bởi AWS Lambda, Kinesis Data Firehose, Kinesis Data Analytics

- Kinesis Data Firehose: Lưu data vào target destinationse
     
  <img src="https://i.imgur.com/nspyWpf.png">
     
  -  Kinesis Data Firehose đọc data từ Kinesis Data Streams lên tới 1 MB 1 lần, Lambda function transform record và truyền vào 1 lượng lớn data vào target database hoặc target destination (batch write)
  -  Dịch vụ được quản lý đầy đủ, tự động mở rộng, serverless
    - AWS: Redshift / Amazon S3 / ElasticSearch
    - 3rd party partner: Splunk / MongoDB / DataDog / NewRelic / ...
    - Custom: Send to any HTTP endpoint
  - Trả phí cho data thông qua Firehose
  - Gần như real-time
    - Đỗ trễ tối thiểu là 60 giây cho non full batches
    - Tối thiểu 30 MB data 1 lần
  - Hỗ trợ nhiều định dạng dữ liệu, conversion, transformation, nén
  - Hỗ trợ custom data transformation sử dụng AWS Lambda
  - Có thể gửi lỗi hoặc all data tới backup S3 bucket

- So sánh Data Stream và Firehose

<img src="https://i.imgur.com/USrhzpo.png">

  
  
- Kinesis Data Analytics: Phân tích data stream với SQL hoặc Apache Flink
  - Phân tích real-time sử dụng SQL
  - Quản lý đầy đủ, không cần server
  - Tự động mở rộng
  - Có thể tạo stream out of real-time queries
  - Use cases:
    - Time-series analytics
    - Real-time dashboard
    - Real-time metrics
 
  
  <img src="https://i.imgur.com/hCM1hcj.png">
  
- Kinesis Video Stream: Capture, xử lý và lưu video stream
  - Giúp cho việc truyền video theo cách bảo mật từ các thiết bị kết nối tới AWS dễ dàng hơn để phân tích, thực hiện machine learning (ML), phát lại và các phép xử lý khác
  - cho phép bạn phát lại video để xem trực tiếp và theo nhu cầu, đồng thời nhanh chóng xây dựng các ứng dụng có thể tận dụng phân tích hình ảnh và video của máy tính bằng cách tích hợp với Amazon Rekognition Video và các thư viện cho framework ML như Apache MxNet, TensorFlow và OpenCV
  - Kinesis Video Streams cũng hỗ trợ WebRTC, một dự án nguồn mở cho phép truyền phát nội dung phương tiện và tương tác theo thời gian thực giữa các trình duyệt web, ứng dụng di động và các thiết bị được kết nối thông qua các API đơn giản. Các chức năng sử dụng điển hình bao gồm trò chuyện bằng video và truyền phát nội dung phương tiện ngang hàng
  
---------------------------------------
  
### Tạo Kinesis Data Stream
  
<img src="https://i.imgur.com/6iWHehM.png">

- Sau khi tạo data stream vào cấu hình option trong **enhanced fan-out** chọn **Enhanced (shard-level) metrics** chọn tick tất cả
     
<img src="https://i.imgur.com/J2gTtLt.png">
  
- Bật cloudshell trên aws và check version với lệnh 
```
aws --version
```
  
- Tạo data và put vào stream (lệnh dưới sử dụng cho aws cli ver 2)
```
aws kinesis put-record --stream-name test --partition-key user1 --data "user signup" --cli-binary-format raw-in-base64-out
aws kinesis put-record --stream-name test --partition-key user1 --data "user login" --cli-binary-format raw-in-base64-out
aws kinesis put-record --stream-name test --partition-key user1 --data "user logout" --cli-binary-format raw-in-base64-out
```

<img src="https://i.imgur.com/RUz5m9S.png">
  
- show mô tả stream data để lấy **shard-id**
```.
aws kinesis describe-stream --stream-name test
```
<img src="https://i.imgur.com/NUgNCPM.png">
  
- Tạo shard iterator
```
aws kinesis get-shard-iterator --stream-name test --shard-id shardId-000000000000 --shard-iterator-type TRIMHORIZON
```
  
<img src="https://i.imgur.com/na1bWOl.png">
  
- Lấy record (phần **Data** đã được mã hóa base64 có thể dùng decode để giải mã ra nội dung)
```
aws kinesis get-records --shard-iterator <shard iterator>
```

<img src="https://i.imgur.com/1BV9CqQ.png">


### Tạo Kinesis Data Firehose

- Tạo Firehose
  
<img src="https://i.imgur.com/Xf6DxuE.png">

- Chọn bucket chứa data cho Firehose
  
<img src="https://i.imgur.com/YwGGfI0.png">

- Chọn buffer, nén  ( nếu buffer và interval càng lớn thì độ trễ càng cao, vì sau thời gian interval thì buffer mới đẩy data về bucket)
  
<img src="https://i.imgur.com/UlnFNdx.png">

- Push object trên data stream bằng cloudshell
```
aws kinesis put-record --stream-name test --partition-key user1 --data "user signup" --cli-binary-format raw-in-base64-out
aws kinesis put-record --stream-name test --partition-key user1 --data "user login" --cli-binary-format raw-in-base64-out
aws kinesis put-record --stream-name test --partition-key user1 --data "user logout" --cli-binary-format raw-in-base64-out
```
  
- Sau khi push data vào kiểm tra trong bucket sau 1 khoảng thời gian interval đã setup ở trên sẽ thấy data

<img src="https://i.imgur.com/8feD2dK.png">  

<img src="https://i.imgur.com/3MlonTn.png">

- Ordering data vào Kinesis
  - Image chứa 100 truck, ví dụ có 5 truck, truck nào cùng partition key sẽ truyền data trên cùng shard
  
  <img src="https://i.imgur.com/y4RtuyW.png">
  
- So sánh Kinesis Data Stream và SQS FIFO, một ví dụ khi 100 trucks, Kinesis Data Stream có 5 shard, và SQS có 1 FIFO
  
<img src="https://i.imgur.com/4dMrFKh.png">
  
  
  - Tùy vào bài toán sẽ chọn SQS FIFO hay Kinesis, ví dụ muốn nhiều consumer thì chọn FIFO hoặc muốn truyền lượng lớn truck và lượng lớn data thì lại dùng Kinesis
  
  
- So sánh giữa SQS - SNS - Kinesis

<img src="https://i.imgur.com/t1FD6ko.png">
  
  
## Amazon MQ
  
**Nếu một ngày bạn phải migrate ứng dụng từ on-premise lên cloud mà không thiết kế lại và ứng dụng sử dụng một số giao thức chuẩn như MQTT hoặc MQP thì Amazon MQ là 1 lựa chọn
  
- Không giống như SQS, SNS là những giao thức độc quyền của AWS
- Ứng dụng chạy trên on-premise có thể sử dụng các giao thức mở như: MQTT, MQP, AMQP, STOMP, Openwire, WSS, 
- Khi migrate lên cloud và thiết kế lại ứng dụng để sử dụng SQS và SNS chúng ta có thể sử dụng Amazon MQ
- Amazon MQ = managed Apache ActiveMQ
- Amazon MQ không mở rộng như SQS/SNS
- Amazon MQ chạy trên a dedicated machine, có thể chạy HA với failover
- Amazon MQ có cả 2 tính năng queue như SQS và topic như SNS

- Amazon MQ High Availability
  - Có node active và standby ở 2 AZ khác nhau, khi quá trình failover thất bại thì storage cũng sẽ được mount vào node standby vì thế data ở 2 node active và standby là như nhau chính vì thế user có thể lấy data từ node standby

  <img src="https://i.imgur.com/YlGeqRi.png">
