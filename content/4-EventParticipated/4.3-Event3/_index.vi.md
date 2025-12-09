---
title: "Event 3 "
date: "2025-01-01"
weight: 1
chapter: false
pre: " <b> 4.3. </b> "
---

{{% notice warning %}}
⚠️ **Lưu ý:** Các thông tin dưới đây chỉ nhằm mục đích tham khảo, vui lòng **không sao chép nguyên văn** cho bài báo cáo của bạn kể cả warning này.
{{% /notice %}}

# Bài thu hoạch “AWS Well-Architected Security Pillar Workshop”

### Mục Đích Của Sự Kiện

* Cung cấp kiến thức tổng quan và có hệ thống về bảo mật trên AWS dựa trên 5 Security Pillars của AWS Well-Architected Framework
* Xây dựng tư duy security-first, coi bảo mật là nền tảng ngay từ khâu thiết kế kiến trúc, không phải bước bổ sung sau cùng
* Hướng dẫn các best practices thực tế về:
* Giới thiệu công cụ AI hỗ trợ development lifecycle
  * Identity & Access Management (IAM)
  * Logging, detection và continuous monitoring
  * Infrastructure và network protection
  * Data encryption, key và secrets management
  * Incident response và forensics

### Danh Sách Diễn Giả

- **Tran Duc Anh** - AWS Cloud Club Captain (SGU)
- **Tran Doan Cong Ly** - AWS Cloud Club Captain (PTIT)
- **Danh Hoang Hieu Nghi** - AWS Cloud Club Captain (HUFLIT)
- **Nguyen Tuan Thinh** - Cloud Engineer Trainee
- **Nguyen Do Thanh** - Cloud Engineer
- **Thinh Lam** - FCJer
- **Viet Nguyen** - FCJer
- **NMendel Grabski (Long)** - ex Head of Security & DevOps - Cloud Security Solution Architect
- **Quang Tinh Truongh** - AWS Community Builder - Platform Engineer at TymeX
### Nội Dung Nổi Bật

#### Identity & Access Management (IAM) – nền tảng của security
- Thiết kế IAM hiện đại với roles thay vì users
- Áp dụng nguyên tắc least privilege
- Sử dụng IAM Identity Center (SSO) cho mô hình multi-account
- Quản lý và kiểm soát quyền bằng SCP, permission boundaries và Access Analyzer

#### Detection & Continuous Monitoring

- Triển khai logging tập trung với CloudTrail, VPC Flow Logs, CloudWatch Logs
- Phát hiện mối đe dọa bằng GuardDuty và tổng hợp cảnh báo qua Security Hub
- Tự động hóa cảnh báo và phản ứng sự cố với EventBridge

#### Infrastructure Protection
- Thiết kế mạng an toàn với VPC segmentation, private subnet và VPC endpoints
- Áp dụng mô hình defense in depth sử dụng Security Groups, NACLs
- Bảo vệ nâng cao với AWS WAF, Shield và Network Firewall
- Best practices cho bảo mật EC2, ECS/EKS và container workloads
  
#### Incident Response & Automation

- Xây dựng quy trình Incident Response theo chuẩn AWS
- Thực hành các playbook thực tế: lộ khóa IAM, S3 public, EC2 bị malware
- Thu thập forensics và duy trì chain of custody
- Tự động hóa xử lý sự cố bằng Lambda, Step Functions

### Những Gì Học Được

#### Tư duy bảo mật đúng ngay từ thiết kế
- Security không phải bước bổ sung sau cùng, mà là yếu tố nền tảng trong kiến trúc cloud.
- Áp dụng security-first mindset khi thiết kế, triển khai và vận hành hệ thống trên AWS.
- Hiểu rõ mối liên hệ giữa Security – DevOps – AI/ML, trong đó security là điều kiện để hệ thống vận hành bền vững.
  
#### Kiến Trúc Kỹ Thuật

- Tất cả các thành phần được kết nối chặt chẽ và được giám sát liên tục, không phụ thuộc vào một điểm kiểm soát duy nhất.
- IAM đóng vai trò là cửa ngõ đầu tiên của toàn bộ kiến trúc.
- Áp dụng kiến trúc layered network security.


### Trải nghiệm trong event

Tham gia workshop **“AWS Well-Architected Security Pillar”** mang đến cho tôi một góc nhìn rất thực tế và toàn diện về cách xây dựng hệ thống bảo mật trên AWS theo đúng chuẩn kiến trúc. Dù chỉ diễn ra trong buổi sáng, workshop vẫn có mật độ kiến thức cao và tập trung đúng vào những vấn đề cốt lõi mà các team kỹ thuật thường gặp trong thực tế.

#### Học hỏi từ các diễn giả có chuyên môn cao
- Các diễn giả đến từ AWS Community, Cloud Security và doanh nghiệp thực tế đã chia sẻ góc nhìn security-first trong thiết kế và vận hành hệ thống cloud.
- Thông qua những ví dụ sát với môi trường production, tôi hiểu rõ hơn cách các security best practices được áp dụng trong doanh nghiệp, thay vì chỉ dừng ở mức lý thuyết hay checklist.

#### Trải nghiệm kỹ thuật thực tế
- Nội dung workshop được trình bày theo 5 Security Pillars của AWS Well-Architected Framework, giúp tôi có cái nhìn có hệ thống về kiến trúc bảo mật end-to-end.
- Các phần về IAM, logging, detection và incident response giúp tôi hình dung rõ hơn cách các thành phần bảo mật liên kết với nhau trong một kiến trúc thực tế.
- Đặc biệt, các ví dụ về incident scenarios (lộ IAM key, S3 public, EC2 nhiễm malware) giúp tôi hiểu rõ quy trình phản ứng sự cố thay vì chỉ “biết dịch vụ”.
- 
#### Ứng dụng công cụ hiện đại
- Trực tiếp tìm hiểu cách kết hợp các dịch vụ như CloudTrail, GuardDuty, Security Hub và EventBridge để xây dựng hệ thống detection và alerting liên tục.
- Hiểu rõ vai trò của AWS KMS, Secrets Manager trong việc bảo vệ dữ liệu và quản lý secrets một cách an toàn.
- Nhận thấy tiềm năng của automation trong security, từ auto-remediation bằng Lambda đến orchestrate workflows với Step Functions.


#### Kết nối và trao đổi
- Workshop tạo môi trường mở để trao đổi với các anh/chị có kinh nghiệm làm security và cloud engineering, giúp tôi học được nhiều bài học ngoài slide.
- Qua các chia sẻ thực tế, tôi nhận ra security không chỉ là trách nhiệm của một team, mà cần phối hợp chặt chẽ giữa engineering, operations và business.

#### Bài học rút ra
- Security phải bắt đầu từ kiến trúc, không thể vá lỗi ở giai đoạn cuối
- IAM và logging là hai nền móng quan trọng nhất, nếu làm sai sẽ kéo theo nhiều rủi ro phía sau.
- Incident Response cần được chuẩn bị trước bằng playbooks và automation, không thể xử lý hiệu quả nếu chỉ phản ứng khi sự cố xảy ra.
- Việc sử dụng các dịch vụ managed của AWS giúp giảm gánh nặng vận hành và hạn chế lỗi con người trong bảo mật.
  