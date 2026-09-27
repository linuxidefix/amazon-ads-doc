---
title: Overview
description: Learn more about design, use cases, and set up for developers using the Amazon Ads API. 
type: guide
interface: api
tags:
    - mcp
---
## Skills

### What are Skills?

Skills are pre-defined prompt definitions provided by Amazon Ads that guide AI applications through Amazon Ads workflows and tool usage. There are two types of skills:

- **Workflow Skills** orchestrate multi-step processes by combining one or more MCP tools into structured, repeatable patterns. They handle tool sequencing, error recovery, and result formatting — so users can accomplish complex tasks through a single natural language request without needing to understand the underlying tool orchestration.
- **Capability Skills** provide informational guidance on how to effectively use specific tools or features. Rather than orchestrating actions, they describe tool capabilities, explain input/output formats, and offer best practices — helping AI applications generate correct tool calls and interpret results accurately.

### How Skills Work

Skills are triggered automatically by compatible AI applications (such as Claude Code) based on the context of your request. 

For example:

* The `dsp-campaign-analysis` workflow skill activates when a user asks something like **"How did my Holiday Promo Q4 campaign perform?"** Behind the scenes, it orchestrates a multi-step workflow: resolving the campaign name to an ID, creating two separate performance reports (one for `ROAS`/sales metrics and one for audience segment breakdowns), polling until both reports complete, and presenting a combined analysis — all from a single conversational request.
* Asking **"Show me my campaign performance for last month"** triggers the `ads-reporting` skill, which handles report creation, polling, and retrieval on your behalf.

Skills abstract away multi-step tool orchestration and tool usage complexity so you can focus on your advertising goals rather than API mechanics.

> [NOTE] Skills are supported in AI applications that recognize Anthropic skill definitions. Availability may vary by client.
