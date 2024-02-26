---
title: "Detailed Cloud and DevOps Experience report, 2013-"
date: 2023-04-01T17:04:33-05:00
draft: false
---

This is a sampling of some of the projects I’ve completed in the past 10+ years.
Most of the following projects were done for large enterprises and Fortune 500 companies.

- Designed and built a startup's foundational multi-account architecture in AWS using Infrastructure as Code tooling (CloudFormation & Troposphere), configuration management (Puppet), and automation (scripting in Bash, Python, etc.). This was prior to landing zone or Control Tower solutions being commonly available.

- Built automation (in Python) to extract data from Excel spreadsheets and generate Terraform variables files that plugged into Terraform templates that could take an unknown # of elements (i.e. 3 EC2 instances, 2 S3 buckets and 0 RDS databases). This code was built prior to Terraform having looping capabilities.
  Built similar automation targeting a wide set of core native services in both AWS and Azure.

- Used Terraform as the underlying IaC language to build products in AWS Service Catalog.

- Security audit of AWS environment, Prisma rulesets, and analysis of pipeline-based deployment of IaC code with recommendations for automatic security scans using a policy-as-code tool (Bridgecrew’s Checkov).

- Deployed and updated multi-account foundational architectures using the AWS Landing Zone solution (prior to the availability of the Control Tower service).

- Deployed and updated multi-account foundational architectures using the AWS Control Tower service.

- Migration of a large-scale application to AWS, architected with a Disaster Recovery (DR) failover region. Application was a complex, multi-service architecture spread across over 20 EC2 servers running both Windows & Linux, multiple S3 buckets, an RDS Oracle database, an FSx share and other ancillary services.

  - Terraform codebase for production environment of over 3000 SLoC.
  - Over 200 TB of data was transferred and synchronized from on-prem.
  - Documented detailed DR failover runbook and successfully tested failover of the production environment in AWS from the primary region to the DR region.

- Wrote a Python3 package that reads in a set of GCP labels (from a YAML file), compares it to the resources in a GCP Project(s) and updates the labels on those resources. This is valuable for labeling un-labeled resources in projects for the purposes of billing, auditing, reporting, etc.

- Built a custom GitHub Actions pipeline for validating, planning and applying Terraform IaC with a Pull Request-based authorization before the Apply phase. The pipeline built environments based on whether or not there were actual changes to code and/or variables.

- Built the infrastructure (using Terraform IaC) for a data lake environment that included the use of these AWS services: S3, Redshift, SFTP, Lambda, Glue, Athena, SageMaker and more.

- Coded system to deploy a SQL Server cluster running in AWS on EC2 servers. This used a combination of Terraform, SSM documents (AWS Systems Manager), and PowerShell DSC to install and configure SQL Server.
