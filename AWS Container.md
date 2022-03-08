# Aws Container

<img src="https://i.imgur.com/pvndrSD.png">

- Amazon ECR (Elastic Container Registry): giống như docker hub lưu image cho container, có Private ECR và Public ECR
- Sự khác nhau giữa vm và container

<img src="https://i.imgur.com/YwJyFVw.png">

- Luồng hoạt động của container

<img src="https://i.imgur.com/ljBNdGU.png">
     
- ECS: Container của Amazon
- EKS: K8S của Amazon
- Fargate: Serverless container platform của Amazon
     

- Với ECS ta cần cung cấp và bảo trì cho hạ tầng EC2 instance
     
  <img src="https://i.imgur.com/FjA987N.png">

  - Có thể tích hợp với Application Load Balancer
  - Trên mỗi container sẽ có ECS agent để join container vào cluster và để khởi chạy ECS task (task giống như 1 file cấu hình cho container), để khởi tạo container cần run EC2 instance trước, nếu sử dụng AMI cho ECs thì ECS agent sẽ cấu hình trước

  <img src="https://i.imgur.com/ZRaEKPJ.png">

- Fargate: chạy container trên AWS, không quản lý hạ tầng (serverless offering)
  - Run container trên cơ sở RAM/CPU
  - Fargate sẽ run 1 task không cần khởi tạo EC2 và để truy cập vào task cần ENI (Elastic Network Interface)

  <img src="https://i.imgur.com/FyfrXDR.png">
  
- IAM role cho ECS task: IAM role gắn cho EC2 instance (EC2 instance profile và EC2 instance role) 
  - Để sử dụng cho ECS agent gọi API tới ECS service để register EC2 instance vào ECS cluster 
  - Nó cũng gửi logs vào cloudwatch 
  - Pull docker image từ ECR
  - Tham chiếu dữ liệu trong Secret Mangager hoặc SSM Parameter Store sử dụng EC2 instance profile 
  - Role chỉ sử dụng cho ECS agent

- ECS task role: Cho phép task nào gắn với role nào (ví dụ muốn cho phép EC2 task upload file vào S3 bucket)
  - Sử dụng role khác nhau cho ECS service khác nhau
  - Task role định nghĩ ra rule cho các service gắn nó

<img src="https://i.imgur.com/obvJ2U6.png">

- ECS data volume + EFS file system
  - Làm việc với cả 2 EC2 task và Fargate task
  - Có thể mount EFS volume vào task
  - Task chạy trên mọi AZ có thể chia sẻ cùng dữ liệu trên EFS volume 
  - Fargate + EFS = serverless + data storage mà không cần quản lý server

  <img src="https://i.imgur.com/HJ5juzC.png">

- Application Loadbalancer hỗ trợ tìm đúng port trên EC2 instance, ALB sẽ forward mọi request tới ECS task, ta phải cho phép truy cập trên security group của EC2 instance mọi port từ ALB security group
  - Tính năng Dynamic Port Mapping trên ALB cho phép tự động gắn port vào container để exposse ra ngoài

<img src="https://i.imgur.com/Bg45i64.png">

- Fargate sẽ create 1 ENI là 1 IP duy nhất nhưng các task lại sử dụng cùng 1 port (Elastic network interfaces) cho mỗi task, ta cần cho phép  ENI security group cho task port từ ALB security group 

<img stc="https://i.imgur.com/LqWojqD.png">

- ECS task invoked by Event Bridge ( Event Bridge là một bus bắt sự kiện thay đổi dữ liệu)
  - Upload dử liệu vào S3 bucket, và sau đó sử dụng fargate xử lý object đó và insert 1 số data vào DynamoDB .
  - Ta tạo 1 Amazon Event Bridge đọc dữ liệu từ S3 bucket, và Event Bridge có 1 rule target run tới ECS task, và task này được cho phép truy cập bới ECS task role, task cũng được phép lấy data từ S3 và  insert dynamoDB metadata. Khi dữ liệu được xử lý xong kết quả sẽ được lưu trong Amazon DynamoDB table 

<img src="https://i.imgur.com/5wn1Rej.png">

- ECS Scaling
  - Sử dụng **Cloudwatch metric** để giám sát tài nguyên sau đó trigger đến cloud **Cloudwatch alarm**, khi số task tăng lên cần tài nguyên sẽ tự động scale ECS service 
  - Đôi khi Tài nguyên của EC2 instance đang có không đủ dung lượng cho task mới ta cần scale nó bằng Amazon **ECS capacity provider** để thêm EC2 instance vào ECS cluster 
  
  <img src="https://i.imgur.com/NEXr8Qf.png">
  
- ECS scaling với SQS    
  - Tương tự như trên, khi nhiều message thì metric queue length của **Cloudwatch metric** trigger tới **Cloudwatch alarm** sẽ auto scale ECS service để tạo task mới, và nếu scale thêm tài nguyên cho EC2 instance sẽ dùng **ECS capacity provider** để thêm EC2 instance vào ECS cluster

  <img src="https://i.imgur.com/y5cS2E5.png">

- ECS rolling update: có 2 giá trị update đó là minimun healthy percen và maximum percen, với giá trị minunum ví dụ set là 90 thì sẽ có 10% số task bị stop để nâng cấp lên version mới, tượng tự nếu minimum là 100 và maximum là 150 thì sẽ không có task nào bị stop và 50% số lượng task ban đầu sẽ được thay thế sau khi số lượng task up date thêm đã tạo xong và xóa bỏ task version cũ , tiếp tục là 50% còn lại sẽ update, việc này đảm bảo 100% minimum task sẽ không có task nào bị stop

  - Minimun=100 maximum=150 (đảm bảo 100% số task vẫn running, tạo 50% task mới update sau đó mới xóa 50% task cũ và thay thế, tương tự với số lượng task còn lại)

  <img src="https://i.imgur.com/LHV6hNo.png">
  
  - Minimum=50 maximum=100 (50% số task bị stop sau đó mới tạo task mới được update, và 50% còn lại tương tự)

  <img src="https://i.imgur.com/dsORQtV.png">

- ECR (elastic container registry): Lưu trữ, quản lý và triển khai container trên AWS, nó chứa image của container
- Tích hợp đầy đủ với ECS và IAM cho việc bảo mật và image sau khi update được hỗ trợ lưu bởi S3
- Hỗ trợ scan image lỗi, version, tag, lifecycle
- Amazon EKS (elastic kubernetes service) quản lý k8s cluster trên AWS
  - K8s là opensource system tự động quản lý, deploy, scaling container sử dụng docker
  - K8s thay thế cho ECS nhưng khác API
  - EKS hỗ trợ EC2 nếu muốn deploy worker node hoặc Fargate nếu muốn deploy serverless container
  - Sử dụng cho việc người dùng muốn migrate hệ thống K8s on-premise lên cloud sử dụng K8s
  - EKS running on EC2, mỗi EKS pod tương đương với ECS task, EKS pod running on EKS node, EKS node quản lý bởi auto scaling group. Nếu muốn expose EKS service cần setup private load balancer hoặc public loadbalancer để expose port ra ngoài 
  
  <img src="https://i.imgur.com/rL0XJDs.png">
















