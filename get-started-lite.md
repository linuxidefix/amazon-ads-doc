---
title: Connecting to the Amazon Ads MCP Server (Lite)
description: Learn how to connect to and use the Lite version of the Amazon Ads Model Context Protocol (MCP) Server, which exposes a compact three-tool interface for agentic access to Amazon Ads
type: guide
interface: api
tags:
  - mcp
---

# Connecting to the Amazon Ads MCP Server (Lite)

## **What is the Lite version?**

The **Lite** version of the Amazon Ads MCP Server exposes the same Amazon Ads capabilities as the Regular version, but through a compact **three-tool interface** instead of registering every operation as its own tool. Rather than loading the full tool catalog into your model's context, the Lite server gives your agent three meta-tools that let it *discover*, *inspect*, and *run* any underlying Amazon Ads operation on demand:

* **`amazon_ads_mcp-search_tools`** — Search the Amazon Ads tool catalog with a natural-language query. The response returns the tools that directly match **plus the other tools in each matched domain (namespace)**, each with its name, namespace, and description — so a single search surfaces the whole domain the request falls into, not just the closest hits.
* **`amazon_ads_mcp-describe_tools`** — Retrieve the full input schema for one or more tools returned by search, so the agent knows exactly which fields to populate.
* **`amazon_ads_mcp-invoke_tool`** — Invoke a specific tool by name with the arguments assembled from the described schema.

The agent follows a simple **search → describe → invoke** loop: it searches for the right operation, describes its schema, then invokes it. This keeps the model's context small and predictable regardless of how many operations Amazon Ads exposes, which is useful for smaller-context models, cost-sensitive workloads, or clients that only need a handful of operations per session.

>[NOTE] The Lite version is functionally equivalent to the Regular version — any operation you can perform with the Regular server can also be performed through the Lite server's three tools. The difference is only in how tools are surfaced to your model. If you want every Amazon Ads operation registered as an individual, directly-callable tool, use the [Regular version](mcp/get-started) instead.

## **Lite vs. Regular at a glance**

| Version | Regular | Lite |
|---|---|---|
| Endpoint path | `/mcp` | `/mcp/lite` |
| Tools exposed | One tool per Amazon Ads operation | Three tools: `search_tools`, `describe_tools`, `invoke_tool` |
| Context footprint | High initial context (all operational tools from MCP), grows with the total number of operations from MCP | Low initial context (3 tools), grows only with more tools discovered on-demand |
| Best for | Clients that want every operation visible and directly callable, or manage progressive discovery on client-side | Smaller-context models, cost-sensitive or focused sessions |

## **Account Identifier Context Modes**

The Lite version supports the same account context modes as the Regular version.

By default, the MCP server uses a **Dynamic Account Context** mode. In this mode, the LLM asks you to provide the account identifiers needed to invoke each selected tool. You can discover the account query tool through `amazon_ads_mcp-search_tools` and then invoke it to get all accounts the user has access to. Once provided to the LLM, this account may be reused for subsequent requests or until new account identifiers are provided to the LLM.

* **`profileId`**: The identifier of a profile associated with the advertiser account.
* **`managerAccountId`**: The manager account ID.
* **`advertiserAccountId`**: The identifier can be either a DSP advertiser account, global account ID, or AMC instance ID depending on the tool you are trying to call.

If you would instead like to support an account identifier context which behaves more closely to the Amazon Ads APIs, the server also supports a **Fixed Account Context** mode. In this mode, you can set your account identifiers as statically-defined headers. Doing so indicates to the server not to prompt your LLM for account information, and instead uses the pre-set headers for this information. Please note that for this mode, all headers should correspond with a single account.

## **Required Headers**

The Lite version uses the same headers as the Regular version. Each request requires the following headers:

* **`Amazon-Ads-ClientId`:** The client identifier of an LwA application authorized to access the MCP Server.
* **`Authorization`**: The string **`Bearer`** _prepended_ to an access token representing the permission of that application to access data and services for a given Amazon user.

**Fixed Account Context** requires the **additional headers** defined below:

