---
title: Taking AWS Migration Siriusly
date: 2023-06-06T19:41:52+02:00
draft: true
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

## Migration Preparation

>  By failing to prepare, you are preparing to fail — Benjamin Franklin

Preparation is instrumental to the success of your migration effort.
It's common for organizations to set up a "task force" for migration initiatives as the application owning team might not have experience with AWS or cloud migrations, or have insight into how things are done in other applications or business units in the organization.
A typical team composition might include a facilitator (project manager/scrum master/team lead), engineers from the application owning team (and from the team that operates it if not the same team), a solutions architect familiar with the application, a cloud solutions architect and some migration experts (for example a database migration expert from AWS or an AWS Partner).

Start by introducing the team members to each other, agreeing on roles and responsibilities, and defining the scope of the migration.
The team needs to work together like a well-oiled machine to migrate as efficiently as possible.
It's unlikely that the team will be doing everything right from the get-go so promote a team culture that instills the value of close collaboration, effective communication and constant feedback.

### Define Your Objectives

Be explicit about what you are trying to accomplish. What does a successful migration look like? Define [SMART](https://www.atlassian.com/blog/productivity/how-to-write-smart-goals) objectives and document them somewhere where everyone in the team can view them.
Any effort by any team member should be directly linked to one or more of the objectives. If it's not, then that effort should be directed elsewhere.

{{< notice tip >}} Some objectives might contradict each other. Decide which objective is more important and make the tradeoff explicit. {{< /notice >}}

One of the most common tradeoffs that you will make is that of balancing the time needed to complete the migration with the benefits of using cloud native services.
Migrations are usually very costly for organizations because they need to pay for both the old and the new environments. This period of time is sometimes referred to as the "migration bubble."
For organizations that have their own data centers it's even more costly because decommissioning the application on-premises doesn't necessarily reduce cost since the physical infrastructure cannot be decommissioned.
The smaller the migration bubble, the more cost-effective the migration. If there are tight deadlines for the migration focus on migrating the applications with as few changes as possible (ie. rehost/lift and shift) and then iteratively make improvements to the application architecture once you are in the cloud.

Checklist:
- Are the objectives aligned with the business case for the migration?
- Are the objectives SMART?
- Are any tradeoffs explicitly noted?
- Are the objectives documented somewhere?
- Are all the team members aware of the objectives?

### Take Stock

Take stock of both the source and destination environments

Reference architecture diagram from the Bank of Sirius repo:
![Architecture Diagram](https://github.com/nginxinc/bank-of-sirius/raw/master/docs/architecture.png)

### Decide on Migration Strategy

Reference the different migration strategies
https://docs.aws.amazon.com/prescriptive-guidance/latest/large-migration-guide/migration-strategies.html

### Plan



## Migration Execution

Reference the [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/)

### Architecture


