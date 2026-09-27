---
title: Download Skills from the Amazon Ads MCP Server
description: Learn how to download and use skills from the Amazon Ads MCP Server
type: guide
interface: api
tags:
  - mcp
---
## Skills

The Amazon Ads MCP Server provides downloadable skill definitions that you can use to enhance your AI client capabilities. Skills are pre-defined sets of instructions and configurations that guide an AI agent through specific advertising workflows.

### Skill Server URLs

These URLs are not region agnostic. Select the endpoint URL that corresponds to your region.

|Region|Endpoint URL|
|---|---|
|`NA` (North America)|https://advertising-api.amazon.com/skills|
|`EU` (Europe)|https://advertising-api-eu.amazon.com/skills|
|`FE` (Far East)|https://advertising-api-fe.amazon.com/skills|

### Listing Available Skills

To list all available skills, send a `GET` request to the `/skills` endpoint. The request requires the same `Amazon-Ads-ClientId` and `Authorization` headers as the MCP server.

The response is a JSON object containing a `skills` array, where each item includes a skill name and description:

```json
{
  "skills": [
    {
      "name": "example-skill-name",
      "description": "A description of what this skill does."
    },
    {
      "name": "another-skill-name",
      "description": "A description of another skill."
    }
  ]
}
```

### Downloading Skills as ZIP

To download a skill as a ZIP archive, add the `download=true` query parameter:

```http
GET /skills?name=example-skill-name&download=true
```

This returns a ZIP file (for example, `example-skill-name.zip`) containing the files of the skill.

To download all skills as a single ZIP archive:

```http
GET /skills?name=all&download=true
```

This returns `all-skills.zip` with each skill organized in its own folder.

Use `wget` to save the file:

```bash
wget --content-disposition \
     --header="Amazon-Ads-ClientId: YOUR_CLIENT_ID" \
     --header="Authorization: Bearer YOUR_ACCESS_TOKEN" \
     "https://advertising-api.amazon.com/skills?name=example-skill-name&download=true"
```

### Retrieving Skill Files as JSON

To retrieve the files for a specific skill as JSON, append the `name` query parameter with the skill name to the `/skills` endpoint:

```http
GET /skills?name=example-skill-name
```

To download files for all available skills at once, use `name=all`:

```http
GET /skills?name=all
```

The response is a JSON object containing a `files` array. Each item includes the filename, path, and content of a skill file:

```json
{
  "files": [
    {
      "filename": "skill.md",
      "path": "example-skill-name/skill.md",
      "content": "# Skill instructions\n..."
    },
    {
      "filename": "input-schema.json",
      "path": "example-skill-name/input-schema.json",
      "content": "{ \"type\": \"object\", ... }"
    }
  ]
}
```

### Loading Skills into Your AI Client

Once you have downloaded a skill definition, you can load it into your AI client. 

#### Claude Code

**Step 1:** Download all available skills as a ZIP archive and extract them into your skills directory:

```bash
wget --header="Amazon-Ads-ClientId: YOUR_CLIENT_ID" \
     --header="Authorization: Bearer YOUR_ACCESS_TOKEN" \
     "https://advertising-api.amazon.com/skills?name=all&download=true" \
     -O all-skills.zip
unzip all-skills.zip -d .claude/skills/
```

**Step 2:** In Claude Code, skills are loaded from the `.claude/skills/` directory. Once the skill files are saved to this location, Claude Code will automatically detect and make them available for use.

**Step 3:** You can then invoke the skill within Claude Code by referencing it in your prompt, and the agent will follow the instructions defined in the skill.

#### Kiro

Same as the above steps, but put skills under `.kiro/skills/`.
