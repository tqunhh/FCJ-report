---
title: "Worklog Week 2"
date: "2025-01-01"
weight: 1
chapter: false
pre: " <b> 1.2. </b> "
---

### Week 2 Objectives:

* Understand the role of Route 53.
* Deploy AWS Backup.

### Tasks to be implemented this week:
| Day | Tasks                                                                                                                                                                                   | Start Date | Completion Date | Resources                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | - Learn Hybrid DNS concept <br> - Set up DNS infrastructure <br>&emsp; + Create Route 53 Outbound Endpoint <br>&emsp; + Create Route 53 Inbound Endpoint <br>&emsp; + Configure Resolver Rules                                                                                | 15/09/2025   | 15/09/2025      | https://000010.awsstudygroup.com/vi/
| 3   | - Learn and set up VPC Peering <br> -  Understand why to use peering instead of NAT/Internet gateway <br> - Create CloudFormation template, Security Group, EC2 <br> - Update Network ACL <br> - Create peering connection and update route tables                                     | 16/09/2025   | 16/09/2025      | https://000019.awsstudygroup.com/vi/ |
| 4   | - Learn about Transit Gateway concept and advantages <br> - **Hands-on:** <br>&emsp; + Create Transit Gateway <br>&emsp; + Transit Gateway Attachments <br>&emsp; + Transit Gateway Route Tables <br>&emsp; + Add Transit Gateway Routes to VPC Route Tables  | 17/09/2025   | 17/09/2025      | <https://000020.awsstudygroup.com/vi/> |
| 5   | - Learn how to create, manage, store, automate and scale EC2 Instance on AWS <br> - Distinguish between EBS volume and Instance Store <br> - How to automatically configure EC2 at initialization using User Data Script <br> - Learn how to deploy auto scaling system <br>                  | 18/09/2025   | 18/09/2025      |  |
| 6   | - **Hands-on:** <br>&emsp; + Create S3 bucket to store backup data <br>&emsp; + Deploy infrastructure <br>&emsp; + Create Backup Plan                                                                        | 19/09/2025   | 19/09/2025      | <https://000013.awsstudygroup.com/vi/> |


### Week 2 Achievements:

* Understand Hybrid DNS concept:  
  * Role of Route 53 Resolver when handling internal & external domain names
  * Components in hybrid DNS: Outbound Endpoint, Inbound Endpoint, Resolver Rules

* Distinguish two types of storage:
  * EBS (Elastic Block Store) – virtual disk attached to instance.
  * Instance Store – temporary disk, lost when machine is turned off.

* Transit Gateway operates as a "central router" between multiple VPCs (and possibly on-prem)

* How to create Amazon S3 Bucket to store data on cloud.
* Set up Bucket Policy, Permissions, and Public Access Block.

* Deploy AWS Backup:
  * Create Backup Vault to store backups
  * Create Backup Plan and schedule automatic backups.
  * Assign resources (EC2, EBS, RDS, EFS, …) to backup plan
  * Test and try restoring backed up data.

