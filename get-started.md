---
title: Connecting to the Amazon Ads MCP Server
description: Learn how to connect to and use the Amazon Ads Model Context Protocol (MCP) Server for agentic access to Amazon Ads
type: guide
interface: api
tags:
  - mcp
---

# Connecting to the Amazon Ads MCP Server

## **Getting Started:**

**Step 1:** Use an existing LwA application and Amazon Developer account with access to the Amazon Ads API or [complete the onboarding steps](https://advertising.amazon.com/API/docs/en-us/guides/onboarding/overview)

**Step 2:** [Complete an authorization grant](https://advertising.amazon.com/API/docs/en-us/guides/get-started/overview) and retrieve **`access`** and **`refresh`** tokens. Save the access and refresh tokens — you will need them to connect to the MCP server later.

>[TIP:OAuth 2.1 authorization]If you prefer not to manage access tokens manually, you can use [OAuth 2.1 authorization](mcp/oauth) instead. OAuth 2.1 lets your MCP client handle token retrieval automatically through a browser-based login flow — no tokens in your config file required.

## **Account Identifier Context Modes**
While the Amazon Ads API requires you to input specific `accountIds` in headers or fields, like inserting a `profileId` into the `Amazon-Advertising-API-Scope` header, the MCP server handles this differently.

By default, the MCP server uses a **Dynamic Account Context** mode. In this mode, the LLM asks you to provide the account identifiers needed to invoke each selected tool. You can use the `query_advertiser_accounts` tool to get all accounts the user has access to. Once provided to the LLM, this account may be reused for subsequent requests or until new account identifiers are provided to the LLM. The MCP server requires the account identifiers to be passed as a parameter in the request body.

* **`profileId`**: The identifier of a profile associated with the advertiser account.

* **`managerAccountId`**: The manager account ID.

* **`advertiserAccountId`**: The identifier can be either a DSP advertiser account, global account ID, or AMC instance ID depending on the tool you are trying to call.

If you would instead like to support an account identifier context which behaves more closely to the Amazon Ads APIs, the server also supports a **Fixed Account Context** mode. In this mode, you can set your account identifiers as statically-defined headers. Doing so indicates to the server not to prompt your LLM for account information, and instead uses the pre-set headers for this information. Please note that for this mode, all headers should correspond with a single account.

## **Required Headers**

To connect to the Amazon Ads MCP server, each request to the Amazon Ads MCP server requires the following headers:

* **`Amazon-Ads-ClientId`:** The client identifier of an LwA application authorized to access the MCP Server.
* **`Authorization`**: The string **`Bearer`** _prepended_ to an access token representing the permission of that application to access data and services for a given Amazon user.

**Fixed Account Context** requires the **additional headers** defined below: profile ID header, account ID header, or manager account ID header, and a header to indicate fixed mode. Dynamic Account Context only needs the two basic headers defined above (`Amazon-Ads-ClientId` and `Authorization`).

*  **`Amazon-Ads-AI-Account-Selection-Mode`**: The header that indicates to the MCP server that the client wants to operate in fixed account scope mode.
    * Currently, the server only supports `Amazon-Ads-AI-Account-Selection-Mode: FIXED`
    * If you prefer to use dynamically-set account identifiers, do not include this header
* **`Amazon-Advertising-API-Scope`**: The identifier of a profile associated with the advertiser account.
* **`Amazon-Ads-AccountID`**: The identifier can be either a DSP advertiser account, global account ID, or AMC instance ID depending on the tool you are trying to call.
* **`Amazon-Ads-Manager-AccountID`**: Alternatively to profile ID or account ID, you could pass in the manager account ID. Please note that not all tools work on manager account scope.

Please note that not all three account identifier headers (`Amazon-Advertising-API-Scope`, `Amazon-Ads-AccountID`, `Amazon-Ads-Manager-AccountID`) need to be included, but at least one is required. The header + value `Amazon-Ads-AI-Account-Selection-Mode: FIXED` is always required when using this context mode.

## **MCP Server URL**

These URLs are not region agnostic. For connecting to the server in the `NA` region, please use the `NA` URL.

|Region|Endpoint URL|
|---|---|
|`NA` (North America)|https://advertising-ai.amazon.com/mcp|
|`EU` (Europe)|https://advertising-ai-eu.amazon.com/mcp|
|`FE` (Far East)|https://advertising-ai-fe.amazon.com/mcp|

## Connecting to the MCP server using Kiro CLI

**Step 1: Fetch the `clientId` of your application as well as `access token` and `refresh token`**

Sign in to https://developer.amazon.com/ with your Amazon **developer** account credentials, and then navigate to https://developer.amazon.com/loginwithamazon/console/site/lwa/overview.html. You must be able to retrieve your **`clientId`** from that page as shown below:

![client_id and client_secret.png](/_images/mcp/client_id_and_client_secret.png)

**Step 2: Connect to the MCP server via Kiro CLI by adding the MCP server configuration to `~/.kiro/settings/mcp.json`.**

>[NOTE] Ensure that you fill in your `clientId` and `access token` in the `Amazon-Ads-ClientId` and `Authorization` headers.

**Fixed Account Context:**

```json
{
  "mcpServers": {
    "amzn-ads-mcp": {
      "url": "https://advertising-ai.amazon.com/mcp",
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

>[NOTE] `Amazon-Ads-AccountID` or `Amazon-Ads-Manager-AccountID` can also be used above in addition to or instead of `Amazon-Advertising-API-Scope`.

**Dynamic Account Context:**

```json
{
  "mcpServers": {
    "amzn-ads-mcp": {
      "url": "https://advertising-ai.amazon.com/mcp",
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

**Step 3 (optional): Set up tool filters through Kiro configuration**

If you would like to disable any of the available tools, you can do so through your Kiro configuration by adding `disabledTools`. The following example disables the `delete_campaign` and `delete_ad` tools.

```json
{
  "mcpServers": {
    "amzn-ads-mcp": {
      "url": "https://advertising-ai.amazon.com/mcp",
      "headers": {
        "Authorization": "Bearer Atza|<your_access_token>",
        "Amazon-Ads-ClientId": "<your_client_id>",
        "Accept": "application/json, text/event-stream"
      },
      "timeout": 60000,
      "disabled": false,
      "disabledTools": ["campaign_management-delete_campaign", "campaign_management-delete_ad"]
    }
  }
}
```

**Step 4: Run Kiro CLI**

Once your MCP configuration is properly set up:

1. Start Kiro CLI by running `kiro-cli`
2. Look for a tick mark (✓) next to your MCP server name during startup — this indicates a successful connection
3. To verify the connection, use the `/mcp` command to see the list of connected MCP servers
4. To see available tools, use the `/tools` command to list available tools.

This confirms your Amazon Ads MCP server is properly connected and ready to use.

![Kiro CLI Startup](/_images/mcp/kiro_cli_startup.png)
![MCP Tools](/_images/mcp/mcp_tools.png)

**Step 5: Utilize the Amazon Ads MCP Server**

In the **fixed account context**, you can start directly by creating campaigns or performing other operations.
Example of next steps:

```
Create an SP campaign
```
![Create SP Campaign Fixed 1](/_images/mcp/create_sp_campaign_fixed-1.png)
![Create SP Campaign Fixed 2](/_images/mcp/create_sp_campaign_fixed-2.png)

The Amazon Ads MCP server uses the header provided from the **fixed account context** to create the campaign.

In the **dynamic account context**, before creating campaigns or performing other operations, you must first identify your available advertiser accounts.

>[NOTE] For clients with multiple marketplaces: First select your marketplace, then select your profile. Following this sequence ensures operations execute correctly.

1. Ask Kiro to list your accounts with this prompt:
   ```
   Can you give me the advertiser accounts that I have access to?
   ```
2. This triggers the `account_management-query_advertiser_account` tool and returns a list of all advertiser accounts which can be accessed with the current context.
3. Note the account IDs from the results — you will **need** to specify these in subsequent commands.

Example of next steps:

```
Create an SP campaign using the profile id for amzn1.ads-account.g.id
```

The account ID retrieved becomes the profile/account identifier for all future campaign management operations.
![Query Accounts](/_images/mcp/query_advertiser_accounts.png)
![Create SP Campaign Dynamic 1](/_images/mcp/create_sp_campaign_dynamic-1.png)
![Create SP Campaign Dynamic 2](/_images/mcp/create_sp_campaign_dynamic-2.png)

---

## Sample Python Client Library and Conversational Agent
If you are interested in connecting to the Amazon Ads MCP Server programmatically through Python, a sample client library implementation and a working demo are provided that you can copy and customize for your needs. The library includes a fluent builder interface for easy configuration, support for all regions (`NA`, `EU`, `FE`), optional tool filtering, and automatic handling of statically-defined account identifier headers if preferred.

**Prerequisites:**
You will need to install the following dependencies which are used for initializing your client. You can follow the installation steps for each dependency here:
1. [Strands Python API](https://strandsagents.com/latest/documentation/docs/)
2. [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk/tree/main?tab=readme-ov-file#installation)

Next, if you are running this on your local machine, you will need to configure AWS so that your local execution has access to your AWS credentials. This is not needed to connect to the server, but it is necessary to connect your agent to an LLM.

For more on foundational model access with AWS, see here: https://docs.aws.amazon.com/bedrock/latest/userguide/model-access.html

First, [install the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html) if it is not already installed. After that, you can run `aws configure` to connect your local environment to an AWS account that has access to your desired model. See [here](https://docs.aws.amazon.com/cli/latest/reference/configure/) for more on using `aws configure`.

Finally, fetch the `clientId` of your application, `access token`, and `refresh token`.
Sign in to https://developer.amazon.com/ with your Amazon developer account credentials, and then navigate to https://developer.amazon.com/loginwithamazon/console/site/lwa/overview.htm to retrieve the required ID and tokens.

**Sample Client + Conversational Playground**

Create a Python file in your local environment and paste in the following code snippet:

```python
from mcp.client.streamable_http import streamablehttp_client
from strands.tools.mcp import MCPClient
import sys
import io
from enum import Enum


class Region(Enum):
    NA = "na"
    EU = "eu"
    FE = "fe"


def _derive_server_url(region: str) -> str:
    if region == "na":
        return "https://advertising-ai.amazon.com/mcp"
    elif region == "eu":
        return "https://advertising-ai-eu.amazon.com/mcp"
    elif region == "fe":
        return "https://advertising-ai-fe.amazon.com/mcp"
    raise ValueError(f"Unsupported region: {region}")


class AmazonAdsMCPClient:
    def __init__(self, mcp_client: MCPClient):
        self._client = mcp_client

    def __enter__(self):
        self._client.__enter__()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        old_stderr = sys.stderr
        sys.stderr = io.StringIO()
        try:
            self._client.__exit__(exc_type, exc_val, exc_tb)
        except Exception:
            pass
        finally:
            sys.stderr = old_stderr

    def get_tools(self, disabled_tools=None):
        """
        Get available tools from the MCP server, excluding disabled tools.

        Args:
            disabled_tools: List of tool names to exclude

        Returns:
            List of available MCP tools
        """
        all_tools = self._client.list_tools_sync()
        if not disabled_tools:
            return all_tools

        disabled_set = set(disabled_tools)
        return [tool for tool in all_tools if tool.tool_name not in disabled_set]


class MCPClientBuilder:
    def __init__(self):
        self._region = None
        self._client_id = None
        self._auth_token = None
        self._account_id = None
        self._profile_id = None
        self._manager_account_id = None

    def region(self, region: Region):
        self._region = region.value
        return self

    def client_id(self, client_id: str):
        self._client_id = client_id
        return self

    def auth_token(self, auth_token: str):
        self._auth_token = auth_token
        return self

    def account_id(self, account_id: str):
        self._account_id = account_id
        return self

    def profile_id(self, profile_id: str):
        self._profile_id = profile_id
        return self

    def manager_account_id(self, manager_account_id: str):
        self._manager_account_id = manager_account_id
        return self

    def build(self) -> AmazonAdsMCPClient:
        missing = []
        if not self._client_id:
            missing.append("client_id")
        if not self._auth_token:
            missing.append("auth_token")
        if missing:
            raise ValueError(f"Missing required parameters: {', '.join(missing)}")

        url = _derive_server_url(self._region)
        headers = {
            "Amazon-Ads-ClientId": self._client_id,
            "Authorization": self._auth_token
        }
        # Populate fixed account headers if present
        has_fixed_account = False
        if self._account_id:
            headers["Amazon-Ads-AccountId"] = self._account_id
            has_fixed_account = True
        if self._profile_id:
            headers["Amazon-Advertising-Api-Scope"] = self._profile_id
            has_fixed_account = True
        if self._manager_account_id:
            headers["Amazon-Ads-Manager-AccountId"] = self._manager_account_id
            has_fixed_account = True

        if has_fixed_account:
            headers["Amazon-Ads-AI-Account-Selection-Mode"] = "FIXED"

        mcp_client = MCPClient(lambda: streamablehttp_client(url=url, headers=headers))
        return AmazonAdsMCPClient(mcp_client)

# Sample Playground Client using Strands
if __name__ == "__main__":
    from strands import Agent

    try:
        client = MCPClientBuilder() \
            .region(Region.NA) \
            .client_id("<your_client_id_here>") \
            .auth_token("Bearer Atza|<your_lwa_token_here>") \
            .build()

        with client:
            tools = client.get_tools()
            agent = Agent(tools=tools, model="us.anthropic.claude-3-5-sonnet-20240620-v1:0")
            print(f"✓ Agent created with {len(tools)} tools")

            print("\nStarting conversation loop. Type 'quit' to exit.")
            while True:
                try:
                    user_input = input("\n\033[94mYou: \033[0m").strip()
                    if user_input.lower() in ['quit', 'exit', 'q']:
                        break
                    if not user_input:
                        continue
                    print("\033[92m\nAgent:\n\033[0m")
                    agent(user_input)
                    print("\n")

                except KeyboardInterrupt:
                    print("\nExiting...")
                    break
                except Exception as e:
                    print(f"Error: {e}")

        sys.exit(0)

    except Exception as e:
        print(f"✗ Error: {e}")
        sys.exit(1)
```

Now, you can start the conversational agent by running your Python script.

You can edit this snippet or use it as a reference point to create your own Ads agent.

>[WARNING] This snippet is for demo purposes only. Use caution when adapting it for production applications.

If you prefer to use statically-defined account identifiers (Fixed Context Mode), you can add them to your client builder:

```python
client = MCPClientBuilder() \
            .region(Region.NA) \
            .client_id("<your_client_id_here>") \
            .auth_token("Bearer Atza|<your_lwa_token_here>") \
            .profile_id("your_profile_id") #Corresponds to Amazon-Advertising-Api-Scope
            .build()
```

The following account identifier headers are supported. Please only set what is appropriate for your use case:

```python
.profile_id("your_profile_id") #Corresponds to Amazon-Advertising-Api-Scope
.account_id("your_account_id") #Corresponds to Amazon-Ads-AccountId
.manager_account_id("your_manager_account_id") #Corresponds to Amazon-Ads-Manager-AccountId
```

If you prefer using dynamically-set account identifiers provided by your LLM, do not include any of these three values when initializing your client.

**Troubleshooting:**

```
Error: the client initialization failed
```
Most likely, this means that your LwA token has expired. Follow the prerequisite steps above to fetch a new one, update your client initialization with the fresh header, and restart the code snippet execution to create a new session.

```
Error: An error occurred (ExpiredTokenException) when calling the ConverseStream operation: The security token included in the request is expired
```
This indicates that your application does not have access to your AWS credentials, or your credential access has expired. Typically, this error occurs when you try prompting the agent, but your agent does not have access to your selected LLM. Follow the prerequisite steps and run `aws configure` to automatically grant your local environment access to your AWS account.
