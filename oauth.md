---
title: Connecting to the Ads MCP Server using OAuth 2.1
description: Set up OAuth 2.1 authorization for the Amazon Ads MCP Server with AI clients such as Kiro CLI, Claude Code, Claude Desktop, and ChatGPT
type: guide
interface: api
tags:
  - mcp
  - oauth
  - pkce
keywords:
  - authorization
  - oauth
  - pkce
  - mcp
---

# Connecting to the Ads MCP Server using OAuth 2.1

The Amazon Ads MCP Server supports OAuth 2.1. With OAuth 2.1, your MCP client handles token retrieval automatically through a browser-based login flow — you do not need to manage access tokens in your configuration file.

This guide walks you through setting up OAuth 2.1 authorization for Kiro CLI, Claude Code, Claude Desktop, and ChatGPT.

>[NOTE]For the standard token-based connection method (where you provide an access token in your configuration file), see [Connecting to the Amazon Ads MCP Server](mcp/get-started).

## Public vs private client

| Mode | What you provide | Token behavior |
|---|---|---|
| **Public client** | `Client ID` only | Access token expires after 1 hour. A browser popup prompts you to re-authenticate. |
| **Private client** | `Client ID` + `Client Secret` | Refresh token is stored securely in your system keychain (macOS) or credentials file. Tokens refresh automatically. |

## Prerequisites

