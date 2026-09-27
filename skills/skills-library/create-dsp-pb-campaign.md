---
title: Create DSP P+/B+ Campaign Skill
description: Learn about the Create DSP Performance+ or Brand+ Campaign Skill
type: guide
interface: api
tags:
  - mcp
---
## Create DSP P+/B+ Campaign Skill

**What does this skill do?**
This workflow skill lets you create and launch a DSP Performance+ (P+) or Brand+ (B+) campaign through a single conversational request. It handles the full setup — from configuring your campaign budget and schedule, to linking your `ASIN`s for conversion tracking, to creating ad groups for each eligible tactic and inventory type. Once everything is in place, it activates the campaign so it is ready to deliver. The skill supports P+ tactics such as customer acquisition, remarketing, retention, and maximize performance, as well as B+ prospecting, across inventory types including display, online video, and streaming TV.

**When to read this skill:**
Your agent should read the Create DSP P+/B+ Campaign Skill before creating or launching a DSP Performance+ (P+) or Brand+ (B+) campaign.

**Example prompt:** "Create a DSP P+ campaign for my products (`ASIN`s B0EXAMPLE1 and B0EXAMPLE2) in the US with a $500 budget, starting tomorrow, targeting `DISPLAY` inventory."

The P+/B+ Skill guides the agent to call the appropriate MCP tools for executing the workflow below:

1. Gathers inputs from the user request
2. Creates DSP campaign with the specified goal type
3. Sets `ASIN` conversion tracking for the provided `ASIN`s
4. Queries eligible tactics and selects those matching the user request
5. Creates ad groups for the selected tactics
6. Asks user about creatives
7. Activates campaign and ad groups
8. Presents summary with campaign ID, ad group IDs, and activation status
