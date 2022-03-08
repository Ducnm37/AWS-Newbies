
- IAM: Tạo user, group, policy cho user và group có quyền thao tác với service hoặc resource nào của aws

- IAM role: sử dụng để gán quyền cho service chạy trên AWS, ví dụ như S3 chúng ta assign role chỉ read bucket thì khi show ra chỉ show được bucket mà không sửa xóa được bucket, hoặc user A có role 1 được phép xem máy ảo trên user a, role 1 này assign cho user B thì user B xem dc máy ảo của user A. 
- IAM Credentials Report (account-level): Thống kê danh sách các tài khoản và trạng thái đăng nhập của họ

- IAM Access Advisor (user-level): Hiển thị các quyền được cấp của service cho user, và hiển thị lần truy cập cuối cùng của service

- MFI: giống như bảo mật 2 lớp giúp việc bảo mật password cho account hơn, nó giống như googleauthen

<img src="https://i.imgur.com/MIoXWhG.png">

- Cài đặt AWS CLI: AWS CLI dùng để truy cập vào AWS thao tác với dịch vụ như tạo máy ảo, xóa,... thông qua CLI thay vì thao tác trên giao diện. Đầu tiên cần cài đặt AWS CLI trên máy của người dùng (window, linux, mac), linux thì có gói **awscli** 

  Sau khi cài đặt AWS CLI tiến hành tạo access key và private key cho user cần access
  
<img src="https://i.imgur.com/QLoP78E.png">

  Tiếp theo vào command line trên máy user configure key
  
<img src="https://i.imgur.com/NBtsVw1.png">

- AWS Cloudshell: là trình trên giao diện aws thao tác bằng dòng lệnh cli giúp bạn dễ dàng quản lý, khám phá và tương tác với các tài nguyên AWS của mình một cách an toàn, nó tương tự như AWS CLI nhưng sẽ bị hạn chế region
  
  AWS CloudShell có sẵn trong các region sau:
  
  - US East (Ohio)

  - US East (N. Virginia)

  - US West (Oregon)

  - Asia Pacific (Mumbai)

  - Asia Pacific (Sydney)

  - Asia Pacific (Tokyo)

  - Europe (Frankfurt)

  - Europe (Ireland)

- Billing: xem thông tin chi phí sử dụng, mặc định ban đầu các IAM user không có quyền xem billing mà chỉ có root user mới được xem, ta có thể enable permission cho phép IAM user xem billing bằng cách vào tài khoản user root -> vào mục **my account** -> kéo xuống mục **IAM User and Role Access to Billing Information** và chọn edit -> 
chọn **Activate IAM Access** và update sau đó quay trở lại account IAM user để xem

<img src="https://i.imgur.com/8sejKxn.png">

<img src="https://i.imgur.com/vOWCGQS.png">

Budget: Đặt ngân sách cho việc sử dụng dịch vụ trên AWS, vào mục **Budget** -> chọn create Budget trong có có thể set budget tự động gia hạn sau mỗi tháng (Recurring budget
) hoặc đặt budget cho 1 lần (Expiring budget) trong mục **Set budget amount**

<img src="https://i.imgur.com/zem0MSS.png">

------------------------

- EC2: Dich vụ máy ảo (Elastic compute cloud)
  - Sử dụng storage là SSD và EBS (Amazon Elastic Block Store)
  - Sử dụng ELB (Elastic Load Balancing) để cân bằng tải
  - Sử dụng ESG (Ayto-Scaling Group) để mở rộng các dịch vụ
  - OS: linux, window, mac
  - Truyền biến cài đặt gói (sử dụng bass shell) trong mục **user data** khi tạo máy ảo
  - Show Show IAM role trong vm **aws iam list-users**
- Security group
- ssh thông qua link domain DNS chứ ko phải IP, hoặc nếu ssh bằng key với ip public thì cần phân quyền cho key **chmod 400 keyname.pem**
- Purchase option:
  - Dedicate Host: Mua cả cục cluster
  - On Demand: Dùng bao nhiêu mua từng đấy, khi nào dùng thì mua
  - Reserved: Đặt kế hoạch mua trước và có thể nhận được giảm giá
  - Spot Instance: Cho phép người mua đấu thầu, người trả giá cao thì request tạo instance của họ sẽ được active và running, và có thể sẽ bị mất nếu người khác trả cao hơn. Nó chỉ phù hợp cho các tác vụ hoặc ứng dụng xử lý thời vụ hoặc không quan trọng nhằm tiết kiệm khá nhiều chi phí không cần thiết. Ví dụ như thuê EC2 on-demand mất 0.1$ 1 giờ thì thuê spot instance có thể chỉ mất 0.04$ cho 1 giờ. 
    - Spot Fleet là lựa chọn kết hợp giữa Spot Instances và On-Demand Instance cho phép tự động request Spot Instances với chi phí rẻ nhất
    - Spot Block Instances cho phép thời gian từ 1-6 giờ sử dụng mà không bị gián đoạn. Hiện tại chưa thấy có trên giao diện chỉ thấy cấu hình qua AWS Command Line Interface (CLI) bằng option block-duration-minutes:
