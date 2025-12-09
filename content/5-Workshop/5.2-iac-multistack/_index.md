---
title: "Infrastructure as Code - Multi-Stack Pattern"
date: 2025-12-08
weight: 2
chapter: true
pre: "<b> 5.2. </b>"
alwaysopen: false
---

## Infrastructure as Code with Multi-Stack Deployment

This section explains the **Infrastructure as Code (IaC) strategy** used to build the Travel Guide Application infrastructure from scratch using **CloudFormation/SAM** with a **multi-stack deployment pattern**.

### Overview

The Travel Guide Application is built using a microservices architecture deployed across multiple AWS CloudFormation stacks. This approach provides better isolation, faster deployments, and reduced blast radius when making changes.

![Multi-Stack Architecture](/images/5-Workshop/5.2-Prerequisite/CloudFormation_Mutilstack.png)

### Key Concepts

- **Infrastructure as Code**: All infrastructure defined in CloudFormation templates
- **Multi-Stack Pattern**: Separation of core resources and service-specific resources
- **Cross-Stack References**: Sharing resources between stacks using CloudFormation Exports/Imports
- **Deployment Orchestration**: Automated deployment using Bash scripts
- **Environment Management**: Parameter-based configuration for multiple environments

### Content

- [IaC Strategy & Tool Selection](5.2.1-iac-strategy/)
- [Multi-Stack Architecture](5.2.2-multistack-architecture/)
- [Cross-Stack References](5.2.3-cross-stack-references/)
- [Deployment Orchestration](5.2.4-deployment-orchestration/)
- [Parameter Management](5.2.5-parameter-management/)
- [Lessons Learned & Best Practices](5.2.6-lessons-learned/)
