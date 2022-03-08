# Create Auto Scaling Group
## 1. Create launch teamplate

<img src="https://i.imgur.com/gl8V4h2.png">

<img src="https://i.imgur.com/UxgIxMD.png">

```
#!/bin/bash
yum update -y
yum install httpd -y
systemctl start httpd
systemctl enable httpd
echo "<h1>hello, Surprise motherfucker $(hostname -f)</h1>" > /var/www/html/index.html
```

## 2.Create Scaling Group

- Trước đó cần tạo target group và loadbalancer

<img src="https://i.imgur.com/4I0Y5BZ.png">

<img src="https://i.imgur.com/3QsP4cN.png">

Gắn target group và enable ELB (Elastic load balancer)

<img src="https://i.imgur.com/svoVUwq.png">

<img src="https://i.imgur.com/TGXPBv9.png">


## 3. Policy Auto Scaling Group

- Giúp tracking resource theo polocy bạn define, ví dụ nếu CPU quá 40% sẽ tự động scale thêm 1 instance

<img src="https://i.imgur.com/VkbMmB7.png">

<img src="https://i.imgur.com/0uS6e4Q.png">

Bạn có thể check trên cloudwatch 
<img src="https://i.imgur.com/VsEvjFn.png">

- Ví dụ 2 AZ, west-1a có 3 instance và west-1b có 4 instance thì (Default Termination Policy cho Auto Scaling Group là sẽ terminate instance nào oldest template config) ACG sẽ teminate 1 instance của west-1b có oldest version config nhất để cân bằng các AZ
- Default việc khởi tạo lại hoặc khởi tạo thêm instance của Auto Scaling Group có chu kỳ 5 phút 1 lần (300s)
