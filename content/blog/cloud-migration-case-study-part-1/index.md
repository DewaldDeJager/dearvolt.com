---
title: "Cloud Migration: A Case Study (Part I)"
date: 2023-06-06T19:41:52+02:00
draft: false
description:
slug:
authors: []
tags: ["cloud computing", migration, AWS]
categories: []
externalLink:
series: ["A Cloud Migration Case Study"]
type: posts
---

I've been involved predominantly in migration projects recently and have noticed some common mistakes and where there were gaps in the migration initiatives.
The goal of this post is to go through a small migration from start to finish to show the process and highlight considerations for every step of the way.
We will not cover the building of business cases for cloud migration and will work on the presumption that you have already decided to migrate to the cloud.

This post will be divided into two parts: The first will be about migration preparation, aimed at both technical and non-technical audiences and will mostly be vendor-agnostic.
The second will be a walkthrough of the migration of a sample application. This will be very technical and assumes basic knowledge of AWS and containerized applications, and familiarity with the CLI.
The sample application that we will be migrating is called [Bank of Sirius](https://github.com/nginxinc/bank-of-sirius) (hence the title) which is a "productionized" fork of the popular [Bank of Anthos](https://github.com/GoogleCloudPlatform/bank-of-anthos) project.
I chose this application as it is open source, relatively small and a good representation of what you might be tasked with migrating in practice.

## Migration Preparation

>  By failing to prepare, you are preparing to fail<br>
> — <cite>Benjamin Franklin</cite>

Preparation is instrumental to the success of your migration effort.
It's common for organizations to set up a "task force" for migration initiatives as the application owning team might not have experience with AWS or cloud migrations, or have insight into how things are done in other applications or business units in the organization.
A typical team composition might include a facilitator (project manager/scrum master/team lead), engineers from the application owning team (and from the team that operates it if not the same team), a solutions architect familiar with the application, a cloud solutions architect and some migration experts (for example a database migration expert from AWS or an AWS Partner).

Start by introducing the team members to each other, agreeing on roles and responsibilities, and defining the high-level scope of the migration.
The team needs to work together like a well-oiled machine to migrate as efficiently as possible.
It's unlikely that the team will be doing everything right from the get-go so promote a team culture that instills the value of close collaboration, effective communication and constant feedback.

### The Stages of Migration

1. Define Objectives
2. Discover
3. Plan
4. Validate
5. Execute
6. Finalize

### Define Your Objectives

Be explicit about what you are trying to accomplish. What does a successful migration look like? Define [SMART](https://www.atlassian.com/blog/productivity/how-to-write-smart-goals "How to write SMART goals") objectives and document them somewhere where everyone in the team can view them.
Any effort by any team member should be directly linked to one or more of the objectives. If it's not, then that effort should be directed elsewhere.

{{< notice tip >}} Some objectives might conflict with one other. Decide which objective is more important and why, then make the tradeoff explicit. {{< /notice >}}

One of the most common tradeoffs that you will make is that of balancing the time needed to complete the migration with the benefits of using cloud native services.
Migrations are usually very costly for organizations because they need to pay for both the old and the new environments. This period of time is sometimes referred to as the "migration bubble."
For organizations that have their own data centers it's even more costly because decommissioning the application on-premises doesn't necessarily reduce cost since the physical infrastructure cannot be decommissioned.
The smaller the migration bubble, the more cost-effective the migration. If there are tight deadlines for the migration, focus on migrating the applications with as few changes as possible (ie. rehost/lift and shift) and then iteratively make improvements to the application architecture once you are in the cloud.

{{< notice example Checklist >}}
- Are the objectives aligned with the business case for the migration?
- Are the objectives SMART?
- Are any tradeoffs explicitly noted?
- Are the objectives documented somewhere?
- Are all the team members aware of the objectives?
{{< /notice >}}

### Take Stock And Finalize Scope

Now that we have the objectives defined it's time to flesh out the scope.
We need to build an inventory of every component that forms part of the application and any supporting components.
A good starting point would be to reference architecture diagrams and engage with the owning team.

{{< notice tip >}} Focus on the as-is or current state in this step. The idea is not to start planning the future state but to gain a clear and deep understanding of the current state. {{< /notice >}}

The Bank of Anthos project provides a logical architecture diagram depicting the web server, five APIs and two database servers:

{{< figure src="https://github.com/DewaldDeJager/anthos-on-aws/blob/c1dec7826735954bdf9bcddbfa778eb506b02d4f/docs/img/architecture.png?raw=true" caption="Bank of Anthos Conceptual Architecture Diagram." attr="Google Cloud" >}}

This gives us a good idea of how the components of the application work with one another, but for the purposes of a migration we want to drill down and see the physical resources.

There are a number of tools at our disposal to make the stocktaking of the application's physical resources a little easier:
1. The organization might already maintain an inventory of these components in a configuration management database as part of their IT service management processes.
2. Most cloud providers and virtualization platforms support some form of tagging which can also be used to find the components belonging to an application.
3. Services like AWS Migration Hub and Azure Migrate come with a range of tools to automatically discover workloads and integrations used by those workloads.

Discovery tools have the benefit of being a truer reflection of the systems at that point in time; Documentation and diagrams may become out-of-date and teams don't always tag resources very diligently.

There are also many components that need to be considered that aren't shown on the architecture diagram.
These components may be exclusive to the application being migrated or may be shared by multiple applications. Some examples include:
- Networking components
  - The components involved in the flow of ingress and egress traffic
  - DNS servers and entries
  - Service discovery mechanisms
  - Routing configuration and rules
- Security components
  - Understanding whether there are any firewalls in the ingress or egress flows
  - Encryption of data at rest and in transit (including things like provisioning of TLS certificates and TLS termination)
  - Remote access, anti-malware or intrusion detection software
- Database components
  - Views
  - Indexes
  - Stored procedures
- Observability components and operational tools
  - How are logs, metrics and traces collected, stored and used?
  - Dashboards and tools used to query the telemetry data
  - Runbooks and other operational documentation
- Compliance with laws and regulations
- CI/CD tools used to build and deploy the application
- Any fault tolerance or disaster recovery mechanisms

By using a combination of tools and information sources, we will be able to gather enough information to produce a bill of materials.
It's also important to explicitly list things that are out of scope but that may be presumed to be in scope.

In the case of the Bank of Anthos we have an architecture diagram, the application is containerized and the infrastructure is defined as code.
This saves an enormous amount of time in the discovery phase.
The robustness and portability we gain from using virtualization (containers in this case) allows us to deploy the application to a new environment with little room for error as all the needed components are bundled together.
We're also able to set up new infrastructure with minimal complications as we can easily reference the code to understand the old infrastructure.

To fully understand what needs to be done as part of the migration, we need to look at the current state of the infrastructure being migrated to.
Organizations will typically set up an AWS landing zone prior to migrating workloads.
Teams will be granted access to their application accounts (and any other relevant accounts in a multi-account landing zone).
The below architecture diagram depicts the current state of the AWS account we will be migrating to:

{{< figure src="nonprod-aws-account.png" caption="Architecture diagram showing the current state of the destination AWS account" >}}

By looking at the configuration of all the components, we can identify the work that needs to be done to get to the desired state.
The architecture diagram shows only two availability zones.
Since each region has at least three availability zones, this indicates that the VPC depicted is not the default VPC created by AWS and that we need to review how everything has been configured.
We also need to make sure that the team has the access needed to perform the migration[^5].
Some services, such as AWS CloudTrail and AWS Config, may already be set up with some default organization-wide configuration.
AWS Config may be configured to automatically remediate non-compliant resources, so it is a good idea to go through the rules configured in the account.

{{< notice example Checklist >}}
- Is the bill of materials documented somewhere?
- Does the bill of materials specify which components are exclusively used by the application being migrated and that can safely be decommissioned post-migration?
- Does the bill of materials include information about the teams that own the materials or that are needed to make changes to any of the components?
- Have any components that will no longer be needed explicitly been noted as out of scope?
- Has the team gone through the bill of materials and understand the scope of the migration project?
- Has the team analyzed the account(s) that the application will be migrated to and noted any potential gaps?
- Has the team revised the migration objectives to ensure they are still attainable now that the scope is well defined?
{{< /notice >}}

### Decide on Migration Strategy

With our objectives defined and a clear understanding of the application's current state, it's time to start planning the future state.
The 7 Rs[^1] [^2] are widely used in industry and provide a shared vocabulary that can be used to discuss different approaches to migrating an application.

Each of the migration strategies will require varying amounts of investment and will have different rates of return.
The investment is the time and effort put into the migration as well as the inherent risk of each strategy.
This risk of each strategy is also variable depending on the skills available in the teams performing the migration.
The return on investment is typically in the form of reduced operational expenditure and a lower total cost of ownership but can also be indirect such as increased change velocity of teams and decrease in emergency response times.

{{< figure src="migration-strategies.jpg" >}}

It's important to consider the business case, the migration objectives, the current state and the future state when deciding on migration strategies[^3].
Migration doesn't have to be, and generally shouldn't be, a "big bang" type of release.
Doing a migration iteratively results in less risk, allows for agility and lets the business capitalize on the benefits of the migration sooner.
The majority of migrations will use the rehost or replatform strategies.
It's common to first rehost or replatform an application to shorten the migration bubble and then to rearchitect it once in the cloud.

{{< notice tip >}} Migrate then modernize. {{< /notice >}}

We want to assess every component in the Bank of Anthos bill of materials and choose the most appropriate migration strategy.
The Bank of Anthos services are containerized and running in a Kubernetes cluster.
AWS offers a number of container services [^4] but due to the number of containers needed to support a production environment and the amount of control needed, we will focus on the container orchestration offerings.

The easiest migration strategy would be to _relocate_.
By relocating the resources to Amazon EKS, the migration requires minimal effort and lowers the risk of the migration.
EKS can use Fargate as a serverless compute capacity provider reducing the operational overhead.
Additionally, the idle on-premises hardware can be leveraged using Amazon EKS Anywhere until it is decommissioned, minimizing the cost of the migration bubble.

While Kubernetes is incredibly powerful and extensible, it can be complex and requires special skills to successfully operate a production cluster.
Amazon ECS provides a good balance between control and ease of use, and integrates well with other AWS services.
ECS can use Fargate as a capacity provider and there is also an ECS Anywhere offering.
Migrating the services from Kubernetes to ECS would be a _replatform_, meaning there is more effort and risk than relocating, but there is also significantly more value.

{{< notice warning >}} The remainder of this blog post is a work in progress. {{< /notice >}}

Relocate - Could go to EKS on Fargate - That would be the fastest and would still technically be serverless
Rehost is going from OpenShift for example
Replatform from GKS to ECS
Re-architect - Go to Lambdas
Refactor could be using DynamoDB instead of Postgresql or introducing something like SQS/MSK for processing transactions
Service discovery built in to Kubernetes now needs to be handled by CloudMap
Ingress route replaced with a load balancer and target group associated with the service
DNS moves to Route53
Move from GCR to ECR

Data migration strategies? Backup and restore, backup and restore with some availability, continuous replication

### Plan and Validate

This is where the majority of the time will be spent.

Testing tools can do what you need them to do
Database migration, especially heterogeneous
ECS mounting a secret as a file isn't possible - Had to change the code

SLI/SLO/SLAs and downtime
MTTR (Or reliability target, error budget), RPO and RTO
Architecture
Changes to CI/CD
Change management processes? Or ITSM in general?
DR exercises?
Test strategy? eg. ingress traffic

Do all team members have the access needed?


#### Footnotes

[^1]: See the AWS prescriptive guidance article on large [migration strategies](https://docs.aws.amazon.com/prescriptive-guidance/latest/large-migration-guide/migration-strategies.html)
[^2]: The 8th R, remediation, is not widely accepted to be a migration strategy but rather an optional migration task. This task entails tackling technical debt such as updating operating system versions. See the [InfoSys whitepaper](https://www.infosys.com/about/knowledge-institute/insights/documents/cloud-migration.pdf) or [the Azure Cloud Adoption Framework documentation](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/migrate/migration-considerations/migrate/remediate) for more details.
[^3]: A useful flowchart that can be used to choose a migration strategy can be found in the AWS prescriptive guidance article on [application portfolio assessment](https://docs.aws.amazon.com/prescriptive-guidance/latest/application-portfolio-assessment-guide/prioritization-and-migration-strategy.html#migration-r-type)
[^4]: See this article on [choosing an AWS container service](https://aws.amazon.com/getting-started/decision-guides/containers-on-aws-how-to-choose/)
[^5]: This includes making sure all team members have access to assume the appropriate roles and ensuring that the roles have sufficient permissions to perform migration tasks. The [IAM Policy Simulator](https://policysim.aws.amazon.com) is a helpful tool for testing policies.
