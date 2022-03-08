# Route 53

- Route 53 là dịch vụ quản trị về DNS network của aws

<img src="https://i.imgur.com/80y5BuP.png">

- Route 53 được đặt tên theo port 53 DNS

- Route 53 là một Domain Registrar

- Mỗi bản record chứa:
  - Domain/subdomain Name - e.g.,example.com
  - Record Type - e.g.,A or AAAA
  - Value - e.g.,123.456.789.123

- Route 53 hỗ trợ các định dạng record:
  - (must know) A / AAAA / CNAME / NS
  - (Advanced) CAA / DS / MX / NAPTR / PTR / SOA / TXT / SPF / SRV

- Record Type
  - A map to hostname ipv4
  - AAAA map to hostname ipv6
  - CNAME map a hostname to another hostname
    - Mục tiêu là một domain name phải có 1 bản ghi A hoặc AAAA
    - Không thể tạo 1 bản ghi CNAME cho top node của DNS namespace (zone apex): không thể thao example.com nhưng có thể tạo www.example.com
  - Name servers cho Hosted Zone : kiểm soát traffic đến 1 domain (hosted zone chứa bản ghi điều hướng traffic tới domain và subdomain)

- Hosted Zone
  - Public Hosted Zone: Điều hướng traffix tới domain trên internet
  - Private Hosted Zone: Điều hướng traffix tới domain private trong 1 hoặc nhiều VPCs
- Buy 1 domain trên Route 53 chi phí tối thiểu 12$ 1 năm
- Cache TTL: Là thời gian cache domain, ví dụ set up cache TTL có thời gian 300s thì trong 300 đó domain sẽ không bị thay đổi về ip mặc dù trên hệ thống đã đổi ip khác nhưng khi truy cập vẫn là ip cũ, và sau 300s thì mọi thứ mới được cập nhật (việc này giúp client đỡ tốn thời gian request đến route53 để hỏi bản ghi domain có ip là gì)

<img src="https://i.imgur.com/f5BIIT0.png">

- AWS Resource: loadbalancer, CloudFront,S3 website, API gateway, Elastic Beanstalk, VPC interface endpoint, global accelerator, Route53 Record cùng hosted zone... có tác dụng tạo ra 1 AWS hostname (ví dụ myapp.mydomain.com)
- CNAME có tác dụng trỏ 1 hostname tới mọi hostname khác, nó chỉ sử dụng cho non root domain
- Alias là 1 tính năng mở rộng của DNS
  - có tác dụng trỏ 1 hostname tới 1 AWS Resource ( ví dụ: myapp.mydomain.com -> myold.alibaba.com), Alias sử dụng cho cả root domain và non root domain, sử dụng free và native với healthcheck
  - Tự động check sự thay đổi của IP nguồn
  - Không giống như CNAME nó sử dụng cho top node of a DNS namespace (Zone Apex)
  - Alias Record luôn ở định dạng A/AAAA cho AWS Resource (ipv4/ipv6)
  - Không thể set TTL
  - Không thể set Alias Record cho EC2 DNS name
- Health Check: 
  - HTTP Health Check cho public resource
  - Monitor Endpoint
  - Monitor another Health Check
  - Monitor CloudWatch
  - Hỗ trợ giao thức HTTP, HTTPS, TCP
- Routing Policy
  - Simple: 
    - Setup 1 bản ghi 1 ip hoặc 1 bản ghi nhiều ip, lúc đó sẽ random trong số các ip đó map vào bản ghi và sau khi hết TTL thì lại random lại
    - Không có Health Check
  
  <img src="https://i.imgur.com/zlRMRWS.png">
  
  - Weighted:
    - Kiểm soát tỉ lệ request tới mỗi resource được chỉ định
    - DNS Record phải cùng tên và loại
    - Có thể sử dụng Health checks
    - Use case: sử dụng load balancing giữa các regions, test new apps version
    - Set Weighted = 0 để stop sending request tới các resource
    - Nếu tất cả các bản ghi là 0 thì tất cả các bản ghi sẽ quay trở lại chỉ số ngang nhau
    
    <img src="https://i.imgur.com/HV4Ss8s.png">
  
  - Failover
    - Cho phép khi bản ghi primary chết thì bản ghi secondary sẽ thay thế
    - Có thể dùng Health Checks
  - Latency based
    - Điều hướng tới các resource với độ trễ thấp nhất giữa người dùng và AWS Regions
    - Có thể dùng Health Checks
  - Geolocation
    - Cho phép user truy cập tại bản ghi cùng region, gần nhất
    - Có thể dùng Health Checks
  - Multi-Value answer
    - Sử dụng cho multiple resource 
    - Một client khi truy cập sẽ chỉ truy cập được vào 1 bản ghi cố định trong số các bản ghi được cấu hình, các client khác nhau có thể truy cập vào cố định 1 bản ghi khác nhau. Khi 1 bản ghi bị lỗi sẽ chuyển sang dùng bản ghi còn lại (Ví dụ: Multi-Value cấu hình gồm bản ghi A và B, client 1 khi truy cập sẽ vào cố định bản ghi A (hoặc B) và client2 tương tự nhưng có thể chỉ truy cập cố định vào bản ghi B (hoặc A) nó không giống như ELB là thay đổi khi refresh page.
    - Có thể sử dụng health check 
    - Lên tới 8 bản Records cho mỗi multi-Value query
    - Multi-Value không phải là một thay thế cho ELB
  - Geoproximity (sử dụng tính năng Route 53 Traffic Flow)
    -  Sử dụng để kéo traffic từ resource region này sang resource region kia
    -  Việc truy cập nhiều traffic hay không phụ thuộc vào chỉ số Bias, chỉ số Bias region nào càng cao thì traffic sẽ truy cập vào region đó càng nhiều và ngược lại (chỉ số được set từ 1 - 99)
    -  AWS Resource (AWS region)
    -  non AWS Resource (chỉ định kinh độ và vĩ độ) 

- Traffic flow:  Nó giống như 1 lược đồ hiển các Routing Policy bằng việc kéo thả, sắp xếp chúng kết nối với nhau nhằm mục đích cấu hình hệ thống Record policy phức tạp
- Có thể dùng Route 53 để quản lý domain mua ở trang khác như Godaddy....bằng viecj khai báo DNS name của Route 53 Amazon trong phần DNS management của Godaddy từ đó Godaddy sẽ redirect sang Route 53

<img src="https://i.imgur.com/LPeuk16.png">