```
      $ aws ec2 request-spot-instances \
       --block-duration-minutes 360 \
       --instance-count 2 \
       --spot-price "0.25" ...
```

- Placement Group: AWS hiện cung cấp 3 loại placement group có sẵn trong EC2 và tất cả chúng đều tồn tại vì những lý do khác nhau. Về cơ bản chúng cho phép chúng ta nhìn thấy hoặc kiểm soát vị trí thực tế - nơi mà các EC2 instance khởi chạy ở các mức độ khác nhau. Chúng ta có thể lợi dụng tính năng này để cải thiện hiệu suất hoặc độ tin cậy

  - Cluster: Gom trung các instance vào một cluster trong AZ (availability zone), Cluster placement groups chỉ khả dụng trong một AZ duy nhất trong cùng một VPC hoặc peered VPCs.
    <img src="https://i.imgur.com/kjdWnBv.png">
    <img src="https://i.imgur.com/rReHj6j.png">

    - Ưu điểm: Do các instance kết nối với nhau trong cùng 1 cluster trên 1 AZ, các instance đặt gần nhau về mặt vật lý nên sẽ có hiệu suất mạng kết nối tối đa
    - Nhược điểm: Do các instance cùng nằm cùng vị trí vật lý nên sẽ bị chỉ định vùng khả dụng ban đầu, có nghĩa là khi khởi tạo ban đầu ở đâu thì các instance sau sẽ chỉ ở đó, thứ 2 là việc đặt chung 1 vùng sẽ có nguy cơ mất mát dữ liệu tất cả khi vùng đó bị lỗi.
    - Lưu ý: Nên khởi chạy tất cả các instances trong groups cùng một lúc để đảm bảo dung lượng. Có thể thêm các instances vào sau, nhưng có khả năng gặp phải lỗi dung lượng không đủ. Giả sử bạn đã đưa 4 EC2 instances vào một cluster placement group, thì AWS sẽ tìm một thời điểm thích hợp để khởi chạy 4 EC2 instances này từ đó. Chúng có thể đảm bảo không gian này đủ cho thêm 2 instance nữa trong trường hợp muốn khởi chạy thêm nhưng nếu bạn muốn chạy thêm tận 4 instance, bạn có thể sẽ gặp vấn đề về dung lượng. Vì vậy, bằng cách chọn các instances cùng loại và khởi chạy tất cả chúng từ ban đầu, việc khởi chạy placement group với các instance sẽ có cơ hội thành công cao hơn
  - Partition:  là một nhóm cơ sở hạ tầng biệt lập bên trong AWS các partition không chia sẻ phần cứng với nhau, các instance được triển khai thành một partition placement group (PPG), tối đa 7 partition trong 1 AZ và số EC2 instance có thể lên tới 100, ngoài ra các partition có thể đặt ở AZ khác nhau trong cùng 1 region. Mỗi partition là một rack độc lập trong AZ, partition placement group (PPG) có thể trải rộng trên nhiều AZ trong một khu vực. Thường được sử dụng bởi khối lượng công việc phân tán và quy mô lớn, chẳng hạn như Hadoop, Cassandra và Kafka...
    - Ưu điểm: Khi 1 partition bị lỗi thì các partition khác không bị ảnh hưởng có thể hạn chế miền chịu lỗi và mất mát dữ liệu, có xu hướng chỉ được sử dụng cho các triển khai cơ sở hạ tầng lớn, chúng được chia đều cho tất cả các phân vùng cơ sở hạ tầng trong một region và do đó, bạn sẽ tạo một partition placement group có thể trải rộng trên các vùng
    - Nhược điểm: Không được hỗ trợ cho Dedicated Hosts. Đối với Dedicated Instances, có thể có tối đa 2 partitions và tốc độ không nhanh bằng cluster placement
    <img src="https://i.imgur.com/il8nxqZ.png">
    <img src="https://i.imgur.com/g0JKPHJ.png">
    
  - Spread: Là một nhóm các instances được đặt trên phần cứng riêng biệt, có thể trải rộng trên nhiều AZ. Được thiết kế cho tối đa 7 instance trên mỗi AZ cần được tách riêng. Mỗi instance chiếm một phân vùng. Kiến trúc này phù hợp cho các email servers, domain controllers, file servers, và application HA pairs.
    <img src="https://i.imgur.com/Zl2QuIh.png">
    
    - Ưu điểm: Cô lập được các vùng lỗi
    - Nhược điểm: Không được hỗ trợ cho Dedicated Instances hoặc Dedicated Hosts

    **Tạo placement group và gán instance vào placement group trên aws**
    
    <img src="https://i.imgur.com/svw0EVb.png">
    Chọn loại placement group khi  khởi tạo của EC2 để gán vào placement group
    <img src="https://i.imgur.com/kzuGXb5.png">
    Chọn target partition
    <img src="https://i.imgur.com/9NSJIIH.png">
    
