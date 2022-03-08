# ElasticCache 

- Sử dụng để quản lý redis hoặc memcached
- Cache là in-memory tăng hiệu xuất và giảm độ trễ, giảm tải cho database
- Dữ liệu sẽ đọc và ghi dữ liệu vào ElasticCache trước sau đó sẽ flush xuống database
- Tất cả các cache trên Elasticache không hỗ trợ IAM authenticattion
- IAM policy chỉ sử dụng cho AWS API-level security
- Redis auth: Có thể sử dụng tạo token khi tạo 1 Redis cluster
  - Hỗ trợ bảo mật đường truyền SSL
- Memcached: Hỗ trợ SASL-based authentication (advanced)

<img src="https://i.imgur.com/W3rJ7ph.png">

- Cơ chế lưu sesion:
  - User login vào Apps, và Apps sẽ lưu sesion vào ElasticCache
  - Khi User truy cập vào Apps trên instance khác thì instance sẽ lấy lại data user login trên sesion đã lưu ở ElasticCache
  
  <img src="https://i.imgur.com/8G3v2ca.png">

- Cơ chế đọc ghi vào cache
  - Lazy loading: khi ứng dụng không đọc từ cache mà đọc từ database trước sau đó ứng dụng mới ghi data vào Elasticache sau đó mới load data từ Elasticache
  - Write through: Ứng dụng ghi vào Elasticache trước sau đó mới ghi vào database
  - Sesion store: Lưu tạm thời sesion vào cache sử dụng tính năng TTL (time to live)
  
  <img src="https://i.imgur.com/tvOwUJs.png">


ElasticCache so sánh Redis và Memcached

### Redis

- Multi AZ với Auto failover
- Có thể mở rộng Read replicas
- High Availability
- Backup và restore
- Dữ liệu bền vững với AOF persistence  (Append Only File một kỹ thuật ghi dữ liệu của redis)

### Memcached

- Multi node cho partition data sharing
- No HA
- Non persistent
- No backup & restore
- Multi threaded
- Phù hợp với việc chia sẻ data


--------------------------------------------------------------------------
--------------------------------------------------------------------------

# Tìm hiểu qua về redis

Sự ra đời của Redis
Câu chuyện bắt đầu khi tác giả của Redis, Salvatore Sanfilippo (nickname: antirez), cố gắng làm những công việc gần như là không thể với SQL Database!

Server của antirez nhận 1 lượng lớn thông tin từ nhiều trang web khác nhau thông qua JavaScript tracker, lưu trữ n page view cho trừng trang và hiển thị chúng theo thời gian thực cho user, kèm theo đó là lưu trữ 1 lượng nhỏ lịch sử hiển thị của trang web. Khi số lượng page view tăng đến hàng nghìn page trên 1 giây, antirez không thể tìm ra cách tiếp cận nào thực sự tối ưu cho việc thiết kế database của mình. 

Tuy nhiên, anh ta nhận ra rằng, việc lưu trữ 1 danh sách bị giới hạn các bản ghi không phải là vấn đề quá khó khăn. Từ đó, ý tưởng lưu trữ thông tin trên RAM và quản lý các page views dưới dạng native data với thời gian pop và push là hằng số được ra đời. Antirez bắt tay vào việc xây dựng prototype bằng C, bổ sung tính năng lưu trữ thông tin trên đĩa cứng và… Redis ra đời.

### 1. Redis là gì?

- Ngày nay, khái niệm NoSQL trở nên không còn xa lạ trong giới Công Nghệ Thông Tin (CNTT). Đi kèm với đó là sự ra đời của hàng loạt hệ quản trị cơ sở dữ liệu (DBMS) phát triển dựa trên đặc thù của NoSQL: Non-relational (không quan hệ), Distributed (phân tán), Open-source (mã nguồn mở), Horizontally scalable (dễ dàng mở rộng theo chiều ngang).

- Redis là cơ sở dữ liệu mang phong cách NoSQL, lưu trữ dữ liệu với dạng KEY-VALUE với nhiều tính năng được sử dụng rộng rãi. Nó có thể hỗ trợ nhiều kiểu dữ liệu như: strings, hashes, lists, sets, sorted. Đồng thời có thể cho phép scripting bằng ngôn ngữ Lua.

- NoSQL là một khái niệm chỉ về một lớp các hệ cơ sở dữ liệu không sử dụng mô hình quan hệ (RDBMS). RDBMS vốn tồn tại khá nhiều nhược điểm như có hiệu năng không tốt nếu kết nối dữ liệu nhiều bảng lại hay khi dữ liệu trong một bảng là rất lớn.

- Thuật ngữ NoSQL đánh dấu bước phát triển của thế hệ CSDL mới: phân tán (distributed) + không ràng buộc (non-relational).
Đặc chưng cơ bản của Redis

### 2. Master/Slave Replication
Đây không phải là đặc trưng quá nổi bật, các DBMS khác đều có tính năng này, tuy nhiên chúng ta nêu ra ở đây để nhắc nhở rằng, Redis không kém cạnh các DBMS về tình năng Replication.


### 3. In-memory
Không như các DBMS khác lưu trữ dữ liệu trên đĩa cứng, Redis lưu trữ dữ liệu trên RAM, và đương nhiên là thao tác đọc/ghi trên RAM. Với người làm CNTT bình thường, ai cũng hiểu thao tác trên RAM nhanh hơn nhiều so với trên ổ cứng, nhưng chắc chắn chúng ta sẽ có cùng câu hỏi: Điều gì xảy ra với data của chúng ta khi server bị tắt?

