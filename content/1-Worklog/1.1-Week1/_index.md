---
title: "Week 1 Worklog"

weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

### Week 1 Objectives:

* Connect and get acquainted with members in the First Cloud Journey.
* Understand basic AWS services, how to use the console & CLI.

### Tasks to be deployed this week:
| Day | Task                                                                                                                                                                                                 | Start Date | Completion Date | Resource                              |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| Mon | - Get acquainted with FCJ members <br> - Read and note the rules and regulations at the internship unit                                                                                                   | 29/09/2025   | 29/09/2025      | |
| Tue | - Research AWS and its service categories <br>&emsp; + Compute <br>&emsp; + Storage <br>&emsp; + Networking <br>&emsp; + Database <br>&emsp; + ... <br>                                            | 30/09/2025   | 30/09/2025      | <https://cloudjourney.awsstudygroup.com/> |
| Wed | - Create an AWS Free Tier account <br> - Explore AWS Console & AWS CLI <br> - **Practice:** <br>&emsp; + Create an AWS account <br>&emsp; + Install & configure AWS CLI <br> &emsp; + How to use AWS CLI | 01/10/2025   | 01/10/2025      | <https://cloudjourney.awsstudygroup.com/> |
| Thu | - Research basic EC2: <br>&emsp; + Instance types <br>&emsp; + AMI <br>&emsp; + EBS <br>&emsp; + Security Group <br>&emsp; + ... <br> - Methods to remotely SSH into EC2 <br> - Research Elastic IP   <br>                   | 02/10/2025   | 02/10/2025      | <https://cloudjourney.awsstudygroup.com/> |
| Fri | - **Practice:** <br>&emsp; + Create an EC2 instance (T2.micro, Amazon Linux 2, new Key Pair) <br>&emsp; + Configure **Security Group** to open SSH port <br>&emsp; + SSH connection from personal machine <br>&emsp; + Attach **EBS volume** & mount to EC2                                                                                                         | 03/10/2025   | 03/10/2025      | <https://cloudjourney.awsstudygroup.com/> |

<br>

---

### Week 1 Outcomes:

* **Relationships:** Got acquainted and connected with the members of the **First Cloud Journey (FCJ)** group.
* **Basic AWS Knowledge:**
  * Understood **what AWS is** and grasped the basic service categories:
    * Compute (**EC2**), Storage (**S3, EBS**), Networking (**VPC, Security Group**), Database (**RDS**), etc.
* **AWS Account & Tools Practice:**
  * Successfully created and configured an **AWS Free Tier account**.
  * Became familiar with the **AWS Management Console** and learned how to find, access, and use services from the web interface.
  * Installed and configured **AWS CLI** on the computer including:
    * Default **Access Key, Secret Key, Region**.
* **EC2 Usage:**
  * Grasped the basic concepts of **EC2 (Instance types, AMI, EBS, Security Group)**.
  * Successfully created an **EC2 instance** (t2.micro) and established an **SSH** connection from a personal machine.
  * Learned how to manage and attach an **EBS volume** to the EC2 instance.
* **AWS CLI Usage:**
  * Used AWS CLI to perform basic operations such as:
    * Checking account and configuration information (e.g., `aws configure list`).
    * Getting a list of regions (e.g., `aws ec2 describe-regions`).
    * Viewing EC2 services (e.g., `aws ec2 describe-instances`).
    * Creating and managing key pairs.
* **Summary:** Capable of connecting between the web interface and CLI to manage AWS resources in parallel.