- An active Amazon Developer account at [developer.amazon.com](https://developer.amazon.com)
- An approved application with access to the Amazon Ads API
- Your `Client ID` from your Login with Amazon (LwA) security profile
- An MCP-compatible AI client (Kiro CLI, Claude Code, Claude Desktop, or ChatGPT)

## Step 1: Configure your Login with Amazon security profile

Your LwA security profile must include the correct callback URL so that the MCP client can complete the authentication handshake.

1. Go to [developer.amazon.com](https://developer.amazon.com) and sign in.
2. Navigate to **Login with Amazon** under your application settings.
3. Open your **Security Profile** and go to the **Web Settings** tab.
4. Under **Allowed Return URLs**, add the callback URL for your MCP client (see below).
5. Save your changes.

### Callback URLs by client

| Client | Callback URL |
|---|---|
| Kiro CLI | `http://localhost:<PORT>/oauth/callback` — Set the port in your `mcp.json` config (see Step 2) |
| Claude Code | `http://localhost:<PORT>/callback` — Set the port via `--callback-port` or config (see Step 2) |
| Claude Desktop | `https://claude.ai/api/mcp/auth_callback` |
| ChatGPT | `https://chatgpt.com/connector/oauth/<APP_ID>` — `APP_ID` is provided by OpenAI in the MCP configuration panel |
| Amazon Quick | `https://quick.aws.com/sn/oauthcallback` |

>[WARNING]If you do not add the required callback URL to your developer account, the OAuth 2.1 flow will fail with a redirect URI mismatch error.

![Redirect URI error when callback URL is not configured](/_images/mcp/oauth/redirect_uri_error.png)

## Step 2: Add the MCP server to your client

### Kiro CLI

Update the Kiro CLI config file at `~/.kiro/settings/mcp.json`:

```json
{
  "mcpServers": {
    "amzn-ads-mcp": {
      "type": "http",
      "url": "https://advertising-ai.amazon.com/mcp",
      "oauth": {
        "redirectUri": "localhost:<your-callback-port>",
        "clientId": "<your-client-id>"
      }
    }
  }
}
```

Replace `<your-client-id>` with your LwA `Client ID` and `<your-callback-port>` with a fixed port number (for example, `8001`). Use this same port when configuring the Allowed Return URL in Step 1.

To authenticate, run `kiro-cli` then press Ctrl+Y to copy the authentication URL to the clipboard and then open it in your browser.

![Kiro CLI OAuth 2.1 authentication prompt](/_images/mcp/oauth/kiro_cli_oauth_prompt.png)

### Claude Code

You can configure the connection using the `claude mcp` command or by editing the config file directly.

>[NOTE]Private client mode is supported by `claude mcp` only.

**Public client:**

```bash
claude mcp add --transport http amzn-ads-mcp https://advertising-ai.amazon.com/mcp --client-id <your-client-id> --callback-port <your-callback-port>
```

**Private client:**

```bash
claude mcp add --transport http amzn-ads-mcp https://advertising-ai.amazon.com/mcp --client-id <your-client-id> --client-secret --callback-port <your-callback-port>
```

You will be prompted to enter your `Client Secret`.

**Alternatively, edit `~/.claude.json` directly (public client):**

```json
{
  "mcpServers": {
    "amzn-ads-mcp": {
      "type": "http",
      "url": "https://advertising-ai.amazon.com/mcp",
      "oauth": {
        "clientId": "<your-client-id>",
        "callbackPort": <your-callback-port>
      }
    }
  }
}
```

To authenticate, run `claude`, type `/mcp`, choose `amzn-ads-mcp`, and select **Authenticate**:

![Claude Code MCP authentication menu](/_images/mcp/oauth/claude_code_authenticate.png)

### Claude Desktop

1. Go to **Customize** > **Connectors**.
2. Click the plus sign and choose **Add custom connector**.
3. Enter the server name (e.g., Amazon Ads MCP) and the server URL (`https://advertising-ai.amazon.com/mcp`).
4. Specify your `Client ID` and `Client Secret` under Advanced settings.
5. Click **Add**.

![Claude Desktop Add custom connector dialog](/_images/mcp/oauth/claude_desktop_connector.png)


### ChatGPT

1. Click your profile and select **Settings**.
2. Select the **Apps** tab and click **Create app** next to **Advanced settings**.
3. Enter the server name (e.g., Amazon Ads MCP) and the server URL (`https://advertising-ai.amazon.com/mcp`).
4. Click **Advanced OAuth settings**.
5. In the **Client registration panel**, switch to "User-Defined OAuth client".
6. Add the Callback URL to the Allowed Return URLs of your developer account (see Step 1).
7. Enter your `Client ID` and `Client Secret` (optional).
8. Click **Create**.

![ChatGPT Add MCP server dialog](/_images/mcp/oauth/chat_gpt_connector.png)

## Step 3: Authenticate

After configuring your client, the OAuth 2.1 flow begins:

1. Your MCP client opens the LwA login page in your browser.
2. Sign in with your Amazon advertising account credentials.
3. Review the permissions and click **Allow**.

![LwA consent screen](/_images/mcp/oauth/lwa_allow_consent.png)

After granting access, you will see a confirmation page. You can close the browser window and return to your MCP client.

![Authentication successful confirmation](/_images/mcp/oauth/authentication_successful.png)

You are now connected. Try a prompt like:

```
Show me my advertising accounts
```

The MCP client will call the `account_management-query_advertiser_account` tool and return your account IDs, marketplace coverage, and account metadata.

## See also

- [Connecting to the Amazon Ads MCP Server](mcp/get-started)
- [Amazon Ads MCP Server overview](mcp/mcp-overview)
- [Login with Amazon documentation](https://developer.amazon.com/docs/login-with-amazon/documentation-overview.html)
- [MCP security authorization specification](https://modelcontextprotocol.io/docs/tutorials/security/authorization)

---
## Documentation Index & Resources

- **Full Markdown Index**: [README.md](https://raw.githubusercontent.com/linuxidefix/amazon-ads-doc/main/README.md) ([GitHub View](https://github.com/linuxidefix/amazon-ads-doc/blob/main/README.md))
- **Full HTML Index**: [index.html](https://raw.githubusercontent.com/linuxidefix/amazon-ads-doc/main/index.html) ([GitHub View](https://github.com/linuxidefix/amazon-ads-doc/blob/main/index.html))
- **OpenAPI Specifications (Tools)**: [Tools Directory](https://raw.githubusercontent.com/linuxidefix/amazon-ads-doc/main/README.md#2-tools--openapi-specifications-17-tools)
- **Skills Library**: [Skills Directory](https://raw.githubusercontent.com/linuxidefix/amazon-ads-doc/main/README.md#4-skills-library-12-skills)
