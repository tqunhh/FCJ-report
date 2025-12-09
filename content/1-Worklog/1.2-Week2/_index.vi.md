---
title: "Worklog Tuần 2"
date: "2025-01-01"
weight: 1
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu tuần 2:

* Hiểu vai trò của Route 53.
* Triển khai AWS Backup.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | - Tìm hiểu khái niệm Hybrid DNS <br> - Thiết lập cơ sở hạ tầng DNS <br>&emsp; + Tạo route 53 Outbound Endpoint <br>&emsp; + Tạo Route 53 Inbound Endpoint <br>&emsp; + Cấu hình Resolver Rules                                                                                | 15/09/2025   | 15/09/2025      | https://000010.awsstudygroup.com/vi/
| 3   | - Tìm hiểu và thiết lập VPC Peering <br> -  Hiểu lý do dùng peering thay vì NAT/Internet gateway <br> - Tạo template CloudFormation, Security Group, EC2 <br> - Cập nhật Network ACL <br> - Tạo kết nối peering và cập nhật route tables                                     | 16/09/2025   | 16/09/2025      | https://000019.awsstudygroup.com/vi/ |
| 4   | - Tìm hiểu về khái niệm và ưu điểm của Transit Gateway <br> - **Thực hành:** <br>&emsp; + Tạo Transit Gateway <br>&emsp; + Transit Gateway Attachments <br>&emsp; + Transit Gateway Route Tables <br>&emsp; + Thêm Transit Gateway Routes vào VPC Route Tables  | 17/09/2025   | 17/09/2025      | <https://000020.awsstudygroup.com/vi/> |
| 5   | - Tìm hiểu cách tạo, quản lý, lưu trữ, tự động hóa và mở rộng EC2 Instance trên AWS <br> - Phân biệt giữa EBS volume và Instance Store <br> - Cách tự động cấu hình EC2 khi khởi tạo bằng User Data Script <br> - Tìm hiểu cách triển khai hệ thống auto scaling <br>                  | 18/09/2025   | 18/09/2025      |  |
| 6   | - **Thực hành:** <br>&emsp; + Tạo S3 bucket để lưu dữ liệu backup <br>&emsp; + Triển khai hạ tầng <br>&emsp; + Tạo Backup Plan                                                                        | 19/09/2025   | 19/09/2025      | <https://000013.awsstudygroup.com/vi/> |


### Kết quả đạt được tuần 2:

* Hiểu khái niệm Hybrid DNS:  
  * Vai trò của Route 53 Resolver khi xử lý tên miền nội & ngoại
  * Các thành phần trong DNS hybrid: Outbound Endpoint, Inbound Endpoint, Resolver Rules

* Phân biệt hai loại storage:
  * EBS (Elastic Block Store) – ổ đĩa ảo gắn với instance.
  * Instance Store – ổ đĩa tạm, mất khi tắt máy.

* Transit Gateway hoạt động như “router trung tâm” giữa nhiều VPC (và có thể on-prem)

* Cách tạo Amazon S3 Bucket để lưu trữ dữ liệu trên cloud.
* Thiết lập Bucket Policy, Permissions, và Public Access Block.

* Triển khai AWS Backup:
  * Tạo Backup Vault để lưu trữ bản sao lưu
  * Tạo Backup Plan và lên lịch tự động sao lưu.
  * Gán tài nguyên (EC2, EBS, RDS, EFS, …) vào kế hoạch backup
  * Kiểm tra, thử khôi phục dữ liệu đã sao lưu.


