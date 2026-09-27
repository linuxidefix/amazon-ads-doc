---
title: Overview
description: Learn more about design, use cases, and set up for developers using the Amazon Ads API. 
type: guide
interface: api
tags:
    - mcp
---
## Amazon Ads MCP Server Overview

### What is the Amazon Ads MCP Server?

The Amazon Ads MCP Server is a standardized access layer for AI models and agents. It transforms complex multi-field API operations into simple conversational queries, making Amazon Ads insights such as campaigns, performance metrics, billing, and account information accessible to large language models (LLMs) and AI applications through natural language interactions.

**What is Model Context Protocol?**

MCP ([Model Context Protocol](https://modelcontextprotocol.io/docs/getting-started/intro)) is an open-source standard that enables developers to build secure, two-way connections between their data sources and AI-powered applications.

### How it works

**The key participants in the MCP architecture are:**

* **Amazon Ads MCP Server**: The Amazon Ads layer that standardizes and exposes Amazon Ads features and data through tools, resources, and prompts.
* **MCP Client**: The connector that securely manages data exchange between the AI application and the MCP Server.


**Primitives:**

MCP primitives are the fundamental building blocks that define how AI applications and external systems can interact. They specify the types of contextual information that can be shared with AI applications and the range of actions that can be performed.

The Amazon Ads MCP Server translates Amazon Ads APIs into three standardized MCP primitives:

* A **Tool** is a function exposed to an agent with a description, input properties, and return values that allows the agent to perform an action.
* A **Skill** is a pre-defined prompt definition provided by Amazon Ads that guides AI applications through Amazon Ads workflows and tool usage.


Amazon Ads builds, maintains, and updates the MCP Server so that developers and AI applications can consistently access the latest functionality without re-engineering integrations. LLMs can stay up to date with Amazon Ads context, enabling faster innovation and reducing maintenance overhead.

Amazon Ads does not own or operate the LLMs or AI applications that connect through the MCP Server. Developers can connect to the MCP Server using Claude (Anthropic), ChatGPT (OpenAI), Amazon Kiro, Amazon Bedrock, Amazon AgentCore, and other MCP-compatible applications. The Amazon Ads MCP Server provides a standardized bridge between those external AI systems and Amazon Ads. You maintain full control and flexibility over your AI experiences while benefiting from reliable access to Amazon Ads context.

For additional MCP terms and concepts, refer to documentation from [Model Context Protocol](https://modelcontextprotocol.io/docs/learn/architecture).

## Typical use cases

The following are some example prompts and use cases that you can perform using the Amazon Ads MCP Server:

* **Query campaign performance** with requests such as "Show me campaign performance for October 2025 on `[account_id]`"
* **Access account information** by asking "Show me a list of all of my advertising accounts" or "Show me my recent invoices on `[account_id]`"
* **Generate reports** with requests such as "Create a Campaign report for `[account_id]`"
* **Update campaign settings** through simple instructions such as "Increase my campaign budget to $500 on `[campaign_id]`"

**Advanced tools**

* **Create and launch a campaign** in one prompt with requests such as "Create a Sponsored Products campaign in the US and Canada for `[ASIN]` with a $20 budget and `[ASIN]` with a CAD$15 budget."
* **Scale campaigns to new markets** with prompts such as "Add UK to `[campaign_id]` with a £10 budget."
