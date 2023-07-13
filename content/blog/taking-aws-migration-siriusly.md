---
title: Taking AWS Migration Siriusly
date: 2023-06-06T19:41:52+02:00
draft: false
description:
slug:
authors: []
tags: []
categories: []
externalLink:
series: []
---

I've been involved predominantly in migration projects recently and have noticed some common mistakes and where there were gaps in the migration initiatives.
The goal of this post is to go through a small migration from start to finish to show the process and highlight considerations for every step of the way.
We will not cover building of business cases for cloud migration and will work on the presumption that you have already decided to migrate to the cloud.

This post will be divided into two parts: The first will be about migration preparation, aimed at both technical and non-technical audiences and will mostly be vendor-agnostic.
The second will be a walkthrough of the migration of a sample application. This will be very technical and assumes basic knowledge of AWS and containerized applications, and familiarity with the CLI.
The sample application that we will be migrating is called [Bank of Sirius](https://github.com/nginxinc/bank-of-sirius) (hence the title) which is a "productionized" fork of the popular [Bank of Anthos](https://github.com/GoogleCloudPlatform/bank-of-anthos) project.
I chose this application as it is open source, relatively small and a good representation of what you might be tasked with migrating in practice.

{{< table-of-contents >}}

## Migration Preparation

>  By failing to prepare, you are preparing to fail<br>
> — <cite>Benjamin Franklin</cite>

Preparation is instrumental to the success of your migration effort.
It's common for organizations to set up a "task force" for migration initiatives as the application owning team might not have experience with AWS or cloud migrations, or have insight into how things are done in other applications or business units in the organization.
A typical team composition might include a facilitator (project manager/scrum master/team lead), engineers from the application owning team (and from the team that operates it if not the same team), a solutions architect familiar with the application, a cloud solutions architect and some migration experts (for example a database migration expert from AWS or an AWS Partner).

Start by introducing the team members to each other, agreeing on roles and responsibilities, and defining the high-level scope of the migration.
The team needs to work together like a well-oiled machine to migrate as efficiently as possible.
It's unlikely that the team will be doing everything right from the get-go so promote a team culture that instills the value of close collaboration, effective communication and constant feedback.

### Define Your Objectives

Be explicit about what you are trying to accomplish. What does a successful migration look like? Define [SMART](https://www.atlassian.com/blog/productivity/how-to-write-smart-goals "How to write SMART goals") objectives and document them somewhere where everyone in the team can view them.
Any effort by any team member should be directly linked to one or more of the objectives. If it's not, then that effort should be directed elsewhere.

{{< notice tip >}} Some objectives might conflict with one other. Decide which objective is more important and why, then make the tradeoff explicit. {{< /notice >}}

One of the most common tradeoffs that you will make is that of balancing the time needed to complete the migration with the benefits of using cloud native services.
Migrations are usually very costly for organizations because they need to pay for both the old and the new environments. This period of time is sometimes referred to as the "migration bubble."
For organizations that have their own data centers it's even more costly because decommissioning the application on-premises doesn't necessarily reduce cost since the physical infrastructure cannot be decommissioned.
The smaller the migration bubble, the more cost-effective the migration. If there are tight deadlines for the migration, focus on migrating the applications with as few changes as possible (ie. rehost/lift and shift) and then iteratively make improvements to the application architecture once you are in the cloud.

Checklist:
- Are the objectives aligned with the business case for the migration?
- Are the objectives SMART?
- Are any tradeoffs explicitly noted?
- Are the objectives documented somewhere?
- Are all the team members aware of the objectives?

### Take Stock And Finalize Scope

Now that we have the objectives defined it's time to flesh out the scope.
We need to build an inventory of every component that forms part of the application and any supporting components.
A good starting point would be to reference architecture diagrams and engage with the owning team.

The Bank of Anthos project provides a logical architecture diagram depicting the web server, five APIs and two database servers:

![Architecture Diagram](https://github.com/nginxinc/bank-of-sirius/raw/master/docs/architecture.png "Bank of Anthos Conceptual Architecture Diagram")

This gives us a good idea of how the application works but for the purposes of a migration we want to drill down and see the physical resources.
There are also many components that need to be considered that aren't shown on the architecture diagram.
From a networking perspective you need to understand the flow of ingress and egress traffic, considering things like DNS servers and entries, service discovery mechanisms and routing (among others).
Physical placement of resources needs to be considered as some workloads require tightly coupled node-to-node communication and require low latency more than high availability.
Some security considerations include understanding whether there are any firewalls in the ingress or egress flows, where TLS termination is done, or whether there is required remote access, anti-malware or intrusion detection software.
Additional things to consider include the monitoring systems, the CI/CD tools and any fault tolerance or disaster recovery mechanisms.

There are a number of tools at our disposal to make the stocktaking of the application's physical resources a little easier:
1. The organization might already maintain an inventory of these components in a configuration management database as part of their IT service management processes.
2. Most cloud providers and virtualization platforms support some form of tagging which can also be used to find the components belonging to an application.
3. Services like AWS Migration Hub and Azure Migrate come with a range of tools to automatically discover workloads and integrations used by those workloads.

Discovery tools have the benefit of being a truer reflection of the systems at that point in time; Documentation and diagrams may become out-of-date and teams don't always tag resources very diligently.

By using a combination of these tools and information sources, we will be able to gather enough information to produce a bill of resources and therefore define the scope.


CMDB[^1]

Kubernetes labels and AWS tags also make it easy to find everything you need

In the case of the Bank of Sirius we have an architecture diagram and the infrastructure is defined as code.

Reference architecture diagram from the Bank of Sirius repo:

Checklist:
- Is the bill of resources documented somewhere?
- Has the team gone through the bill of resources and understood all the moving parts?

### Decide on Migration Strategy
Example footnote[^2]
Reference the different migration strategies


https://docs.aws.amazon.com/prescriptive-guidance/latest/large-migration-guide/migration-strategies.html
https://www.infosys.com/about/knowledge-institute/insights/documents/cloud-migration.pdf


Rehost - Could go to EKS on Fargate - That would be the fastest and would still technically be serverless
Replatform from GKS to ECS
Re-architect - Go to Lambdas
Refactor could be using DynamoDB instead of Postgresql or introducing something like SQS/MSK for processing transactions
Service discovery built in to Kubernetes now needs to be handled by CloudMap
Ingress route replaced with a load balancer and target group associated with the service
DNS moves to Route53
Move from GCR to ECR



### Plan



SLI/SLO/SLAs and downtime
MTTR (Or reliability target, error budget), RPO and RTO
Architecture
Changes to CI/CD
Change management processes? Or ITSM in general?
DR exercises?

## Migration Execution

Reference the [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/)

### Architecture



### Init


Generate an RSA key-pair for use in the microservices as documented here: https://github.com/DewaldDeJager/anthos-on-aws/tree/main/extras/jwt

```shell
copilot init --app bank-of-sirius --name frontend --type "Load Balanced Web Service" --image ghcr.io/nginxinc/bos-frontend:v1.3.0 --port 8080
```

You will be prompted to deploy the service to a test environment - Skip that for now.
The `init` command created a basic manifest in `copilot/frontend/manifest.yml` that we can build on.
Since the Bank of Anthos project was built to be deployed to Kubernetes we're fortunate enough to have the infrastructure defined as code.
By referencing the Kubernetes manifest files we can implement the same configuration in ECS.

Let's use the information from the Kubernetes manifest for the web server, [`frontend.yaml`](https://github.com/DewaldDeJager/anthos-on-aws/blob/main/kubernetes-manifests/frontend.yaml), to make some changes to our Copilot manifest:

1. Add the `http.healthcheck` field with a value of `/z/health/readiness`. This will configure the application load balancer health checks.
2. Add the `network.vpc.placement.subnets` field and specify the subnet IDs for the private subnets.
   By default, Copilot creates a VPC and deploys the service to the public subnets (to avoid NAT gateway costs).
   In this case our networking infrastructure is already set up and we want to follow best practice so we deploy to the private subnets.
3. Add the environment variables from the Kubernetes manifest to the `variables` map.

And this:
```yaml

```


#### Footnotes


[^1]: Example footnote
[^2]: Derp
