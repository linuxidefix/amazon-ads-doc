---
title: AMC Workflows Skill
description: Use the AMC Workflows skill to create, execute, and monitor marketing analytics queries in Amazon Marketing Cloud instances
type: guide
interface: api
tags:
  - mcp
---
## AMC Workflows Skill

**What does this skill do?**
[Amazon Marketing Cloud](https://advertising.amazon.com/solutions/products/amazon-marketing-cloud) (AMC) is a secure, privacy-safe, and cloud-based clean room solution in which advertisers can easily perform analytics and build audiences across pseudonymized signals, including Amazon Ads signals as well as their own inputs.

The AMC workflow management service provides you the ability to create, store, and execute parameterized workflows. This skill helps agents use the `amc-get_workflows` tool to discover, run, monitor, and retrieve results from saved workflows in AMC. Learn more about [AMC workflows](https://advertising.amazon.com/API/docs/en-us/guides/amazon-marketing-cloud/reporting/workflow-management-service).
 
Note: This skill does not create new workflows. It can list and execute saved workflows in AMC accounts. 

**When to read this skill:**
Your agent should read the AMC Workflow Skill before invoking any of the following tools:

* `amc-get_workflows`
* `amc-get_workflow`
* `amc-get_workflow_execution_status`
* `amc-get_workflow_execution_download_url`