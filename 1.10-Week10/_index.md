---
title: "Worklog Week 10"

weight: 1
chapter: false
pre: " <b> 1.10. </b> "
---

### Objectives of Week 10:

* Set up a mechanism to export data from DynamoDB to S3 for analysis.
* Continue fixing bugs and optimizing the system.

### Tasks Implemented During the Week:

| Day | Tasks                                                                 | Start Date | End Date   | References |
|-----|------------------------------------------------------------------------|-------------|-------------|------------|
| 2   | Research data export mechanisms from DynamoDB to S3                    | 01/12/2025 | 01/12/2025 | https://docs.aws.amazon.com/amazondynamodb/ |
| 3   | Develop script to export DynamoDB data to S3 (JSON/CSV)                | 02/12/2025 | 02/12/2025 | https://docs.aws.amazon.com/AmazonS3/ |
| 4   | Configure scheduling for periodic data export                          | 03/12/2025 | 03/12/2025 | https://docs.aws.amazon.com/scheduler/ |
| 5   | Fix Backend issues: APIs, data handling, authorization                 | 04/12/2025 | 04/12/2025 | |
| 6   | Fix Frontend issues: UI rendering, forms, user experience              | 05/12/2025 | 05/12/2025 | https://nextjs.org/docs |
| 7   | System-wide testing after bug fixing and data export implementation   | 06/12/2025 | 06/12/2025 | |

### Results of Week 10:

* Successfully implemented **data export from DynamoDB to S3**:
  * Data stored in **JSON/CSV format**.
  * Used for **periodic backups** and **data analytics**.
* Completed **Backend and Frontend bug fixing**:
  * Stabilized core APIs.
  * Improved user interface behavior.
* Conducted **full system testing** after updates.
* **Overall Outcome:** The system is more stable, and the data backup & analytics foundation is fully established.
