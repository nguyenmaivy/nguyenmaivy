---
title: "Worklog Week 11"

weight: 2
chapter: false
pre: " <b> 1.11. </b> "
---

### Week 11 Objectives:

* Improve system quality through Backend Unit Tests and Frontend E2E Tests.
* Build a basic Data Pipeline & Business Intelligence (BI) system using AWS services.
* Finalize and optimize the CI/CD pipeline for both Frontend and Backend.
* Ensure system security by reviewing IAM permissions with the Least Privilege principle.

### Tasks for this Week:
| Day | Tasks                                                                                                                                                                                       | Start Date  | Completion Date | References                                         |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------- | ---------------- | -------------------------------------------------- |
| 2   | - Backend Unit Tests using **Jest**: write unit tests for core Lambdas (`roomCrud.js`, `searchRooms.js`)                                                                                   | 08/12/2025  | 08/12/2025       | https://jestjs.io/                                |
| 3   | - Frontend E2E Tests with **Cypress**: create test scenarios for 3 main flows: <br> &emsp; + Login <br> &emsp; + Search/Filter <br> &emsp; + Post Room                                     | 09/12/2025  | 09/12/2025       | https://www.cypress.io/                           |
| 4   | - Set up **AWS Athena** on exported data from S3 <br> - Write **athena-queries.sql**: <br> &emsp; + Average rental price by District <br> &emsp; + Top 5 most viewed rooms                | 10/12/2025  | 10/12/2025       | https://docs.aws.amazon.com/athena/               |
| 5   | - Configure **Amazon QuickSight** <br> - Build a basic dashboard to visualize Athena query results                                                                                        | 11/12/2025  | 11/12/2025       | https://docs.aws.amazon.com/quicksight/           |
| 6   | - Optimize **deploy-frontend.yml**: Build Next.js → Upload to S3 → Invalidate CloudFront <br> - Review **IAM Permissions** for all Lambdas (Least Privilege principle)                     | 12/12/2025  | 12/12/2025       | https://docs.github.com/en/actions                |

### Week 11 Achievements:

* Backend Unit Tests successfully implemented for:
  * `roomCrud.js`
  * `searchRooms.js`
* Frontend E2E Tests completed using Cypress for:
  * Login
  * Search & Filter
  * Post Room
* System stability improved with **minimum Test Coverage ~70%**.
* Data Pipeline & BI successfully established:
  * AWS Athena configured on S3-exported data.
  * SQL queries created:
    * Average rental price by district
    * Top 5 most viewed rooms
  * Amazon QuickSight dashboard created for data visualization.
* CI/CD pipeline fully completed:
  * Automatic deployment for Frontend (Build → S3 → CloudFront)
  * Backend pipeline reviewed and optimized.
* IAM permissions reviewed and restricted based on the **Least Privilege** principle.
* **Final Result:** The system now supports automated testing, business data analytics (BI), and fully automated CI/CD for both Frontend and Backend.
