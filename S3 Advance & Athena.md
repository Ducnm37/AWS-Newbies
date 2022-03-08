# S3 Advance & Athena

S3 MFA (multi factor authentication): Xác thực hành động qua apps bên thứ 3 như google authen....

- Tạo MFA 

<img src="https://i.imgur.com/pqoCVH5.png">

- Sau khi cấu hình MFA muốn sử dụng để thao tác với resource trên S3 qua command line cần cấu hình s3 bằng lệnh, sau đó điềm access key và private key

```
aws configure --profile <tên mfa profile>
```

- S3 Access Logs: Không nên đặt log s3 cùng bucket với data vì mỗi khi put 1 object sẽ sinh ra 1 log và bucket log sẽ bị loop và làm đầy dữ liệu tốn chi phí lưu trữ (log sẽ có khoảng sau 1 2 giờ)

- Bucket replicate:
  - Cần enable bucket versioning để sử dụng bucket replication, tiếp theo vào mục **management** -> **Replication rules** để tạo rule replicate
  - Cross region replication (CRR): Truy cập có độ trễ thấp hơn, replicate qua account khác được
  - Same region replication (SRR): Tích hợp log, live replicate giữa product và test account
  - Chỉ sau khi active replicate thì các object mới tải lên mới có thể replicate, còn các object trước đó không thể replicate
  - Option **Delete marker replication** mặc định disable có nghĩa là khi xóa object ở bucket 1 thì sẽ ko replicate bản **Delete marker** trong versioning sang bucket 2, khi enable nó thì sẽ replicate bản delete versioning sang bucket 2

  <img src="https://i.imgur.com/0zfz7b6.png">
  
  
- S3 Tier là các loại loại phân vùng lưu trữ cho s3 object tương ứng với khả năng lưu trữ và thời gian lưu trữ
  - Amazon Glacier retrieval
    - expedited (1 đến 5 phút)
    - Standard (3 đến 5 giờ
    - Bulk (5 đến 12 giờ)
    - Storage tồn tại tối thiểu 90 ngày
  - Amazon Glacier Deep Archive
    - Standard (12 giờ)
    - Bulk (48 giờ)
    - storage tồn tại ít nhất trong 180 ngày

<img src="https://i.imgur.com/7n4kvfi.png">

- S3 Event Notification: Bắn event thao tác trên S3
  - Trước khi tạo Event Notice ta sử dụng thông báo qua công cụ SQS Queue

  <img src="https://i.imgur.com/hZxo7vr.png">
       
  - Trong queue vừa tạo vào mục **Access Policy** để tạo policy cho phép S3 được phép đẩy message vào queue
       
  <img src="https://i.imgur.com/cxt4Y4U.png">
  
  <img src="https://i.imgur.com/duyQLkc.png">
  
  - Edit policy và click vào **Policy generator** để tạo mẫu rule

  <img src="https://i.imgur.com/TDasC8E.png">
  
  - Ta được mẫu rule như sau và copy paste vào mục policy và save
  ```
  {
  "Id": "Policy1637994321433",
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1637994224428",
      "Action": [
        "sqs:SendMessage"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:sqs:ap-southeast-1:925663049766:Queu-demo",
      "Principal": "*"
    }
  ]
  ```

  - Tiếp đến phần tạo Event trong S3 bucket mục **Properties** -> **Event notifications** ->**Create event notification**
  
  <img src="https://i.imgur.com/qGmSWtc.png">
  
  - Chọn các thao tác cần giám sát thông báo

  <img src="https://i.imgur.com/QRe0fvb.png">
  
  - Chọn thông báo qua công cụ queue cho S3 (còn các option thông báo khác) 

  <img src="https://i.imgur.com/tqqk77K.png">
  
  - Sau khi tạo xong mỗi lần sửa, xóa, upload, bất kỳ thao tác gì trên object trong bucket đó sẽ được bắn thông báo đến SQS queue, vào SQS queue và chọn mục **Send message**

  <img src="https://i.imgur.com/9FBTtUx.png">
  
  - Trong mục **Send and receive messages** chọn Poll for messages** sẽ thấy thông báo được đẩy về
  - 
  <img src="https://i.imgur.com/Vk2xIQR.png">

--------------------------------------------------------

## S3 Athena

- Là Serverless query service, sử dụng ngôn ngữ SQL để phân tích, truy vấn quản lý log dữ liệu từ S3 (tương tự như bạn search tài liệu trên google)
- S3 Selection dùng để tìm kiếm thông tin trong object với các định dạng CSV, json, ORC, Avro, Parquet (Built on Presto)

<img src="https://i.imgur.com/7weCNUc.png">

<img src="https://i.imgur.com/l6DvbJd.png">



- Query rất nhanh, nhưng ban đầu cần thiết lập cáu hình
- Chi sphis 5$ cho 1 TB data scanned
- Sử dụng nén hoặc dạng cột sẽ giảm chi phí (scan ít)
- Sử dụng trong các trường hợp: Business intellegence/ analytics/ reporting, analyze & query VPC flow logs, ELB logs, CloudTrail...

- Tham khảo mẫu tạo table log tại đây: https://aws.amazon.com/premiumsupport/knowledge-center/analyze-logs-athena/
- Glacier Vault Lock chính sách kiểm soát thao tác trên object
- S3 object lock (phải bật versioning): khóa quyền ghi đè, xóa của người dùng, setup khi tạo bucket, sau khi sử dụng sẽ không thể thay đổi

