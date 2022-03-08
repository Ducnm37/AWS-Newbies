# AWS Security & Encryption: KMS, SSM, Cloud HSM, Shield, WAF

- Server side encryption

<img src="https://i.imgur.com/r2gmjJi.png">

- Client side encryption

<img src="https://i.imgur.com/vAO3BM2.png">

## KMS (Key management service)

- AWS manage key 
  - AWS EBS: Encrypt volume
  - AWS S3: Server side encrypt object
  - AWS Redshift: Encryption data
  - AWS RDS: Encryption data
  - AWS SSM: Parameter store
### KMS - Customer Master Key (CMK)
- Symmetric key (AES-256)
  - Single encryption key sử dụng để mã hóa và giải mã
  - AWS service tích hợp với KMS sử dụng Symmetric CMK
  - Không thể truy cập với key không mã hóa (phải call KMS API để sử dụng)
- Asymmetric (RSA & ECC key pairs)
  - Public key (encryption) và private key (decryption)
  - Sử dụng mã hóa ngoài AWS cho người dùng không thể sử dụng KMS api call

- Có thể tạo sửa xóa key và policies, có thể audit key sử dungh Cloudtrail
- 3 Phương thức sử dụng CMK
  - AWS manage service default (free)
  - User key create in KMS: 1$/month
  - User key imported (256-bit symmetric key): 1$/month
  - API call KMS (0,03$/10000 calls)
- KMS có thể hỗ trợ mã hóa lên tới 4KB per call, nếu data > 4KB sử dụng envelope encryption
- Để cho người khác access vào KMS 
  - Key policies allow user
  - IAM policies allow API call

- Không thể transmit key từ region này sang region khác mà phải mã hóa lại với key mới
- Copy snapshot cross account

<img src="https://i.imgur.com/vupgrta.png">

