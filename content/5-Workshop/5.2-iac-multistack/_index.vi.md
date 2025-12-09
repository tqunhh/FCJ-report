---
title: "Infrastructure as Code - Multi-Stack Pattern"
date: 2025-12-08
weight: 2
chapter: true
pre: "<b> 5.2. </b>"
alwaysopen: false
---

## Infrastructure as Code với Multi-Stack Deployment

Phần này giải thích **chiến lược Infrastructure as Code (IaC)** được sử dụng để xây dựng hạ tầng Travel Guide Application từ đầu bằng **CloudFormation/SAM** với **multi-stack deployment pattern**.

### Tổng quan

Travel Guide Application được xây dựng theo kiến trúc microservices triển khai trên nhiều AWS CloudFormation stacks. Cách tiếp cận này mang lại sự cô lập tốt hơn, triển khai nhanh hơn và giảm thiểu rủi ro khi thay đổi.

![Kiến trúc Multi-Stack](/images/5-Workshop/5.2-Prerequisite/CloudFormation_Mutilstack.png)

### Các khái niệm chính

- **Infrastructure as Code**: Toàn bộ hạ tầng được định nghĩa trong CloudFormation templates
- **Multi-Stack Pattern**: Tách biệt core resources và service-specific resources
- **Cross-Stack References**: Chia sẻ resources giữa các stacks bằng CloudFormation Exports/Imports
- **Deployment Orchestration**: Triển khai tự động bằng Bash scripts
- **Environment Management**: Cấu hình dựa trên parameters cho nhiều môi trường

### Nội dung

- [Chiến lược IaC & Lựa chọn công cụ](5.2.1-iac-strategy/)
- [Kiến trúc Multi-Stack](5.2.2-multistack-architecture/)
- [Cross-Stack References](5.2.3-cross-stack-references/)
- [Deployment Orchestration](5.2.4-deployment-orchestration/)
- [Quản lý Parameters](5.2.5-parameter-management/)
- [Bài học kinh nghiệm & Best Practices](5.2.6-lessons-learned/)
