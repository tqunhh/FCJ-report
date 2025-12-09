---
title: "Event 3"
date: "2025-01-01"
weight: 1
chapter: false
pre: " <b> 4.3. </b> "
---

{{% notice warning %}}
⚠️ **Note:** The information below is for reference purposes only. Please **do not copy verbatim** for your report, including this warning.
{{% /notice %}}

# Takeaways from "AWS Well-Architected Security Pillar Workshop"

### Event Purpose

* Provide comprehensive and systematic knowledge about security on AWS based on the 5 Security Pillars of AWS Well-Architected Framework
* Build security-first mindset, considering security as the foundation from the architecture design phase, not an afterthought.
* Guide practical best practices on:
* Introduce AI tools supporting development lifecycle
  * Identity & Access Management (IAM)
  * Logging, detection and continuous monitoring
  * Infrastructure and network protection
  * Data encryption, key and secrets management
  * Incident response and forensics

### Speaker List

- **Tran Duc Anh** - AWS Cloud Club Captain (SGU)
- **Tran Doan Cong Ly** - AWS Cloud Club Captain (PTIT)
- **Danh Hoang Hieu Nghi** - AWS Cloud Club Captain (HUFLIT)
- **Nguyen Tuan Thinh** - Cloud Engineer Trainee
- **Nguyen Do Thanh** - Cloud Engineer
- **Thinh Lam** - FCJer
- **Viet Nguyen** - FCJer
- **NMendel Grabski (Long)** - ex Head of Security & DevOps - Cloud Security Solution Architect
- **Quang Tinh Truongh** - AWS Community Builder - Platform Engineer at TymeX
### Highlighted Content

#### Identity & Access Management (IAM) – foundation of security
- Design modern IAM with roles instead of users
- Apply least privilege principle
- Use IAM Identity Center (SSO) for multi-account model
- Manage and control permissions with SCP, permission boundaries and Access Analyzer

#### Detection & Continuous Monitoring

- Deploy centralized logging with CloudTrail, VPC Flow Logs, CloudWatch Logs
- Detect threats with GuardDuty and aggregate alerts via Security Hub
- Automate alerting and incident response with EventBridge

#### Infrastructure Protection
- Design secure network with VPC segmentation, private subnet and VPC endpoints
- Apply defense in depth model using Security Groups, NACLs
- Advanced protection with AWS WAF, Shield and Network Firewall
- Best practices for securing EC2, ECS/EKS and container workloads
  
#### Incident Response & Automation

- Build Incident Response process according to AWS standards
- Practice real playbooks: IAM key exposure, public S3, EC2 with malware
- Collect forensics and maintain chain of custody
- Automate incident handling with Lambda, Step Functions

### What I Learned

#### Correct security mindset from design
- Security is not an afterthought, but a foundational element in cloud architecture.
- Apply security-first mindset when designing, deploying and operating systems on AWS.
- Clearly understand the relationship between Security – DevOps – AI/ML, where security is the condition for sustainable system operation.
  
#### Technical Architecture

- All components are tightly connected and continuously monitored, not dependent on a single control point.
- IAM plays the role of the first gateway of the entire architecture.
- Apply layered network security architecture.


### Event Experience

Participating in the **"AWS Well-Architected Security Pillar"** workshop gave me a very practical and comprehensive perspective on how to build security systems on AWS according to proper architectural standards. Although it only took place in the morning, the workshop still had high knowledge density and focused precisely on core issues that technical teams often encounter in practice.

#### Learning from highly specialized speakers
- Speakers from AWS Community, Cloud Security and real enterprises shared security-first perspectives in cloud system design and operation.
- Through examples close to production environments, I better understood how security best practices are applied in enterprises, rather than just stopping at theory or checklists.

#### Practical technical experience
- Workshop content was presented according to the 5 Security Pillars of AWS Well-Architected Framework, helping me have a systematic view of end-to-end security architecture.
- Sections on IAM, logging, detection and incident response helped me better visualize how security components connect with each other in real architecture.
- Especially, examples of incident scenarios (IAM key exposure, public S3, EC2 infected with malware) helped me understand incident response processes instead of just "knowing services".

#### Modern tool application
- Directly learn how to combine services like CloudTrail, GuardDuty, Security Hub and EventBridge to build continuous detection and alerting systems.
- Clearly understand the role of AWS KMS, Secrets Manager in protecting data and managing secrets securely.
- Recognize the potential of automation in security, from auto-remediation with Lambda to orchestrating workflows with Step Functions.


#### Connection and exchange
- The workshop created an open environment to exchange with experienced security and cloud engineering professionals, helping me learn many lessons beyond slides.
- Through real sharing, I realized security is not just the responsibility of one team, but requires close coordination between engineering, operations and business.

#### Lessons learned
- Security must start from architecture, cannot patch holes at the final stage
- IAM and logging are the two most important foundations, if done wrong will lead to many risks later.
- Incident Response needs to be prepared in advance with playbooks and automation, cannot be handled effectively if only reacting when incidents occur.
- Using AWS managed services helps reduce operational burden and limit human errors in security.
  
