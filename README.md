![Hướng dẫn cài đặt saltstack](http://static.squarespace.com/static/524cf70fe4b05018590c3fb3/t/52718a36e4b079ec23e2c895/1383172756774/SaltStack%20logo%20-%20black%20on%20white.png)
=========

# 1. Nhu cầu ?
Khi triển khai Openstack thì khi bạn phải setup 1 hệ thống hàng chục (thậm chí hàng trăm server) trong đó có 1 (hoặc 1 vài) Controller và số còn lại là Compute Node. Cách truyền thống là bạn lên từng node gõ lóc cóc từng lệnh và cứ thế làm từng server 1. 'Pro' hơn chút nữa thì bạn viết 1 script (mình hay dùng bash shell) để chạy tự động từ đầu tới cuối, cách này nhanh hơn cách đầu nhưng bạn vẫn phải copy script lên từng server và gõ lệnh thực thi script đó. Sau này khi trong quá trình hoạt động, bạn cần update 1 vài cấu hình nhỏ (Ví dụ nova.conf) thì bạn phải lên từng Compute Node và sửa, công việc nhàm cmn chán. Trường hợp này bạn có thể sử dụng saltstack (puppet/chef hoặc bất cứ phần mềm quản lý cấu hình nào đó), bạn chỉ ngồi 1 chỗ gõ command 'ra lệnh' cho các server phải cài đặt những gói này, cài dịch vụ này trước rồi tới dịch vụ kia, đảm bảo các dịch vụ sẽ khởi động cùng OS khi server bị reset. Và trường hợp bạn chỉnh sửa 1 vài cấu hình thì sau khi chỉnh sửa bạn lại 'ra lệnh' cho các server lên 'kéo' các file tương ứng về và restart lại dịch vụ, tất cả đều tự động. Rất khỏe :))

Thêm 1 điểm đáng cộng nữa khi sử dụng hệ thống quản lý cấu hình là người quản trị ko cần quan tâm các Server cài đặt Distro gì, phiên bản bao nhiêu, kiến trúc nào. Bạn chỉ cần ra lệnh kiểu như:
- Thằng A mày cài cho tao Apache
- Thằng B mày cài cho tao PHP + MySQL
- .....

# 2. Tại sao chọn Saltstack?
Trên Internet hiện nay có nhiều phần mềm quản lý cấu hình tập trung như puppet/chef/saltstack... Mỗi phần mềm có những thành phần, ưu nhược điểm và có cộng đồng người dùng riêng. Trong đó đông đảo nhất là puppet. Số lượng người sử dụng puppet đông đảo ko hẳn là vì puppet hơn hẳn những thằng còn lại mà 1 phần vì nó ra lâu rồi.

Cá nhân mình đã thử tìm hiểu qua puppet và saltstack, và cuối cùng mình chọn saltstack vì 1 số  lí do sau:
- Miễn phí : puppet cũng có bản miễn phí (community) nhưng bản có 1 hạn chế mà mình ko thích là các máy slave phải định kỳ contact master để 'kéo' các cấu hình mới về. Việc này có thể làm được thông qua cấu hình cronjob nhưng có đôi lúc mình chỉnh sửa mà muốn áp ngay thì ko được. Còn riêng saltstack thì khi chỉnh sửa cấu hình xong, bạn ngồi tại Master 'đẩy' cấu hình tương ứng về cho từng Slave (minion)
- Kiến trúc đơn giản: cá nhân mình thấy saltstack tổ chức 'cây cấu hình' đơn giản, dễ hiểu hơn puppet (hoặc do mình dốt quá nên cảm thấy khó hiểu)
- Còn nhiều nữa mà mình chưa nghĩ ra, khi nào nghĩ ra mình cập nhật sau vậy ;))

# 3. Thành phần của Saltstack
- **Salt Master:** quản lý cấu hình. Bạn chỉnh sửa cấu hình trên này (1 lần duy nhất), chính Salt Master sẽ 'ra lệnh' cho các server làm việc ^^
- **Salt Minion:** 'nhận lệnh' từ Salt Master và thực thi 'lệnh' dựa vào cấu hình tương ứng.

