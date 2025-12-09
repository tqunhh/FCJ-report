---
title: "Worklog Tuần 4"
date: "2025-01-01"
weight: 1
chapter: false
pre: " <b> 1.4. </b> "
---


### Mục tiêu tuần 4:

* Tìm hiểu Storage Gateway
* Học cách chuyển máy ảo (VM) từ môi trường on-premises sang AWS

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | - **Thực hành:** <br>&emsp; + Tạo S3 bucket để lưu dữ liệu backup <br>&emsp; + Triển khai hạ tầng <br>&emsp; + Tạo Backup Plan <br>&emsp; + Thiết lập notifications                                                                                          | 29/09/2025   | 29/09/2025      | <https://000013.awsstudygroup.com/vi/> |
| 3   | - Chuẩn bị máy ảo <br>&emsp; + cài đặt VMware Workstation Pro 17 <br>&emsp; +  tải hệ điều hành Ubuntu <br> - **Thực hành** <br>&emsp; + Export máy ảo từ On-premise <br>&emsp; + Tải máy ảo lên AWS <br>&emsp; + Import máy ảo vào AWS <br>&emsp; + Triển khai EC2 từ AMI           | 30/09/2025   | 30/09/2025      | <https://000014.awsstudygroup.com/vi/> |
| 4   | - **Thực hành:** <br>&emsp; + Thiết lập ACL cho S3 Bucket <br>&emsp; + Export máy ảo từ EC2 <br>&emsp; + Dọn dẹp tài nguyên  | 01/10/2025   | 01/10/2025      | <https://000014.awsstudygroup.com/vi/> |
| 5   | - **Thực hành:** <br>&emsp; + Tạo S3 Bucket và EC2 cho Storage Gateway  <br>&emsp; + Tạo Storage Gateway <br>&emsp; + Kết nối File shares ở On-premise         | 02/10/2025   | 02/10/2025      | <https://000024.awsstudygroup.com/vi/> |
| 6   | - Tìm hiểu Amazon FSx for Windows File Server <br> - Cách tạo file share mới trên FSx <br> - Kiểm tra hiệu năng của file share <br> - Tìm hiểu các tính năng nâng cao trong FSx            | 03/10/2025   | 03/10/2025      | <https://000025.awsstudygroup.com/> |


### Kết quả đạt được tuần 4:

* Hiểu Storage Gateway: 
  * Cấu hình File Gateway
  * Tạo file share (SMB / NFS) qua Gateway
  * Mount share từ máy client (on-premise hoặc EC2)
  * Quản lý chia sẻ file và người dùng

* Học cách chuyển máy ảo (VM) từ môi trường on-premises sang AWS để chạy trên EC2:
  * Sử dụng dịch vụ VM Import/Export của AWS. 
  * Tạo S3 Bucket & Upload File
  * Tạo IAM Role cho Import
  * Thực hiện Import
  * Hiểu quy trình tạo, upload, import, và tạo EC2 từ AMI

* Hiểu Amazon FSx for Windows File Server:
  * Cách tạo file share mới trên FSx, qua giao diện Windows hoặc công cụ quản lý share.
  * Kiểm tra hiệu năng của file share
  * Quản lý các chia sẻ file: mở file share mới, cấp quyền, quản lý user sessions, open files.



