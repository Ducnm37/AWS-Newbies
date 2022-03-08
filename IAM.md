# Giới thiệu về IAM

## AWS Secure Token Service

- Cho phép gán quyền giới hạn và truy cập tạm thời cho AWS resource
- Token có giá trị lên tới 1 giờ (phải làm mới lại)
- Assume Role:
  - Bảo mật nâng cao cho account
  - Cross Account access cho phép role chọn tartget account active
- Assume Role với SAML: Trả lại credential cho user logged với SAML
- Assume Role với Webldentity:
  - Trả về credential cho user logged với 1 identity provider  (facebook login, google login...)
  - AWS khuyến cáo sử dụng Cognito
- Get sesion token: cho MFA từ 1 user hoặc AWS account root user
- Chứng chỉ tạm thời có giá trị từ 15 phút đến 1 giờ

<img src="https://i.imgur.com/FqxSGyk.png">

## Identity Federation & Cognito

- Federation cho phép người dùng bên ngoài tạm thời truy cập vào AWS resource
- user được phép truy cập qua identity role được cung cấp
- Có thể sử dụng mobile user account và cung cấp cho họ IAM permission có thể truy cập vào own resource
- Federation có thể có nhiều flavor:
  - SAML 2.0
  - Custom identity broker
  - Web Identity Federation với AWS Cognito
  - Web Identity Federation không cần AWS Cognito
  - Single Sign On
  - Non-SAML với AWS Microsoft AD
- Sử dụng federation không cần tạo IAM user

<img src="https://i.imgur.com/rbWlcGF.png">

- Để tích hợp Active Directory / ADFS với AWS (hoặc mọi SAML Security Assertion Markup Language 2.0)
- Cung cấp truy cập tới AWS console hoặc CLI (thông qua credential tạm thời)
- Không cần tạo IAM user cho mỗi nhân viên

<img src="https://i.imgur.com/ptQEu2c.png">
--
<img src="https://i.imgur.com/j7JPqCp.png">

- Active Directory FS

<img src="https://i.imgur.com/Ii2YEAj.png">

## SAML 2.0 Federation

- SAML 2.0 cần để setup 1 trust giữa AWS IAM và SAML
- SAML 2.0 enable web-based, cross domain SSO
- Sử dụng STS API: AssumeRole với SAML
- Cách cũ là federation thông qua SAML
- Cách mới là sử dụng Amazon Single Sign On (SSO) Federation quản lý
- Web identity federation

<img src="https://i.imgur.com/OIq8xqq.png">

- Khi ứng dụng người dùng chạy on-premise muốn xác thực AWS mà không hỗ trợ SAML 2.0 thì ta có thể dùng Custom Identity Broker để xác thực user và lấy chứng chỉ tạm thời từ AWS STS sử dụng AssumeRole hoặc GetFederationToken STS API call


## AWS Cognito

- Cho phép truy cập trực tiếp tới AWS resource từ client thông qua mobile, webapp

<img src="https://i.imgur.com/TYm6pQW.png">

  - login vào identity provider
  - Get temporary AWS credential từ Federate identity pool
  - Credential để xác định IAM policy 

## AWS AD

- AWS AD và on-premise AD có thể  xác thực user của nhau bằng cách tạo 1 kết nối trust connection
- AD connector dùng để redirect proxy user tới  on-premise AD xác thực
- Simple AD sử dụng khi không có 1 on-premise AD, nó sẽ là AD cho AWS.

<img src="https://i.imgur.com/ggJ5dZA.png">

- Organizational Unit

  <img src="https://i.imgur.com/2GTD3p4.png">

  <img src="https://imgur.com/Mlg7vXA">
  
- Service Control Policies (SCP):
  - White list hoặc black list account
  - Apply OU hoặc Account level
  - Không apply được master account
  - SCP có thể apply tới all user và role bao gồm cả root cho account
  - SCP không thể apply service-link role (ví dụ 1 service AWS link với AWS Orgnazation)
  - Mặc định SCP không apply
  - Các rule của SCP cần apply rõ ràng
  
  <img src="https://i.imgur.com/Qq7J55z.png">
  
- Migrate Account

  <img src="https://i.imgur.com/H2xziax.png">
  
 ## IAM role vs IAM resource policies
 
 - Với role thì khi 1 user được assign nó sẽ bỏ qua original permission và ăn theo role mới
 - Với resource policy khi user được assign nó sẽ vẫn giữa được original permission và thêm quyền của policy mới
 - Hiểu đơn giản resource policy là quyền bao phủ, còn role là quyền chi tiết

## IAM Boundary

- Giả sử User A được Add permission policy là AdministratorAccess: có nghĩa là User A có quyền như root làm mọi thứ. Nhưng User A sau đó lại add thêm Permission Boundary là AmazonS3FullAccess, vậy cuối cùng user A chỉ có thể access full quyền trên S3 và restrict nhưng quyền còn lại

- Có thể kết hợp với AWS Organizational SCP (Service control policies) 

<img src="https://i.imgur.com/hKwwbsF.png">

## Biểu đồ logic IAM policy 

<img src="https://i.imgur.com/6p3m51H.png">

## AWS Resource Access Manager (RAM)

- Share tài nguyên AWS từ 1 account đang quản lý cho 1 account khác nữa dùng chung
- Mỗi account không thể view, modified, delete, resource khác từ account khác
- Mọi thứ deploy trong VPC có thể nói với resource khác trong VPC biết

## AWS Single Sign-On (SSO)

- SSO sử dụng login với ứng dụng bên thứ 3 cho phép truy cập vào resource AWS
- Tích hợp với AD
  
  <img src="https://i.imgur.com/BrVw6M1.png">

- SSO vs SAML

  <img src="https://i.imgur.com/56eqiXp.png">




