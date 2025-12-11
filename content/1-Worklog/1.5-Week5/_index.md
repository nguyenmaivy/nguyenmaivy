---
title: "Week 5 Worklog"

weight: 1
chapter: false
pre: " <b> 1.5. </b> "
---


### Week 5 Objectives:

* Understand core AWS security services (IAM, SSO, Cognito, KMS, Organizations).
* Practice configuring Security Hub, optimizing EC2, and managing resources using Tags/IAM.
* Set up IAM Users/Groups/Roles, Permission Boundaries, and define project objectives and system architecture.
* Select the project topic and identify the project requirements.

### Tasks to be carried out this week:
| Day | Task                                                                                                                                                                                                   | Start Date | Completion Date | Reference Material                        |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------- | --------------- | ----------------------------------------- |
| 2   | - Familiarize with AWS security services: <br>&emsp; + Shared Responsibility Model <br>&emsp; + Amazon Cognito <br>&emsp; + AWS Organization <br>&emsp; + AWS Identity Center (SSO) <br>&emsp; + AWS KMS <br>                                                                                             | 03/11/2025   | 03/11/2025      |
| 3   | - **Practice:** <br>&emsp; + Enable Security Hub and perform assessments based on each security standard  <br>&emsp; + Optimizing EC2 Costs with AWS Lambda                                          | 04/11/2025   | 04/11/2025      | <https://000018.awsstudygroup.com/> <br>&emsp; <https://000022.awsstudygroup.com/> |
| 4   | - **Practice:** <br>&emsp; + Manage Resources using Tags and Resource Groups <br>&emsp; + Manage access to EC2 services using resource tags with AWS IAM | 05/11/2025   | 05/11/2025     | <https://000027.awsstudygroup.com/> <br>&emsp; <https://000028.awsstudygroup.com/> |
| 5   | - - **Practice:** <br>&emsp; + Limitation of user rights with IAM permission boundary <br>&emsp; + Encrypt at rest with AWS KMS                 | 06/11/2025  | 06/11/2025      | <https://000030.awsstudygroup.com/> <br>&emsp; <https://000033.awsstudygroup.com/> |
| 6   | - Select the project topic and define project objectives <br> - Design the system architecture <br> - **Practice:** <br>&emsp; + Create IAM Groups and IAM Users, and configure Role Conditions <br>&emsp; + Granting authorization for an application to access AWS services with an IAM role                                                                                         | 07/11/2025  | 09/11/2025      | <https://000044.awsstudygroup.com/> <br>&emsp; <https://000048.awsstudygroup.com/> |


### Week 5 Achievements:

* Understand and grasp core AWS security service groups:
  * Shared Responsibility Model
  * Amazon Identity and Access Management
  * Amazon Cognito
  * AWS Organizations
  * AWS Identity Center (SSO)
  * AWS Key Management Service (KMS)
* Practice and operate security services
  * Enable and become familiar with AWS Security Hub.
  * Perform security checks and assessments according to various standards and AWS Best Practices.
  * Identify security findings and determine their handling priority.
* Optimize EC2 costs using AWS Lambda
  * Use Lambda to automatically start/stop EC2 instances on a schedule.
  * Identify unnecessary resources that can be optimized.
* Manage AWS resources using Tags and Resource Groups
  * Establish a standardized tagging system (Environment, Owner, Project, CostCenter).
  * Apply tags to EC2, S3, Lambda, and other resources.
  * Create Resource Groups to monitor and manage resources by group or project.
* Manage access using IAM and Tag Conditions
  * Configure IAM policies with conditions based on resource tags.
  * Limit EC2 access through Resource Tags to separate environments and reduce operational risks.
* Restrict user permissions with Permission Boundaries
  * Configure Permission Boundaries for IAM Users/Roles.
  * Ensure users cannot assign themselves permissions beyond the allowed limit.
* Encrypt data using AWS KMS
  * Configure at-rest encryption for S3, EBS, and related services.
  * Create and manage keys, and control key usage through KMS policies.
* Project topic selection and scope definition
  * Select the project topic and define project objectives, scope, and expected deliverables.
* System architecture design
  * Draft the overall architecture of the application/project.
  * Identify AWS services used, and define authorization, encryption, and logging workflows.
* Advanced IAM practice
  * Create IAM Groups and IAM Users for each functional team.
  * Configure IAM Roles and define access conditions (IP/Tag/Time).
  * Grant application access to AWS services using IAM Roles.
