---
title: "Proposal"

weight: 2
chapter: false
pre: "<b>2.</b>"
---
# Roommate
## Unified AWS Serverless Solution for Real-Time Room Management and Search

### 1. Operational Summary
**RoomMate** is a room search and management platform designed for students and landlords, leveraging **Serverless** and **AWS Free Tier** architecture. The platform allows landlords to easily post listings, manage room data (add, edit, delete), and receive recurring reports. Students can search, filter by multiple criteria (location, price, amenities), save favorite rooms, and chat directly with landlords via the integrated mini-chat feature. The architecture uses Next.js for the user interface, API Gateway + Lambda for the backend, DynamoDB for main data storage, etc., for secure authentication. The key difference lies in the integration of OpenAI to automate tasks such as sending email/Telegram notifications when new, suitable rooms become available or generating statistical reports, increasing the system's interactivity and reliability.

### 2. Problem Statement
*Current Problem*
- Traditional methods of finding accommodation (bulletin boards, social media groups) often **lack detailed filtering systems**, information is unfocused, and is easily disrupted by spam.

- Communication between students and landlords is usually via phone/Zalo, **creating a "friction" (communication barrier)** and making it difficult to track history.

- **Lack of an automatic notification mechanism** when new rooms become available that match students' specific needs.

- Landlords **lack analytical tools** regarding market demand and the number of views on listings to optimize rentals.

*Solution*

The **RoomMate** platform provides a centralized rental search and management solution, integrating an internal **mini-chat** and **automatic notifications** (via n8n) based on individual criteria.

- **AWS Serverless Architecture**: Utilizes **API Gateway + Lambda, DynamoDB** (for room data, users, and chat), and S3 (for images) to ensure scalability at optimal cost (leveraging Free Tier).

- **Key Features**: Unlike generic chat/classifieds applications, **RoomMate** focuses on a positive tenant experience by using AI chatbots to suggest rental rooms based on individual requirements.

*Benefits and Return on Investment (ROI)*
- **Benefits**: Provides a very practical solution for students. Reduces communication friction between parties through internal chat. Create a highly scalable platform (easy to add maps, review). Ensure all 5 core objectives of a technology project are met (serverless, CI/CD, monitoring, security, data pipeline).

- **Return on Investment (ROI)**: Extremely low development and operating costs due to maximizing the use of AWS Free Tier (Lambda, DynamoDB, S3), minimizing server management costs. The value delivered is a complete, practical product with outstanding features thanks to automation and internal chat capabilities.

### 3. Solution Architecture
The platform utilizes the AWS Serverless architecture to build a data management and analysis system. The system uses Amazon API Gateway and AWS Lambda to handle business logic. The web interface is distributed globally via CloudFront, acting as a single access point for both static content and API requests. Data is securely stored in Amazon S3 (for files/images) with protected buckets, while Amazon DynamoDB handles the storage of structured data. The deployment process is fully automated through AWS CodePipeline, sourced from GitHub. The entire operation is monitored by Amazon CloudWatch.

![Roommate Platform Architecture](/images/2-Proposal/platfrom_architecture.png)

*AWS Services Used*

- *AWS Lambda*: Executes business logic functions (Backend Business Logic).

- *Amazon API Gateway*: Receives, authenticates, and routes API requests from the web application to AWS Lambda.

- *Amazon S3*: S3 - Frontend stores static content of the web application (the destination of CodePipeline). S3 - Image Storage stores file and image data with protection mechanisms (S3 - Protected).

- Amazon DynamoDB: Stores structured data (NoSQL), including user data and device information.

- AWS CodePipeline: Automates CI/CD processes for the entire system, deploying the web interface to the S3 Frontend.

- Amazon Cognito: Manages user identities and access rights (authentication), protecting the API Gateway.

- Amazon CloudFront: Distributes web application content globally (from the S3 Frontend) and routes API requests to the API Gateway.

- Amazon CloudWatch: Monitors, logs, and provides alerts for the entire system.

*Component Design*
- *User Interface*: The web application is hosted on the S3 Frontend and distributed globally via CloudFront.

- *API Layer*: The Lambda API Gateway handles business requests after users are authenticated by Cognito.

- *Data Storage*: File and image data are stored in S3 Image Storage (B3 Image Storage). Structured data, users, and devices are stored in DynamoDB.

- *Deployment Pipeline*: AWS CodePipeline automates the build, test, and deployment of changes from GitHub to the S3 Frontend and resources.