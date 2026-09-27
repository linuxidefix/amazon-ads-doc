---
title: Terms Token Skill
description: Learn about the Terms Token Skill for handling advertising terms acceptance during account onboarding
type: guide
interface: api
tags:
  - mcp
---
## Terms Token Skill

**What does this skill do?**
This skill guides agents through the advertising terms-of-service acceptance workflow required before the first account of an advertiser can be registered. It handles token creation, user acceptance tracking, and token status polling. Tokens expire 48 hours after creation and are consumed during account registration.

**When to read this skill:**
Read this skill when your agent needs to onboard a new advertiser, initiate terms acceptance before account creation, or check whether a user has already accepted the advertising terms.

* `terms_token-create_terms_token`
* `terms_token-get_terms_token`
