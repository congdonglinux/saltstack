saltstack
=========

Hướng dẫn cài đặt saltstack

Saltstack là gì?

Saltstack là 1 phần mềm quản lý cấu hình. Nói đơn giản là 1 phần mềm giúp bạn chỉnh sửa file cấu hình trên 1 máy và áp dụng file cấu hình này trên hàng loạt máy. Ngoài quản lý cấu hình thì saltstack còn quản lý cron/file/folders/service... rất nhiều thứ hay ho.

Áp dụng vào việc triển khai Openstack thì khi bạn phải setup 1 hệ thống hàng chục (thậm chí hàng trăm server) trong đó có 1 (hoặc 1 vài) Controller và số còn lại là Compute Node. Cách truyền thống là bạn lên từng node gõ lóc cóc từng lệnh và cứ thế làm từng server 1. Tiên tiến hơn chút thì bạn viết 1 script (mình thích bash shell) để chạy tự động từ đầu tới cuối, cách này nhanh hơn rất nhiều cách đầu nhưng bạn vẫn phải copy script lên từng server và gõ từ lệnh. Và sau này khi trong quá trình hoạt động, bạn cần update 1 vài cấu hình nhỏ trong file nova.conf thì bạn phải lên từng Compute và sửa, công việc nhàm chán. Trường hợp này bạn có thể sử dụng saltstack, bạn chỉ ngồi 1 chỗ gõ command 'ra lệnh' cho các server phải cài đặt những gói này, cài dịch vụ này trước rồi tới dịch vụ kia, đảm bảo các dịch vụ running. Và khi bạn chỉnh sửa 1 vài cấu hình thì sau khi chỉnh sửa bạn lại 'ra lệnh' cho các server lên 'kéo' các file tương ứng về và restart lại dịch vụ, tất cả đều tự động. Rất khỏe :))
