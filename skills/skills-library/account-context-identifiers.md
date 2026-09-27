---
title: Account Context Identifiers Skill
description: Learn about how account context and identifiers work
type: guide
interface: api
tags:
  - mcp
---
## Account Context Identifiers Skill

**What does this skill do?**
This skill maps every Amazon Ads API tool to the correct account identifier it requires—`profileId`, `advertiserAccountId`, or `managerAccountId`—and spells out which HTTP header carries each one. It also flags common mix-ups, like accidentally passing an Entity ID or Profile ID where an `advertiserAccountId` (the `amzn1.ads-account.g.…` string) is expected.

**When to read this skill:**
Read this skill before making any account-scoped Amazon Ads API call. If you are unsure which ID type a tool needs, or which header to set, consult the lookup tables here first to avoid misrouted requests and hard-to-debug auth errors.