*  **`Amazon-Ads-AI-Account-Selection-Mode`**: Indicates to the MCP server that the client wants to operate in fixed account scope mode. Currently the server only supports `Amazon-Ads-AI-Account-Selection-Mode: FIXED`. If you prefer dynamically-set account identifiers, do not include this header.
* **`Amazon-Advertising-API-Scope`**: The identifier of a profile associated with the advertiser account.
* **`Amazon-Ads-AccountID`**: The identifier can be either a DSP advertiser account, global account ID, or AMC instance ID depending on the tool you are trying to call.
* **`Amazon-Ads-Manager-AccountID`**: Alternatively to profile ID or account ID, you could pass in the manager account ID. Please note that not all tools work on manager account scope.

Please note that not all three account identifier headers need to be included, but at least one is required. The header + value `Amazon-Ads-AI-Account-Selection-Mode: FIXED` is always required when using this context mode.

## **MCP Server URL**

The Lite version is served from the `/mcp/lite` path. These URLs are not region agnostic. For connecting to the server in the `NA` region, please use the `NA` URL.

|Region|Endpoint URL|
|---|---|
|`NA` (North America)|https://advertising-ai.amazon.com/mcp/lite|
|`EU` (Europe)|https://advertising-ai-eu.amazon.com/mcp/lite|
|`FE` (Far East)|https://advertising-ai-fe.amazon.com/mcp/lite|

## Connecting to the Lite MCP server using Kiro CLI

**Step 1: Fetch the `clientId` of your application as well as `access token` and `refresh token`**

Sign in to https://developer.amazon.com/ with your Amazon **developer** account credentials, and then navigate to https://developer.amazon.com/loginwithamazon/console/site/lwa/overview.html to retrieve your **`clientId`**.

**Step 2: Connect to the Lite MCP server via Kiro CLI by adding the MCP server configuration to `~/.kiro/settings/mcp.json`.**

>[NOTE] Ensure that you fill in your `clientId` and `access token` in the `Amazon-Ads-ClientId` and `Authorization` headers. The only difference from the Regular configuration is the `/mcp/lite` URL path.

**Fixed Account Context:**

```json
{
  "mcpServers": {
    "amzn-ads-mcp-lite": {
      "url": "https://advertising-ai.amazon.com/mcp/lite",
      "headers": {
        "Authorization": "Bearer Atza|<your_access_token>",
        "Amazon-Ads-ClientId": "<your_client_id>",
        "Amazon-Ads-AI-Account-Selection-Mode": "FIXED",
        "Amazon-Advertising-API-Scope": "<your_profile_id>",
        "Accept": "application/json, text/event-stream"
      },
      "timeout": 60000,
      "disabled": false
    }
  }
}
```

**Dynamic Account Context:**

```json
{
  "mcpServers": {
    "amzn-ads-mcp-lite": {
      "url": "https://advertising-ai.amazon.com/mcp/lite",
      "headers": {
        "Authorization": "Bearer Atza|<your_access_token>",
        "Amazon-Ads-ClientId": "<your_client_id>",
        "Accept": "application/json, text/event-stream"
      },
      "timeout": 60000,
      "disabled": false
    }
  }
}
```

**Step 3: Run Kiro CLI**

Once your MCP configuration is properly set up:

1. Start Kiro CLI by running `kiro-cli`
2. Look for a tick mark (✓) next to your MCP server name during startup — this indicates a successful connection
3. Use the `/mcp` command to see the list of connected MCP servers
4. Use the `/tools` command to list available tools. For the Lite server you should see exactly three tools: `amazon_ads_mcp-search_tools`, `amazon_ads_mcp-describe_tools`, and `amazon_ads_mcp-invoke_tool`.


## **How the three tools work together**

When you make a request, the agent works through the three tools in sequence:

1. **Search** — The agent calls `amazon_ads_mcp-search_tools` with a short query describing what you want (for example, "keyword targets ad group"). The server returns the matching Amazon Ads tools and their descriptions.
2. **Describe** — The agent calls `amazon_ads_mcp-describe_tools` with the tool names it picked from the search results. The server returns each tool's full input schema so the agent knows which fields to fill in.
3. **Invoke** — The agent calls `amazon_ads_mcp-invoke_tool` with the chosen tool name and the assembled arguments to perform the operation.