**j/k:** Thường thì người ta hay đặt master và slave. Bác lập trình viên viết Saltstack chắc cũng ghiền phim Despicable Me nên đặt là minion :v

# 4. Các bước cài đặt ?
### 4.1 Mô hình LAB
----------------

Server	|saltmaster
--------|---------
**eth0**    |10.20.0.100
**eth1**    | Internet Access (để tải gói về cài đặt)
**OS**      | Ubuntu 14.04 amd64

Server  | minion1
--------|-------------
**eth0**			| 10.20.0.150
**eth1**			| Internet Access
**OS**				| Ubuntu 12.04 amd64

Hostname	| minion2
----------|--------
**eth0**			| 10.20.0.151
**eth1**			| Internet Access
**OS**				| CentOS 6.5 X86_64

### 4.2 Cài đặt saltmaster
----------------
```shell
apt-get update && apt-get upgrade -y
apt-get install python-software-properties -y
add-apt-repository ppa:saltstack/salt -y
apt-get update
apt-get install salt-master -y
```

* Cấu hình Saltmaster: **/etc/salt/master**
```shell
# saltmaster sẽ listen trên tất cả các IP
interface: 0.0.0.0

# Saltmaster sẽ đọc các file cấu hình ở /srv/salt
file_roots:
  base:
    - /srv/salt
```

* Restart lại Saltmaster
```base 
/etc/init.d/salt-master restart
```
### 4.3 Cài đặt Salt Minion
----------------
**Minion 1**
```shell
apt-get install salt-minion -y
```
**Minion 2**
```shell
rpm -Uvh http://ftp.linux.ncsu.edu/pub/epel/6/i386/epel-release-6-8.noarch.rpm
yum install salt-minion -y
```

* Cấu hình Minion: **/etc/salt/minion**
```shell
# Chỉ cho minion biết phải đi đâu để nhận 'nhiệm vụ' 
master: 10.20.0.100
```
* Restart Minion
```shell
/etc/init.d/salt-minion restart
```
### 4.4 Kiểm tra quá trình cài đặt
----------------
* **Trên Master**
```shell
salt-key -L
```
- **Nếu thấy kết quả như vầy thì minion đã gởi public key tới cho Master**
```shell
Accepted Keys:
Unaccepted Keys:
minion1
minion2
Rejected Keys:
```

Key này được Master sử dụng để chứng thực các minion (còn có sử dụng để mã hóa dữ liệu qua lại giữa 2 thằng này ko thì ko sure lắm)

Kết quả trên có thể hiểu như thế này: Mới có 2 thằng Minion gởi key lên và đang chờ accept, bây giờ mày có muốn accept hay ko? Ờ, Accept thôi
```shell
salt-key -A
```
*(Nó có hỏi gì thì chọn Y hết nha bà con)*

- **Kiểm tra lại phát**
```shell
salt-key -L
```
```shell
Accepted Keys:
minion1
minion2
Unaccepted Keys:
Rejected Keys:
```

- **Kiểm tra xem Master mà Minion đã 'thông' chưa**

*Trên master gõ lệnh*
```shell
salt '*' test.ping
```
```shell
minion2:
    True
minion1:
    True
```
- **Lấy thông tin về 'Dòng họ' OS các minion đang xài**
```shell
salt '*' grains.item os_family
```
```shell
minion2:
  os_family: RedHat
minion1:
  os_family: Debian
```

Mình xin kết thúc bài này tại đây. Bài này chỉ với 1 mục đích duy nhất là giúp các bạn cài đặt được Salt-Master và Salt-Minion. Ngoài ra còn có 1 vài cấu hình cơ bản để Master và Minion liên lạc được với nhau.

Trong bài viết có 1 số command mình chưa giải thích rõ ràng vì 1 là bản thân mình cũng chưa tìm hiểu kỹ về saltstack (mới tìm hiểu có 1 tuần :(), 2 là để các bạn tự nghiên cứu ;)). Nếu có gì cần trao đổi các bạn có thể vào FB: https://www.facebook.com/cucxabong
