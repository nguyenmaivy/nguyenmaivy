---
title: "Week 6 Worklog"

weight: 1
chapter: false
pre: " <b> 1.6. </b> "
---


### Week 6 Objectives:

* Set up the project architecture using a Monorepo structure.
* Build the initial infrastructure using IaC (Infrastructure as Code) with AWS SAM.
* Complete the Backend MVP for the CREATE room function.
* Design UI wireframes/mockups for the Frontend direction.
* Set up basic CI/CD for upcoming deployments.

### Tasks for this Week:
| Day | Tasks                                                                                                                                                                    | Start Date  | Completion Date | References                                                        |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------- | ---------------- | ------------------------------------------------------------------ |
| 2   | - Create **Monorepo structure**: backend/, frontend/, infrastructure/ <br> - Complete **template.yaml** (IaC): DynamoDB + 2 Lambdas (CRUD & Search) <br> - Run `sam build` | 03/11/2025  | 03/11/2025       | https://docs.aws.amazon.com/serverless-application-model/         |
| 3   | - Backend MVP: Implement **CREATE** function in `roomCrud.js` <br> - Configure `package.json` (aws-sdk, node-fetch)                                                      | 04/11/2025  | 04/11/2025       | https://docs.aws.amazon.com/lambda/                               |
| 4   | - Design UI **wireframe/mockups** for Home, Search, and Post Room pages (no coding UI)                                                                                   | 05/11/2025  | 05/11/2025       | https://www.figma.com                                             |
| 5   | - Set up CI/CD: <br> &emsp; + Configure **AWS Secrets** on GitHub <br> &emsp; + Write initial **deploy-backend.yml**                                                     | 06/11/2025  | 06/11/2025       | https://docs.github.com/en/actions                                |

### Week 6 Achievements:

* Successfully created a well-structured Monorepo with backend/, frontend/, and infrastructure/.
* Completed the IaC file **template.yaml**:
  * 1 DynamoDB Table: **Rooms**
  * 2 Lambda Functions: **roomCrud.js** and **searchRooms.js**
* Successfully ran **sam build**, confirming the infrastructure is ready for deployment.
* Completed Backend MVP for **CREATE Room** function.
* Designed wireframes/mockups for Home, Search, and Post Room pages.
* Completed initial CI/CD setup:
  * Added AWS Secrets to GitHub
  * Created draft deploy-backend.yml
* **Overall Result:** The IaC foundation and CREATE API were successfully defined and prepared for deployment.