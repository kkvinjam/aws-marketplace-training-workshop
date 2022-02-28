---
weight: 3
title: Config
description: Labs for AWS Config
---

#### Introduction

AWS Config is a service that enables you to assess, audit, and evaluate the configurations of your AWS resources. Config continuously monitors and records your AWS resource configurations and allows you to automate the evaluation of recorded configurations against desired configurations. With Config, you can review changes in configurations and relationships between AWS resources, dive into detailed resource configuration histories, and determine your overall compliance against the configurations specified in your internal guidelines. 

Config helps accelerate compliance with governance frameworks such as PCI DSS, SOC 2, SOC 3, and others.

In this lab, we will enforce compliance by creating Config rules, and create State Manager associations that ensure we have complied with a given security requirement. The rules created here are only a small sample of those that AWS provides as managed rules. For a complete list of managed rules see [here](https://docs.aws.amazon.com/config/latest/developerguide/managed-rules-by-aws-config.html). 

#### Important Note

**AWS routinely updates our console experience, so it is possible that the screenshots and guidance provided in these labs may differ slightly from what you experience.**

#### Labs

[Setup](setup)

[Config Rule with Remediation](configrule-with-remediation)

[Config Rule with Lambda Trigger](configrule-with-lambda)

[Resource details and CloudWatch](systems-manager-and-cloudwatch)

[Cleanup](cleanup)

[Advanced labs](dev/builder/reInvent2021/cop322/content/labs/config/advanced)