---
title: "Week 7 Worklog"

weight: 1
chapter: false
pre: " <b> 1.7. </b> "
---


### Week 7 Objectives:

* Deploy AWS infrastructure (Lambda, API Gateway, DynamoDB).
* Build the Frontend MVP (Base UI) and the Post Room form.
* Integrate APIs across Frontend → API Gateway → Lambda → DynamoDB.
* Implement basic backend functions for room search.

### Tasks for this Week:
| Day | Tasks                                                                                                                                                        | Start Date  | Completion Date | References                                 |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------- | ---------------- | ------------------------------------------- |
| 2   | - Deploy AWS infrastructure using **sam deploy** <br> - Create Lambda, API Gateway, DynamoDB <br> - Add API Gateway Endpoint to `frontend/.env.local`         | 10/11/2025  | 10/11/2025       | https://cloudjourney.awsstudygroup.com/     |
| 3   | - Build UI for **Layout (`layout.js`)** <br> - Build UI for **Post Room Page (`post-room/page.js`)** <br> - Install & configure **Tailwind CSS**             | 11/11/2025  | 11/11/2025       |                                             |
| 4   | - Implement **API Proxy** in Next.js at `frontend/api/proxy/`                                                                                                | 12/11/2025  | 12/11/2025       | https://nextjs.org/docs                     |
| 5   | - Integrate Post Room form with **Lambda roomCrud.js** via API Proxy                                                                                         | 13/11/2025  | 13/11/2025       | https://docs.aws.amazon.com/lambda/         |
| 6   | - Implement backend search function: create **searchRooms.js** (READ – get all rooms)                                                                        | 14/11/2025  | 14/11/2025       | https://docs.aws.amazon.com/amazondynamodb/ |
| 7   | - Test entire flow: Frontend → API Proxy → API Gateway → Lambda → DynamoDB <br> - Fix bugs & finalize MVP                                                    | 15/11/2025  | 15/11/2025       |                                             |

### Week 7 Achievements:

* Successfully deployed AWS infrastructure via **sam deploy**.
* Configured Frontend environment (`.env.local`) with API Gateway endpoint.
* Completed base UI:
  * Layout
  * Post Room page
  * Tailwind CSS working smoothly
* Implemented API Proxy in Next.js to forward requests to Lambda.
* Fully integrated Post Room form:
  * Submit → API Proxy → API Gateway → Lambda → DynamoDB.
* Completed backend search function (`searchRooms.js` – READ).
* System successfully:
  * Loads website
  * Allows posting rooms
  * Stores data in DynamoDB

