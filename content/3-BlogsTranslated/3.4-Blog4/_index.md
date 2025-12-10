---
title: "Blog 4"

weight: 1
chapter: false
pre: " <b> 3.4. </b> "
---


# How Zapier runs isolated tasks on AWS Lambda and upgrades functions at scale

by Anton Aleksandrov, Raúl Negrón-Otero, Ankush Kalra, Vítek Urbanec, and Chandresh Patel on 25 JUL 2025 in [Advanced (300)](https://aws.amazon.com/blogs/architecture/category/learning-levels/advanced-300/), [Amazon CloudWatch](https://aws.amazon.com/blogs/architecture/category/management-tools/amazon-cloudwatch/), [Amazon Elastic Kubernetes Service](https://aws.amazon.com/blogs/architecture/category/compute/amazon-kubernetes-service/), [Architecture](https://aws.amazon.com/blogs/architecture/category/architecture/), [AWS Lambda](https://aws.amazon.com/blogs/architecture/category/compute/aws-lambda/), [Customer Solutions](https://aws.amazon.com/blogs/architecture/category/post-types/customer-solutions/), [Monitoring and observability](https://aws.amazon.com/blogs/architecture/category/management-and-governance/monitoring-and-observability/), [Serverless](https://aws.amazon.com/blogs/architecture/category/serverless/) 

[Zapier](https://zapier.com/) is a leading no-code automation provider whose customers use their solution to automate workflows and move data across over 8,000 applications such as Slack, Salesforce, Asana, and Dropbox. Zapier runs these automations through integrations called Zaps, which are implemented using a serverless architecture running on [Amazon Web Services](https://aws.amazon.com/) (AWS). Each Zap is powered by an [AWS Lambda](https://aws.amazon.com/lambda/) fuction.

In this post, you’ll learn how Zapier has built their serverless architecture focusing on three key aspects: using Lambda functions to build isolated Zaps, operating over a hundred thousand Lambda functions through Zapier’s control plane infrastructure, and enhancing security posture while reducing maintenance efforts by introducing automated function upgrades and cleanup workflows into their platform architecture.

## Architecting a secure and isolated runtime environment
Zaps created by Zapier’s users implement tenant-specific business logic, hence they require cross-tenant compute isolation. Code implementing one Zap can’t share an execution environment with code implementing another Zap. Moreover, the same Zap type used by two different tenants can’t share execution environments as well.

To achieve the required level of isolation, Zapier’s engineering team adopted  [AWS Lambda](https://aws.amazon.com/lambda/), a serverless compute service that runs code in response to events and automatically manages cloud compute resources. Minimal [operational overhead](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html), [built-in high availability](https://docs.aws.amazon.com/lambda/latest/dg/security-resilience.html), [automated scaling](https://docs.aws.amazon.com/lambda/latest/dg/lambda-concurrency.html), [high level of isolation](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtime-environment.html), and [pay-per-use model](https://aws.amazon.com/lambda/pricing/) made Lambda a great fit for this use case. Currently, Zapier’s architecture is running over a hundred thousand Lambda functions to support their customer’s integration workflows
Because they’re powered by the open source [Firecracker microVMs](https://firecracker-microvm.github.io/), each function is completely isolated from the others. Moreover, each execution environment belonging to the same function (sometimes referred to as function instances) is also isolated from other execution environments. The following architecture topology diagram uses red lines to represent isolation boundaries. Each execution environment of every function is isolated from its peers and is getting its own virtual resources such as disk, memory, and CPU. For more details, read [Security in AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/lambda-security.html).


<!-- **Kiến trúc giải pháp bây giờ như sau:** -->

> *Image 1. *
 
Zapier’s control plane is architected using [Amazon Elastic Kubernetes Service](https://aws.amazon.com/eks/) (Amazon EKS). A designated database is used to maintain the up-to-date function inventory. Whenever a user creates a new Zap, the control plane creates a corresponding Lambda function and stores a reference in the inventory database. When a Zap is triggered, the control plane retrieves information about a relevant Lambda function and invokes it to facilitate the integration workflow, as illustrated in the following diagram.

> *Image 2. *

## Understanding the runtime deprecation process

When building architectures using the traditional non-serverless compute, cloud engineers are the ones responsible for keeping operating systems and software on their compute instances up to date and applying security and maintenance patches. With serverless architectures and Lambda functions, security patches and minor [runtime](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html) upgrades are handled by AWS automatically, which means customers can focus on delivering business value instead of the undifferentiated heavy lifting of infrastructure management.
When a major Lambda managed runtime version reaches end-of-life, AWS initiates a [deprecation process](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html#runtime-support-policy) through the [AWS Health Dashboard](https://docs.aws.amazon.com/health/latest/ug/aws-health-dashboard-status.html) and direct email communications to affected customers. Because deprecated runtimes eventually lose access to security updates and support, organizations must upgrade to supported runtime versions to avoid potential security risks. Read more about the [shared responsibility model](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html#runtimes-shared-responsibility), [runtime use after deprecation](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html#runtime-deprecation-levels), and [receiving runtime deprecation notifications](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html#runtime-deprecation-notify).

As Zapier’s user base and architectural complexity – and consequently the number of Zaps – were growing, keeping all functions on the most up-to-date major runtime versions became a laborious task. Top contributing factors were:
- High number of functions. At its peak, the Zapier platform was running Zaps using hundreds of thousands of unique Lambda functions. Approximately 35% of these functions were using a runtime that was scheduled for deprecation in the next 12 months.
  
- Zapier architected their data plane environment to be ephemeral – the control plane creates and deletes Lambda functions on demand and manages their lifecycle dynamically. Identifying a specific owner for each affected function wasn’t always straightforward.
  
- Security is paramount at Zapier and upgrading affected functions runtime prior to the deprecation date was an absolute must. At no point could Zapier functions use runtimes after their deprecation date. This was a task which required extra resources.
  
- The upgrade process shouldn’t have had any impact on the end customer experience. At no point should customer experience be affected.
With a short runway, high-volume workload, and the strict requirements of not impacting customer experience, Zapier’s Platform Engineering team took on this challenge of maintaining high security posture in their platform architecture.
### Applying the solution
The solution had three work streams:
1. Reducing the risk by analyzing the architecture and identifying and cleaning up unused functions.
2. Prioritizing upgrades by identifying the most critical and impactful functions.
3. Empowering engineering teams with automated tools and knowledge to streamline the upgrade process in future.
### Identify and clean up unused functions
The first step in streamlining the upgrade process was identifying and removing unused functions. This reduced the total number of functions in Zapier’s architecture that required upgrades, eliminating unnecessary work for the team.
Zapier started by augmenting the function inventory with runtime information using [AWS Trusted Advisor](https://aws.amazon.com/premiumsupport/technology/trusted-advisor/) and [Amazon Cloud Intelligence Trusted Advisor dashboards](https://catalog.workshops.aws/awscid/en-US/dashboards/advanced/trusted-advisor), as illustrated in the following diagram.

> *Image 3. *

This meant the team could build a detailed inventory of functions that were running on soon-to-be deprecated runtimes. Using [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/), Zapier’s platform team started to monitor metrics such as [number of invocations](https://docs.aws.amazon.com/lambda/latest/dg/monitoring-metrics-types.html#invocation-metrics). They identified which functions were active, which functions weren’t used for an extended period, and which functions didn’t have an active owner and could be removed.
One of the primary mechanisms for ownership validation within the organization was using [resource tags](https://docs.aws.amazon.com/whitepapers/latest/tagging-best-practices/what-are-tags.html). Functions that were active, but didn’t have clear ownership, were flagged for additional review before removal. Functions that were confirmed as unused or didn’t have an active owner were marked for deletion. Removing such functions allowed Zapier to significantly simplify their architecture and reduce the number of functions that had to be upgraded.

## Prioritizing upgrades
With a smaller volume of functions to upgrade, Zapier’s platform team prioritized function upgrades based on usage patterns, criticality, and potential customer impact. Three primary prioritization categories were:
- Customer-facing functions – Any functions directly involved in executing user Zaps were marked as high priority. These had to be upgraded first to avoid service disruptions.
  
- Backend infrastructure functions – Internal functions that supported system operations were evaluated based on their importance to platform stability.
  
- High-volume functions – Functions with the highest execution frequency were prioritized because upgrading them would have the greatest impact on reducing operational risk.

Using these factors, Zapier’s platform team has created an upgrade roadmap, ensuring that critical assets were addressed first while minimizing potential disruptions.

Refer to [Retrieve data about Lambda functions that use a deprecated runtime](https://docs.aws.amazon.com/lambda/latest/dg/runtimes-list-deprecated.html) in the Lambda Developer Guide to learn how to identify most commonly and most frequently used Lambda functions in your serverless architecture.

## Empowering engineering teams with automated tools and knowledge
To ensure a smooth and efficient upgrade process across their serverless architecture, Zapier’s team empowered engineering teams with clear guidelines and automated solutions. The platform incorporated two main approaches: Terraform-managed functions and a custom-built Lambda runtime canary tool. Implementing and adopting these tools and practices resulted in reducing the number of functions using soon-to-be deprecated runtimes by 95%.
For functions managed through [infrastructure-as-code](https://aws.amazon.com/what-is/iac/) (IaC), Zapier’s team developed standardized Terraform modules that specified supported runtime versions. Development teams implemented these modules in their configurations:
``` yaml
resource "aws_lambda_function" "example" {
    runtime = "python3.13"  # Updated to supported runtime
}
```

After applying the new module version, teams validated changes by testing the new runtime in staging environments and monitoring Terraform plan outputs to ensure proper runtime version updates.
To efficiently manage most Lambda functions in their architecture, Zapier developed the Lambda runtime canary tool suite. Using this solution, they automated the runtime upgrade process for thousands of active Lambda functions with minimal manual intervention. The tool suite implements several key features:

- Architected for gradual traffic shifting with the Lambda built-in routing mechanism through function [version](https://docs.aws.amazon.com/lambda/latest/dg/configuration-versions.html) and [aliasing](https://docs.aws.amazon.com/lambda/latest/dg/configuration-aliases.html). The tool can gradually shift traffic distribution from an old to a new function version. During this gradual traffic shift, the system monitors [CloudWatch metrics](https://docs.aws.amazon.com/lambda/latest/dg/monitoring-metrics-types.html) for errors and automatically rolls back if error rates exceed acceptable thresholds.
  
- Optimistic upgrade strategy implements direct upgrades for infrequently used functions using a flag value stored in a cache to detect potential issues during the first post-upgrade invocation. If this invocation fails, the control plane retries it using the previous function version. If the retried invocation succeeds, Zapier’s control plane initiates a rollback, assuming the error is most likely due to the runtime upgrade. After rollback, it will log the error and alert relevant stakeholders.

- Integration with existing infrastructure uses an administrative interface and task queue for automated traffic shifting. A database ledger maintains tracking of function states and rollback information.
  
- Operational controls provide manual rollback capabilities and implement centralized control switches for process management. After a function was upgraded to a new runtime and no rollback activity was detected within a set time period, an automated pruning task cleans up older versions.
  
Zapier’s Lambda canary tool, through its integration of gradual traffic shifting, real-time CloudWatch monitoring, and automated rollback mechanisms, established a sustainable framework for managing runtime upgrades across their serverless architecture. This approach not only automated the upgrade process and minimized operational risks but also created a scalable solution that provides continuous runtime upgrades, preventing the use of deprecated runtimes at any point. By allowing continuous function runtime updates with minimal disruption to end user experience, Zapier maintains security and stability while requiring minimal manual intervention. This framework efficiently manages their growing serverless infrastructure, providing both security and operational efficiency for future runtime updates.

## Conclusion

In this post, you’ve learned how Zapier architected their [software-as-a-service](https://aws.amazon.com/what-is/saas/) (SaaS) platform to provide secure, isolated execution environments using AWS Lambda and Amazon EKS, enabling their customers to create hundreds of thousands of Zaps. You’ve learned how Zapier’s team implemented the function runtime upgrade process at scale and reduced the number of functions running on soon-to-be deprecated runtimes by 95%. You’ve seen best practices that were established and techniques that helped Zapier to keep high security posture without impacting customer experience.
Use the following links to learn more about Lambda runtimes and upgrading your functions to the latest runtime versions:
- [Lambda runtimes documentation](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html)
- [Retrieve data about Lambda functions that use a deprecated runtime](https://docs.aws.amazon.com/lambda/latest/dg/runtimes-list-deprecated.html)
- [Managing AWS Lambda runtime upgrades](https://aws.amazon.com/blogs/compute/managing-aws-lambda-runtime-upgrades/) in the AWS Compute Blog
- [AWS Lambda runtime management controls](https://aws.amazon.com/blogs/compute/introducing-aws-lambda-runtime-management-controls/) in the AWS Compute Blog