- Elastic Network Interface (ENI): Giống như virtual network card, nó sử dụng IPv4 và có thể move từ instance này gắn vào instance khác
- EC2 Hibermate: Trạng thái ngủ đông cho vm, Ram của instance sẽ được đưa vào trạng thái preserved (bảo quản) và tất cả dữ liệu trên RAM lúc đó cũng được bảo quản, khi khởi động lại vm os sẽ boot rất nhanh vì OS không bị stop hay restart nó giống như đóng băng trạng thái vm, tất cả trạng thái của RAM sẽ được dump 1 tệp vào trong root EBS volume và sau đó  root EBS volume sẽ được mã hóa và sau đó instance sẽ shutdown nhưng OS thì không, sau khi restart  RAM sẽ được đọc từ EBS volume vào RAM instance và instance running. Việc này sử dụng cho trường hợp muốn giữ quá trình hoạt động lâu dài mà ko stop và khi có nhiều dịch vụ mất nhiều thời gian khởi tạo cần được lưu lại khi chưa muốn làm tiếp
  - Nó không hỗ trợ toàn bộ EC2 instance mà chỉ support các gói (instance type): C3, C4, C5, M3, M4, M5, R3, R4, R5. 
  - Ram size phải nhỏ hơn 150 GB
  - không hỗ trợ bare metal instance
  - root volume: đủ lớn để dump full ram size
  - Sử dụng được cho on-demand instance, reserved instance nhưng không cho spot instance
  - Instance được hibernate quá 60 ngày
- CPU instance: 1 CPU có 1 hoặc nhiều Threads, mỗi Thread được gọi là 1 vCPU. instance có nhiều CPU thì tốc độ càng cao
- Capacity Reservations: Giống như đặt trước tài nguyên trong 1 AZ, nếu bạn tạo instance đúng cấu hình thì sẽ nhảy vào pool Capacity Reservations còn khác thì sẽ không sử dụng Capacity Reservations
- Snapshot volume: Không cần detach volume để snapshot, volume snapshot có thể copy sang AZ or region khác
- AMI (Amazon Machine Image): Cho phép tạo image từ vm đang chạy, linh hoạt trong việc tạo ra image or template cấu hình theo ý muốn 
- EC2 instance store: cung cấp nơi lưu trữ với hiệu xuất cao, iops có thể lên đến hàng triệu, cache và buffer đều tốt, nhược điểm là nếu bạn hủy instance hoặc stop instance thì dữ liệu trên EC2 instance store sẽ mất, vì thế cần có giải pháp backup dữ liệu khi sử dụng
<img src="https://i.imgur.com/a9ZrsXw.png">

- EBS volume type: Có 6 loại volume EBS. Trong đó chạy OS là gp2/gp3 hoặc io1/io2.
  - gp2 / gp3 (SSD): Loại phổ biến giả cả phù hợp với hiệu năng bình thường.
    - gp3 có iops cơ bản là 3000 có thể tăng lên đến 16000 và throughput cơ bản là 125Mib/s có thể tăng lên đến 1000 Mib/s
    - gp2 có iops cơ bản là 3000 có thể tăng lên đến 16000 (3 iops mỗi Gb) và throughput cơ bản là 125Mib/s có thể tăng lên đến 1000 Mib/s
  - io1 / io2 (SSD): SSD volume hiệu năng cao, độ trễ thấp và thông lượng cao. PIOPS (Provisioned IOPS) phù hợp với ứng dụng đòi hiệu suất cao hơn 16000 iops, có hỗ trợ EBS Multi-attach
    - io1 max iops là 32000 cho Nitro EC2 instance và 32000 cho các loại instance khác
    - io2 Độ trể thấp, iops tối đa là 256000 với tỉ lệ 1 GB chứa 1000 iops
  - st1 (HDD): Volume giá rẻ cho việc đọc ghi nhiều hiệu năng thấp, không sử dụng làm boot volume được và không hỗ trợ multi-attach, max iops là 500 và max throughput là 500 Mib/s phù hợp sử dụng cho log processing, data warehouse, big data
  - sc1 (HDD): volume giá rẻ cho việc đọc ghi ít hiệu năng thấp, không sử dụng làm boot volume được và không hỗ trợ multi-attach, max iops là 250 và max throughput là 250 Mib/s
