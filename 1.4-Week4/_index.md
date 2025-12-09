---
title: "Week 4 Worklog"

weight: 1
chapter: false
pre: " <b> 1.4. </b> "
---


### Week 4 Objectives:

* Understand AWS storage services.
* Know how to deploy a Backup system, perform virtual machine Import/Export, and implement Storage Gateway.

### Tasks to be carried out this week:
| Day | Task                                                                                                                                                                                                   | Start Date | Completion Date | Reference Material                        |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------- | --------------- | ----------------------------------------- |
| 2   | - Learn about Amazon Simple Storage Service (S3) với tính năng Access Point và Storage Class của S3 <br>&emsp; - Learn about S3 Static Website & CORS, Control Access, Object Key & Performance, Glacier                                           | 20/10/2025   | 20/10/2025 |<https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html> | 
| 3   | - Learn about Snow Family, Storage Gateway, Backup <br>                | 21/10/2025   | 21/10/2025      | <https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html> |
| 4   | - **Practice:** Deploy AWS Backup to the System <br>&emsp; + Create S3 Bucket và Deploy Infrastructure <br>&emsp; + Create Backup Plan, set up notifications and test restore                                                                   | 22/10/2025   | 22/10/2025      | <https://000013.awsstudygroup.com/> |
| 5   | - **Practice:** VM Import/Export<br>&emsp; + VMWare WorkStation <br>&emsp; + Import virtual machine to AWS   <br>&emsp; +    Export EC2 Instance from AWS                                                             | 23/10/2025   | 23/10/2025      | <https://000014.awsstudygroup.com/> |
| 6   | - **Practice:** <br>&emsp; + Using File Storage Gateway: Create Storage Gateway, File Shares and Mount File shares on On-premises machine <br>&emsp; + Amazon FSx for Windows File Server                      | 24/10/2025   | 24/10/2025      | <https://000024.awsstudygroup.com/> |


### Week 4 Achievements:

* Understand what Amazon Simple Storage Service (S3) is and grasp its core feature groups:
  * Amazon S3 Access Point
  * S3 Static Website & CORS
  * Access Control, Object Key & Performance, Glacier
* Understand the Snow Family, Storage Gateway, and Backup services.
* Created and configured an S3 Bucket, deployed the related infrastructure, set up notifications, and verified successful operation.
* Successfully created a Storage Gateway and File Shares, with the ability to connect file shares from On-premise machines.
* Able to import virtual machines into AWS and export EC2 Instances from AWS:
  * Export virtual machines from On-premise
  * Upload virtual machines to AWS
  * Launch EC2 Instances from AMI
* Able to export EC2 Instances from AWS:
  * Configure ACL for the S3 Bucket
  * Export virtual machines from EC2 Instances
* Set up a shared data storage system for the Windows infrastructure:
  * Create a practice environment to set up a new file share
  * Monitor and evaluate performance
  * Enable components required for deploying FSx on Windows such as user storage quotas, continuous access sharing, etc.
