---
title: "Event 2"
date: "2025-01-01"
weight: 1
chapter: false
pre: " <b> 4.2. </b> "
---


# Takeaways from "DevOps on AWS Workshop"

### Event Purpose

- Equip end-to-end DevOps knowledge on AWS
- Build complete CI/CD pipeline
- Get familiar with and practice Infrastructure as Code (IaC)
- Deploy and operate container-based applications
- Properly understand monitoring & observability
- Share best practices, case studies and DevOps career path
### Speaker List

- **Bao Huynh** - AWS Community Builder
- **Thinh Nguyen** - AWS Community Builder
- **Vi Tran** - AWS Community Builder
### Highlighted Content

#### CI/CD Pipeline with AWS DevOps Services

Workshop went from foundation to practical deployment:

* Source Control
  * AWS CodeCommit
  * Compare GitFlow and Trunk-based development
→ Clearly understand when to use each strategy for different team sizes and release models.

* Build & Test
  * Configure AWS CodeBuild
  * Integrate unit test, integration test into pipeline

* Deployment
  * Blue/Green deployment – zero downtime
  * Canary deployment – controlled rollout
  * Rolling updates – suitable for stateless applications

 **Demo** Complete CI/CD pipeline was the biggest highlight, showing the entire flow from code commit to production deployment, along with rollback and error handling strategies.

#### Infrastructure as Code (IaC)
The IaC section helps change mindset from "click ops" to "infrastructure as software".

* AWS CloudFormation
  * Templates, stack management
  * Drift detection – extremely important in production
  * Nested stacks for complex architecture
  
* AWS CDK
  *  Write infrastructure in TypeScript/Python instead of YAML/JSON
  *  Type safety, IDE support
  *  Reusable constructs and higher-level abstractions
  *  Testing infrastructure code
  
#### Container Services on AWS

Container content was presented very comprehensively:
* Docker Fundamentals
  * Containerization concepts
  * Microservices architecture
  * Best practices for Dockerfile
  * Multi-stage builds
* Amazon ECR
  * Private container registry
  * Image security scanning
  * Lifecycle policies for image management
* Amazon ECS vs EKS
  * ECS: AWS-native, simple setup, deep AWS integration
  * EKS: Kubernetes standard, portable, higher learning curve
  * Auto-scaling and deployment strategies for each service

#### Monitoring & Observability

* One of the most practical and immediately applicable sections:
  * Amazon CloudWatch
  * Standard metrics & custom metrics
  * Log aggregation
  * Alarm with SNS notifications
  * Dashboard design for each stakeholder
* AWS X-Ray
  * Distributed tracing for microservices
  * Service map visualization
  * Performance bottleneck identification
* Observability Best Practices
  * Three pillars: Metrics – Logs – Traces
  * Alerting strategies to avoid alert fatigue
  * On-call rotation & incident response
  
#### DevOps Best Practices & Case Studies

* Progressive Delivery
  * Feature flags
  * A/B testing
  * Canary analysis
* Automated Testing
  * Test pyramid: Unit → Integration → E2E
  * CI/CD integration
  * Quality gates
* Incident Management
  * Incident response procedures
  * Blameless postmortems
  * Learning from failures

### What I Learned

* After the workshop, I better understand:
  * How to design production-ready CI/CD pipeline on AWS
  * When to use Blue/Green, Canary or Rolling deployments
  * IaC mindset and choosing the right tool for each stage
  * Container deployment strategy suitable for workload
  * Importance of observability in distributed systems
  * DevOps is not just tooling but mindset: automation, collaboration, continuous improvement

#### Personal Experience
**Overall Impression**

The workshop was intensive and very practical. Compared to the previous AI/ML workshop, the DevOps workshop focused more on practical operations and immediately applicable skills.

**Highlights**
  * CI/CD pipeline demo with Blue/Green deployment
  * AWS CDK – truly a "game changer"
  * Monitoring & observability section extremely practical
  * Very relatable case studies

**Networking & Q&A**
  * Discussion about career paths in DevOps
  * AWS certification roadmap (DevOps Engineer – Professional)
  * Experience sharing from practitioners doing real DevOps
