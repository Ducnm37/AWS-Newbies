# Dịch vụ monitor trên AWS

## Cloudwatch

- Cung cấp metrics cho mọi service
- Metrics là các giá trị để monitor
- Metrics thuộc về các namespaces
- Dimension là 1 thuộc tính của metric (instance id, enviroment, etc...)
- Metric có timestamps
- Có thể tạo Cloudwatch metric dashboard
- Ec2 instance metric có metric mỗi 5 phút 1 lần
- Với việc monitor chi tiết (trả phí) sẽ có data mỗi 1 phút 1 lần
- Sử dụng monitor chi tiết có thể scale nhanh hơn cho ASG
- AWS free tier có 10 detail metric monitoring (EC2 Memory usage mặc định sẽ ko push mà phải push từ inside instance như custom metric)
- Standard: push metric 60 giây
- High Resolution: 1/5/10/30 giây (chi phí cao)
- Timestamp up to 2 week
- Thu thập log từ Multi-Account, Multi-Region

<img src="https://i.imgur.com/ibSVNBq.png">

## Event Bridge

- Thực hiện cấu hình bắn event service trên AWS cho user hoặc event của partner
- Partner event bus: nhận event từ bên thứ 3 SaaS service hoặc ứng dụng (Zendesk, DataDog, Segment...)


## CloudTrail

- Sử dụng để cấu hình quản lý event
- event mặc định lưu 90 ngày trong Cloudtrail sau đó sẽ xóa

<img src="https://i.imgur.com/eCGT5Pq.png">

## AWS config
- Nhận thông báo NSN cho mọi thay đổi
- AWS config cho mỗi region service
- Có thể tích hợp trên các region và account
- Có thể sử dụng AWS quản lý cấu hình rule, custom rule
- Rule có thể trigger thay đổi cấu hình
- Sử dụng EvenBridge để trigger thông báo khi AWS resource không tuân thủ chính sách
<img src="https://i.imgur.com/zRxtrLS.png">

- Có thể gửi thay đổi cấu hình và thông báo state qua SNS

<img src="https://i.imgur.com/J8Luo0D.png">



## Tổng kết

<img src="https://i.imgur.com/FwiUKSK.png">



<img src="https://i.imgur.com/vACfOTr.png">


## Tips

- Muốn cloudwatch thu thập metric mỗi 1 phút thì enable detailed monitoring (mặc định là 5 phút 1 lần)
- Dùng cloudwatch log metrics filter lỗi error như 1 keyword sau đó sử dụng cloudwatch alarm dựa trên lỗi đã filter
- Để monitor RAM trên cloudwatch thì cần đặt cloudwatch agent 
- CloudWatch Alarm set  trên High-Resolution Custom Metric thường 10 giây 1 lần
- Cloudtrail insight check những hành động bất thường trên AWS account
- AWS config remediations sử dụng để tự động re-configure lại cấu hình theo AWS Config đã set
- AWS Config notification sử dụng để thông báo sự vi phạm rule của AWS config
