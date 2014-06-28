saltstack
=========

Hướng dẫn cài đặt saltstack

Saltstack là gì?

Saltstack là 1 phần mềm quản lý cấu hình. Nói đơn giản là 1 phần mềm giúp bạn chỉnh sửa file cấu hình trên 1 máy và áp dụng file cấu hình này trên hàng loạt máy. Ngoài quản lý cấu hình thì saltstack còn quản lý cron/file/folders/service... rất nhiều thứ hay ho.

Áp dụng vào việc triển khai Openstack thì khi bạn phải setup 1 hệ thống hàng chục (thậm chí hàng trăm server) trong đó có 1 (hoặc 1 vài) Controller và số còn lại là Compute Node. Cách truyền thống là bạn lên từng node gõ lóc cóc từng lệnh và cứ thế làm từng server 1. Tiên tiến hơn chút thì bạn viết 1 script (mình thích bash shell) để chạy tự động từ đầu tới cuối, cách này nhanh hơn rất nhiều cách đầu nhưng bạn vẫn phải copy script lên từng server và gõ từ lệnh. Và sau này khi trong quá trình hoạt động, bạn cần update 1 vài cấu hình nhỏ trong file nova.conf thì bạn phải lên từng Compute và sửa, công việc nhàm chán. Trường hợp này bạn có thể sử dụng saltstack, bạn chỉ ngồi 1 chỗ gõ command 'ra lệnh' cho các server phải cài đặt những gói này, cài dịch vụ này trước rồi tới dịch vụ kia, đảm bảo các dịch vụ running. Và khi bạn chỉnh sửa 1 vài cấu hình thì sau khi chỉnh sửa bạn lại 'ra lệnh' cho các server lên 'kéo' các file tương ứng về và restart lại dịch vụ, tất cả đều tự động. Rất khỏe :))

Saltstack có 2 thành phần
- Salt Master: quản lý cấu hình. Bạn chỉnh sửa cấu hình trên này (1 lần duy nhất), chính Salt Master sẽ 'ra lệnh' cho các server làm việc ^^
- Salt Minion: 'nhận lệnh' từ Salt Master và thực thi 'lệnh' dựa vào cấu hình tương ứng.

j/k: Thường thì người ta hay đặt master và slave. Tác giả viết nên cái này chắc cũng ghiền phim Despicable Me nên đặt là minion  :v

Điểm đặc biệt là mỗi Salt Minion sẽ có những cấu hình khác nhau, do đó sẽ thực hiện những công việc khác nhau.
ể
Ví dụ: Mô hình có 3 server:
SvrA: Saltmaster
SvrB + SvrC: SaltMinion

Người quản trị có th cấu hình cho SvrB cài đặt Apache trong khi SvrC cài đặt mysql + php.

Thêm 1 điểm đáng lưu ý nữa là người quản trị ko cần quan tâm các Minion đang cài đặt Distro nào, phiên bản bao nhiêu. Bạn chỉ cần ra lệnh kiểu như:
ServB cài đặt Apache HTTP Server và đảm bảo dịch vụ này running
ServC cài đặt PHP và MySQL.

Nói dông dài thế đủ rồi. Bây giờ bắt tay vào cài đặt.

Mô hình LAB có 3 server
Server 1:
Hostname: saltmaster
eth0: 10.20.0.100
eth1: Internet Access (để tải gói về cài đặt)
OS: Ubuntu 14.04 amd64

Server 2:
Hostname: minion1
eth0: 10.20.0.150
eth1: Internet Access
OS: Ubuntu 12.04 amd64

Server 3:
Hostname: minion2
eth0: 10.20.0.151
eth1: Internet Access
OS: CentOS 6.5 X86_64


Cài đặt saltmaster
apt-get update && apt-get upgrade -y
apt-get install python-software-properties -y
add-apt-repository ppa:saltstack/salt -y
apt-get update
apt-get install salt-master -y

Cấu hình Saltmaster
/etc/salt/master

# saltmaster sẽ listen trên tất cả các IP
interface: 0.0.0.0

# Saltmaster sẽ đọc các file cấu hình ở /srv/salt
file_roots:
  base:
    - /srv/salt

Restart lại Saltmaster
/etc/init.d/salt-master restart

