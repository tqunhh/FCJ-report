---
title: "Worklog Tuần 3"
date: "2025-01-01"
weight: 1
chapter: false
pre: " <b> 1.3. </b> "
---


### Mục tiêu tuần 3:

* Tìm hiểu khái niệm các dịch vụ lưu trữ 
* Thực hành tạo Amazon 

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | - Triển khai File Storage Gateway <br> - Thiết lập File Sharing <br> - **Thực hành:** <br>&emsp; + Tạo Storage Gateway <br>&emsp; + Tạo các File Shares qua gateway <br>&emsp; + Kết nối File Shares từ máy On-premise (Windows, Linux) qua SMB/NFS                                                                                  | 22/09/2025   | 22/09/2025      | <https://000024.awsstudygroup.com/vi/> |
| 3   |**Thực hành:** <br>&emsp; + Tạo S3 bucket <br>&emsp; + Tải dữ liệu lên bucket <br>&emsp; + Bật tính năng website tĩnh <br>&emsp; + Cấu hình Block Public Access <br>&emsp; + Cấu hình Public Object <br>&emsp; + Kiểm tra website        | 23/09/2025   | 23/09/2025      | <https://000057.awsstudygroup.com/vi/> |
| 4   |**Thực hành:** <br>&emsp; + Tăng tốc bằng CloudFront <br>&emsp; + Chặn truy cập công cộng trực tiếp tới S3 <br> &emsp; + Cấu hình CloudFront distribution để phục vụ file <br> &emsp; + Quản lý version — khôi phục nếu xóa nhầm <br> &emsp; + Di chuyển Object <br> &emsp; + Sao chép Object sang region khác   | 24/09/2025   | 24/09/2025      | <https://000057.awsstudygroup.com/vi/> |
| 5   | - Tổng quan các dịch vụ lưu trữ của AWS <br>&emsp; + Amazon S3 <br>&emsp; + EBS (Elastic Block Store) <br>&emsp; + EFS (Elastic File System) <br>&emsp; + Storage Gateway <br>&emsp; + Snow Family               | 25/09/2025   | 25/09/2025      | |
| 6   | - Tìm hiểu cấu trúc chính của bucket, object và object key trong S3 <br> - Triển khai website tĩnh bằng S3 <br> - Kiểm soát truy cập qua Bucket Policy, ACL, IAM <br> - Giới thiệu S3 Glacier <br> - Tìm hiểu Snow Family                                                                               | 26/09/2025   | 25/09/2025      | |


### Kết quả đạt được tuần 3:

* Triển khai File Storage Gateway: 
  * Biết cách vận hành Storage Gateway để tích hợp lưu trữ file on-premise với S3
  * Thiết lập file share qua SMB / NFS qua gateway
  * Thực hành kết nối từ máy client tới gateway

* Thực hành với Amazon S3:
  * Biết tạo và cấu hình S3 bucket, upload/download dữ liệu.
  * Thiết lập quyền truy cập và host website tĩnh trên S3.
  * Biết các công cụ sao lưu và di chuyển dữ liệu trong AWS.

* Hiểu khái niệm và sự khác nhau giữa các dịch vụ lưu trữ:
  * Amazon S3: lưu trữ đối tượng (object storage).
  * EBS: lưu trữ khối cho EC2.
  * Instance Store: lưu trữ tạm thời trên ổ cứng vật lý của máy chủ EC2.
  * Amazon EFS – hệ thống file chia sẻ
  * AWS Backup, Storage Gateway, Snow Family: sao lưu và di chuyển dữ liệu.
  