Rõ ràng là toàn bộ dữ liệu trên RAM sẽ bị mất khi tắt server, vậy làm thế nào để Redis bảo toàn data và vẫn duy trì được ưu thế xử lý dữ liệu trên RAM. Chúng ta sẽ cùng tìm hiểu về cơ chế lưu dữ liệu trên ổ cứng của Redis trong phần tiếp theo của bài viết.

### 4. Persistent redis

Dù làm việc với data dạng key-value lưu trữ trên RAM, Redis vẫn cần lưu trữ dữ liệu trên ổ cứng. 1 là để đảm bảo toàn vẹn dữ liệu khi có sự cố xảy ra (server bị tắt nguồn) cũng như tái tạo lại dataset khi restart server, 2 là để gửi data đến các slave server, phục vụ cho tính năng replication. Redis cung cấp 2 phương thức chính cho việc sao lưu dữ liệu ra ổ cứng, đó là RDB và AOF. 

**RDB (Redis DataBase file)**
- Cách thức làm việc**
  - RDB thực hiện tạo và sao lưu snapshot của DB vào ổ cứng sau mỗi khoảng thời gian nhất định.

- Ưu điểm
  - RDB cho phép người dùng lưu các version khác nhau của DB, rất thuận tiện khi có sự cố xảy ra.
  - Bằng việc lưu trữ data vào 1 file cố định, người dùng có thể dễ dàng chuyển data đến các data centers khác nhau, hoặc chuyển đến lưu trữ trên Amazon S3.
  - RDB giúp tối ưu hóa hiệu năng của Redis. Tiến trình Redis chính sẽ chỉ làm các công việc trên RAM, bao gồm các thao tác cơ bản được yêu cầu từ phía client như thêm/đọc/xóa, trong khi đó 1 tiến trình con sẽ đảm nhiệm các thao tác disk I/O. Cách tổ chức này giúp tối đa hiệu năng của Redis. Khi restart server, dùng RDB làm việc với lượng data lớn sẽ có tốc độ cao hơn là dùng AOF.

- Nhược điểm
  - RDB không phải là lựa chọn tốt nếu bạn muốn giảm thiểu tối đa nguy cơ mất mát dữ liệu. Thông thường người dùng sẽ set up để tạo RDB snapshot 5 phút 1 lần (hoặc nhiều hơn). Do vậy, trong trường hợp có sự cố, Redis không thể hoạt động, dữ liệu trong những phút cuối sẽ bị mất.
  - RDB cần dùng fork() để tạo tiến trình con phục vụ cho thao tác disk I/O. Trong trường hợp dữ liệu quá lớn, quá trình fork() có thể tốn thời gian và server sẽ không thể đáp ứng được request từ client trong vài milisecond hoặc thậm chí là 1 second tùy thuộc vào lượng data và hiệu năng CPU.

<ỉg src="https://i.imgur.com/m6vkrvU.png">

**AOF (Append Only File)**
- Cách thức làm việc:
  - AOF lưu lại tất cả các thao tác write mà server nhận được, các thao tác này sẽ được chạy lại khi restart server hoặc tái thiết lập dataset ban đầu.

- Ưu điểm
  - Sử dụng AOF sẽ giúp đảm bảo dataset được bền vững hơn so với dùng RDB. Người dùng có thể config để Redis ghi log theo từng câu query hoặc mỗi giây 1 lần.
  - Redis ghi log AOF theo kiểu thêm vào cuối file sẵn có, do đó tiến trình seek trên file có sẵn là không cần thiết. Ngoài ra, kể cả khi chỉ 1 nửa câu lệnh được ghi trong file log (có thể do ổ đĩa bị full), Redis vẫn có cơ chế quản lý và sửa chữa lối đó (redis-check-aof).
  - Redis cung cấp tiến trình chạy nền, cho phép ghi lại file AOF khi dung lượng file quá lớn. Trong khi server vẫn thực hiện thao tác trên file cũ, 1 file hoàn toàn mới được tạo ra với số lượng tối thiểu operation phục vụ cho việc tạo dataset hiện tại. Và 1 khi file mới được ghi xong, Redis sẽ chuyển sang thực hiện thao tác ghi log trên file mới.

- Nhược điểm
  - File AOF thường lớn hơn file RDB với cùng 1 dataset.
  - AOF có thể chậm hơn RDB tùy theo cách thức thiết lập khoảng thời gian cho việc sao lưu vào ổ cứng. Tuy nhiên, nếu thiết lập log 1 giây 1 lần có thể đạt hiệu năng tương đương với RDB.
  - Developer của Redis đã từng gặp phải bug với AOF (mặc dù là rất hiếm), đó là lỗi AOF không thể tái tạo lại chính xác dataset khi restart Redis. Lỗi này chưa gặp phải khi làm việc với RDB bao giờ.
  - Câu hỏi đặt ra là, chúng ta nên dùng RDB hay AOF? Mỗi phương thức đều có ưu/nhược điểm riêng, và có lẽ cần nhiều thời gian làm việc với Redis cũng như tùy theo ứng dụng mà đưa ra lựa chọn thích hợp. Nhiều người chọn AOF bới nó đảm bảo toàn vẹn dữ liệu rất tốt, nhưng Redis developer lại khuyến cáo nên dùng cả RDB, bởi nó rất thuận tiện cho việc backup database, tăng tốc độ cho quá trình restart cũng như tránh được bug của AOF.
  - Cũng cần lưu ý thêm rằng, Redis cho phép không sử dụng tính năng lưu trữ thông tin trong ổ cứng (không RDB, không AOF), đồng thời cũng cho phép dùng cả 2 tính năng này trên cùng 1 instance. Tuy nhiên khi restart server, Redis sẽ dùng AOF cho việc tái tạo dataset ban đầu, bới AOF sẽ đảm bảo không bị mất mát dữ liệu tốt hơn là RDB.