- EFS (Elastic file system): Sử dụng NFSv 4.1, hỗ trợ 1000 NFS client và 10 Gb+ /s throughput , throughput cao thì kích thước file system phải bé
  - Performance mode: Thông lượng cao (media , bigdata)
    - General purpose (default): Nhạy cảm với độ trễ (web server, CMS...)
    - Max I/O : Có thể điều chỉnh mở rộng được thông lượng (big data, media processing)
  - Throughput mode: 
    - Busting: 1TB = 50MiB/s + Burst up to 100 MiB/s
    - Provisioned: có thế set throughput theo dung lượng storage.
- Cách sử dụng EFS: ta cần tạo VM và cài đặt gói "amazon-efs-utils" trên vm
  - Tạo EFS file và assign security group đã mở NFS vào file system đó
  <img src="https://i.imgur.com/KuA2sY8.png">
  
  - Sau đó click vào EFS file system đã tạo chọn attach sẽ có hướng dẫn mount file system vào vm
  <img src="https://i.imgur.com/baq6IQo.png">
------------------------------------------------------------------------
- High Availability & Scalability
  - High Availability: Chạy instace ở nhiều AZ để phòng, auto scaling group multi AZ, load balancer multi AZ
  - Vertical Scalability: Mở rộng kích thước size instance (từ gói t2.nano 0.5G Ram - 1 vCPU  đến gói u-12tb1.metal 12.3T RAM - 448 vCPU)
  - Horizonal Scalability: Tăng số lượng instance, auto scaling group, load balancer
- Có 4 loại load balancer:
  - Classic load balancer (v1 thế hệ cũ): HTTP, HTTPS, TCP, SSL
  - Application load balancer (v2 thế hệ mới): HTTP, HTTPS, WebSocket, load balancer qua header x-forward. Khi sử dụng cần tạo target group để forward data, có thể route traffic tới các Target Groups khác trên cơ sở URL Path, Hostname, HTTP Headers, và Query Strings.
  - Network load balancer (v2 thế hệ mới): TCP, TSL, UDP, có thể load balancer qua static IP. Độ trễ thấp ~ 100 ms (400 ms for ALB) highest performance
  - Gateway load balancer (network layer): operates at layer 3 - IP protocal
- Sticky sesion:  Tính năng này route các yêu cầu cho một session cụ thể đến cùng một máy tính vật lý, và máy tính này phục vụ yêu cầu đầu tiên cho session đó. Tính năng này chủ yếu được sử dụng để đảm bảo một in-proc session nào đó sẽ không bị mất bởi các yêu cầu cho session được route đến các máy chủ khác nhau. Vì yêu cầu từ 1 người dùng luôn được route đến cùng một máy đã phản hồi yêu cầu lần đầu cho session đó, các sticky session có thể gây ra phân phối tải không đồng đều trên các máy chủ. Ta có thể enable ở trong phần target group
- Cross-zone Load Balancer: với cross-zone LB thì traffic sẽ được cân bằng cho các instance trong các AZ là giống như nhau, còn khi không có Cross-zone thì traffic sẽ chia đều cho các zone còn các zone sẽ chia đều lại cho instance
  <img src="https://i.imgur.com/QJSXvRY.png">
  
- SSL chỉ làm việc được với Application Load Balancer và Network Load Balancer, không làm việc được với Classic Load Balancer.
  - Tính năng cho phép Application Load Balancers và Network Load Balancers sử dụng multi SSL certificates trên listener là Server Name Indication (SNI)
- Cookies của Elastic Load Balancer có 3 tên được bảo lưu (reserved) là AWSALB, AWSALBAPP, AWSALBTG) bạn có thể đặt tên khác ngoài 3 tên này cho cookie của riêng bạn

-------------------------------------------------------------------------------------------

EC2 Metadata instance : Command để lấy metadata EC2 instance `curl http://169.254.169.254/latest/meta-data`


