# Weav.com Model Context Protocol (MCP)

Learn how to use the [Model Context Protocol (MCP)](https://modelcontextprotocol.io/introduction) to enable AI agents to securely access and interact with your Weav workspace.

The Weav MCP server is available for Weav workspaces.

## What is Model Context Protocol?

MCP is a protocol that enables AI tools and applications to connect with Weav's data and services in a secure, standardized way. It provides a structured method for AI models to:

- Find and retrieve Weav data (conversations, customers, knowledge base)
- Access specific tools and functionality provided by Weav
- Maintain context about your Weav workspace when working with AI assistants

## How MCP Works

Weav hosts a remote MCP server that follows the authenticated remote MCP specification ([docs](https://modelcontextprotocol.io/specification/2025-03-26/basic/authorization)). This server handles requests from AI tools and provides access to Weav data through a secure interface.

**Connection URL:**

- **Streamable HTTP**: `https://mcp.weav.com/mcp`

When an AI tool or application needs to access Weav data:

1. The tool connects to Weav's MCP server
2. A workspace admin or owner completes OAuth consent in the browser
3. The tool can then access relevant Weav data and functionality for that workspace
4. The connection remains live until it is revoked

## Benefits of Using MCP

- **Secure Access**: All data access is authenticated and authorized. Only a workspace admin or owner can finish consent.
- **Standardized Interface**: Consistent interaction pattern across different AI tools
- **Contextual Understanding**: AI assistants maintain awareness of your Weav inbox, customers, and knowledge base
- **Increased Development Efficiency**: Triage conversations, reply to customers, and update your knowledge base from the AI tools you already use

## Available Tools

The Weav MCP server provides **11 tools** for interacting with your workspace.

### Conversations

#### **search_conversations**

Search support conversations the same way the Weav inbox does.

**Key Features:**

- Matches subject, customer name, or email
- Only returns conversations that have at least one message
- Spam is excluded unless `spam` is `true`
- Optional filters: `status` (`open`, `closed`), `channel` (`email`, `chat`), `limit` (1–50)

#### **get_conversation**

Get a conversation, including recent messages.

**Key Features:**

- Use the conversation UUID returned from `search_conversations`
- Returns conversation metadata and recent message history

#### **send_reply**

Post a reply to a conversation as a workspace teammate.

**Key Features:**

- Requires `conversation_id` and `content`
- Sends the reply in the connected workspace

#### **update_conversation**

Update a conversation's status, priority, or assignee. Does not delete conversations.

**Key Features:**

- `status`: `open` or `closed`
- `priority`: `low`, `normal`, `high`, or `urgent`
- `assignee_id` plus `assignee_type` (`user` or `agent`); omit to leave unchanged, send `null` to unassign

### Customers

#### **search_customers**

Search customers by name or email.

**Key Features:**

- Optional `query` and `limit` (1–50)

#### **get_customer**

Get a customer profile, company, and recent conversations.

**Key Features:**

- Use the customer UUID returned from `search_customers`

### Knowledge base

#### **search_training_data**

Search knowledge base (training data) items by title, description, type, or source.

**Key Features:**

- Optional `type` filter: `text`, `url`, `file`, `video`, `qa`
- Optional `limit` (1–50)

#### **get_training_data**

Get a single knowledge base item.

**Key Features:**

- Use the training data UUID returned from `search_training_data`

#### **add_training_data**

Add knowledge base content as text, Q&A, or a website URL. Every agent in the workspace is given access automatically.

**Key Features:**

- `type`: `text`, `qa`, or `url`
- `url` crawls the website and trains on discovered pages
- `urls` trains only the listed pages, without crawling the rest of the site

#### **delete_training_data**

Delete a knowledge base item.

**Key Features:**

- For website/URL sources, also deletes discovered URLs and chunks

#### **resync_training_data**

Re-scrape an existing URL knowledge base item.

**Key Features:**

- Requires the training data UUID of a URL source

## Setting things up

### Authentication

The MCP server uses **OAuth**. When you connect, Weav opens a browser consent screen.

- Only a workspace **admin** or **owner** can finish consent
- Tokens are scoped to that workspace with the `mcp:use` ability
- Admins can disconnect a client later in **Settings → Developers → MCP connections**

There is no API-token or bearer-token shortcut. Connect through OAuth.

### Configuration Examples

Configuration Guide
The examples below are generic templates. **Always refer to your specific LLM provider's official documentation** for the most up-to-date configuration instructions, as setup details may vary between versions and providers.

For **OAuth authentication**:

```json
{
  "mcpServers": {
    "weav": {
      "command": "npx",
      "args": [
        "mcp-remote",
        "https://mcp.weav.com/mcp"
      ]
    }
  }
}
```

## Required Scopes

The Weav MCP server issues tokens with the `mcp:use` scope. That scope is granted when an admin or owner completes consent for the workspace.

## Debugging and Troubleshooting MCP-Remote

1. **Authentication Problems**

```bash
# Kill existing connections
pkill -f mcp-remote

# Clear MCP auth cache
rm -rf ~/.mcp-auth
```

2. **Connection Testing**

```bash
# Test direct connection (starts the OAuth browser flow)
npx mcp-remote https://mcp.weav.com/mcp
```

3. **View Active MCP Connections**

```bash
ps aux | grep mcp-remote | grep -v grep
```

### Error Handling

- **Authentication failures**: Restart the OAuth flow. Confirm a workspace admin or owner completed consent.
- **401 Unauthorized**: The access token is missing, expired, or the connection was revoked in Settings.
- **Tool errors**: Confirm the IDs you pass belong to the connected workspace.

### Troubleshooting Tips

- Restart the AI agent after configuration changes
- Confirm the URL is `https://mcp.weav.com/mcp` (including `/mcp`)
- Check the browser for OAuth consent errors
- Disconnect and reconnect from **Settings → Developers → MCP connections** if a client is stuck
