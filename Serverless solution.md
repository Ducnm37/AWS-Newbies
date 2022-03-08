## Mobile app

- Expose REST API với HTTPS

- REST API layer

  <img src="https://i.imgur.com/kJDf4w7.png">

  <img src="https://i.imgur.com/cZVxi7T.png">
  
## DynamoDB global table + user welcome email flow

  <img src="https://i.imgur.com/vNWRaht.png">
  
## Thumbnail general flow

  <img src="https://i.imgur.com/Z6koxn1.png">

## Micro service enviroment

  <img src="https://i.imgur.com/580qIVo.png">
  
- Miễn phí thiết kế theo ý muốn
- Synchronous patterns (là các giao thức chủ yếu phục vụ cho việc HTTP/HTTPS request/response, lý do gọi là giao thức đồng bộ là client chỉ thực hiện được tác vụ tiếp theo khi mà nó nhận được response): API gateway, Loadbalancer.
- Asynchronous patterns( sử dụng cho việc phát event, hệ thống messages bất đồng bộ, điểm khác biệt với synchronous là khi messages gửi đi nó không cần đợi response trả về mà có thể tiếp tục thực hiện các tác vụ khác, messages sẽ được xử lý bởi queue hoặc ác messages broker khác): SQS, Kinesis, SNS, Lambda trigger (S3) 
- Thách thức với micro service:
  - Vấn đề chi phí khi tạo lại
  - Vấn đề tối ưu về phần sử dụng
  - Sự phức tạo của việc chạy nhiều phiên bản với nhiều micro service đồng thời
  - Sự ra tăng về các yêu cầu tích hợp riêng rẽ từ code của khách hàng
- Một số thách thức đã được xử lý:
  - API gateway, Lambda đã được tự động mở rộng và trả phí theo hiệu suất sử dụng
  - Dễ dàng clone API và tái sử dụng môi trường
  - Tạo ra client SDK thông qua việc tích hợp Swagger cho API gateway 

## Premium user video service distribute content

  <img src="https://i.imgur.com/sJCuWAk.png">
  
  - Cognito thực hiện xác thực 
  - DynamoDB thực hiện lưu thông tin user
  - serverless application: thực hiện user registration và tạo Cloudfront Signed URL
  - Content lưu ở S3
  - Tích hợp Cloudfront và OAI cho việc bảo mật (user không thể bypass)
  - Cloudfront chỉ có thể sử dụng Signed URLs để ngăn chặn user trái phép

## Big data ingrestion pipeline

  <img src="https://i.imgur.com/bMZwW1c.png">
  
  <img src="https://i.imgur.com/88un2mL.png">
  
  

  