More complex requests simply repeat the loop — for example, an eligibility check followed by a create will search/describe/invoke once for each underlying operation.

>[NOTE] **Fewer searches within a domain.** A `search_tools` call returns not only the direct matches but every tool in each matched domain (namespace), with names and descriptions. Because a session's requests tend to stay within the same domain — a user working in `campaign_management`, for example, will likely follow up with other `campaign_management` operations — the agent already holds those tools in context after the first search. Subsequent requests in the same domain can go straight to `describe_tools` and `invoke_tool` without issuing a new `search_tools` call, reducing round-trips and latency. In effect the first search for a domain acts like a cache warmed by *spatial locality*: fetching one tool brings its neighbors in the same domain along with it, so nearby follow-up operations are already available.

## **Example prompts**

The following examples show the kinds of natural-language requests the Lite server handles, along with the search → describe → invoke path the agent takes behind the scenes. You do not need to name the three tools yourself; the agent selects them automatically based on your request.

### Read a target list

```
Show me the keyword targets in ad group <ad_group_id>, with their state and bids.
```

Behind the scenes: `amazon_ads_mcp-search_tools` ("keyword targets ad group") → `amazon_ads_mcp-describe_tools` (query target schema) → `amazon_ads_mcp-invoke_tool` (query targets).

### Add a keyword target

```
Add the keyword 'wireless headphones' as a broad match target to ad group <ad_group_id> with a $1.25 bid.
```

Behind the scenes: `amazon_ads_mcp-search_tools` ("create keyword target") → `amazon_ads_mcp-describe_tools` (create target schema) → `amazon_ads_mcp-invoke_tool` (create target).

### Update a bid

```
Update the bid for target <target_id> to $1.00.
```

Behind the scenes: `amazon_ads_mcp-search_tools` ("update target bid") → `amazon_ads_mcp-describe_tools` (update bid schema) → `amazon_ads_mcp-invoke_tool` (update bid).

### Check product eligibility

```
Check whether ASIN <asin> is eligible to run Sponsored Products ads in the US.
```

Behind the scenes: `amazon_ads_mcp-search_tools` ("product eligibility") → `amazon_ads_mcp-describe_tools` (eligibility schema) → `amazon_ads_mcp-invoke_tool` (check eligibility).

### Generate a report

```
Generate a performance report for ad group <ad_group_id> with impressions, clicks and spend, then pull the results once it's ready.
```

Behind the scenes: the agent runs the search → describe → invoke loop once to create the report, then again to poll and download the results.

### List your accounts

```
Show me all the advertiser accounts I have access to.
```

Behind the scenes: `amazon_ads_mcp-search_tools` ("advertiser accounts") → `amazon_ads_mcp-invoke_tool` (query advertiser accounts).

>[TIP] These prompts are intentionally simple. In practice you can combine steps in a single request — for example, "check if there are existing keyword targets in ad group <ad_group_id>, then add 'hiking boots' as a broad match target." Both operations live in the same domain, so after the first search the agent already know the create-target tool exists in context and can describe and invoke it without searching again.

## Troubleshooting

```
Error: the client initialization failed
```
Most likely, this means that your LwA token has expired. Fetch a new one, update your `Authorization` header, and restart your client to create a new session.

If the `/tools` command shows more or fewer than three tools, confirm your `url` ends in `/mcp/lite` and not `/mcp`.

---
## Documentation Index & Resources

- **Full Markdown Index**: [README.md](https://raw.githubusercontent.com/linuxidefix/amazon-ads-doc/main/README.md) ([GitHub View](https://github.com/linuxidefix/amazon-ads-doc/blob/main/README.md))
- **Full HTML Index**: [index.html](https://raw.githubusercontent.com/linuxidefix/amazon-ads-doc/main/index.html) ([GitHub View](https://github.com/linuxidefix/amazon-ads-doc/blob/main/index.html))
- **OpenAPI Specifications (Tools)**: [Tools Directory](https://raw.githubusercontent.com/linuxidefix/amazon-ads-doc/main/README.md#2-tools--openapi-specifications-17-tools)
- **Skills Library**: [Skills Directory](https://raw.githubusercontent.com/linuxidefix/amazon-ads-doc/main/README.md#4-skills-library-12-skills)
