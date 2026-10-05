# Weav.com Model Context Protocol (MCP)

Learn how to use the [Model Context Protocol (MCP)](https://modelcontextprotocol.io/introduction) to enable AI agents to securely access and interact with your Weav workspace.

The Weav MCP server is available for Weav workspaces.

This repository documents the **workspace** server (`com.weav/mcp` at `https://mcp.weav.com/mcp`). A separate public server answers pricing and product questions without login:

| Server | URL | Auth | Registry |
|---|---|---|---|
| Weav (this repo) | `https://mcp.weav.com/mcp` | OAuth | `com.weav/mcp` |
| Weav Customer Service | `https://weav.com/mcp` | None | `io.github.dubdubdubco/weav-customer-service` |

The public server cannot read inbox, customers, training data, or Help Center data. Source: [dubdubdubco/weav-site-mcp](https://github.com/dubdubdubco/weav-site-mcp).

## What is Model Context Protocol?

MCP is a protocol that enables AI tools and applications to connect with Weav's data and services in a secure, standardized way. It provides a structured method for AI models to:

- Find and retrieve Weav data (conversations, customers, agent training data, Help Center articles)
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
- **Contextual Understanding**: AI assistants maintain awareness of your Weav inbox, customers, agent training data, and Help Center
- **Increased Development Efficiency**: Triage conversations, reply to customers, and manage agent training data and Help Center articles from the AI tools you already use

## Available Tools

The Weav MCP server provides **32 tools** for interacting with your workspace: 4 conversation tools, 2 customer tools, 5 agent training data tools, and 21 Help Center tools.

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

### Agent training data

These are the sources Weav AI agents answer from, separate from the public Help Center managed by the `kb_*` tools.

#### **search_training_data**

Search agent training data items by title, description, type, or source.

**Key Features:**

- Optional `type` filter: `text`, `url`, `file`, `video`, `qa`
- Optional `limit` (1–50)

#### **get_training_data**

Get a single training data item.

**Key Features:**

- Use the training data UUID returned from `search_training_data`

#### **add_training_data**

Add agent training data as text, Q&A, or a website URL. Every agent in the workspace is given access automatically.

**Key Features:**

- `type`: `text`, `qa`, or `url`
- `url` crawls the website and trains on discovered pages
- `urls` trains only the listed pages, without crawling the rest of the site

#### **delete_training_data**

Delete a training data item.

**Key Features:**

- For website/URL sources, also deletes discovered URLs and chunks

#### **resync_training_data**

Re-scrape an existing URL training data item.

**Key Features:**

- Requires the training data UUID of a URL source

### Help Center knowledge base

The `kb_*` tools manage the workspace's public Help Center articles and categories, separate from agent training data managed by `*_training_data`.

**Browse**

#### **kb_get_overview**

Get counts and available locales for the Help Center.

**Key Features:**

- No parameters

#### **kb_list_categories**

List Help Center categories.

**Key Features:**

- Optional `limit` (1–50, default 20) and `page` (default 1)

#### **kb_list_articles**

List Help Center article summaries.

**Key Features:**

- Optional filters: `category_id` or `uncategorized`, `status` (`draft`, `published`), and `locale`
- Optional `limit` (1–50, default 20) and `page` (default 1)

#### **kb_search_articles**

Search Help Center article titles and content.

**Key Features:**

- Required `query` (1–255 characters)
- Optional filters: `locale`, `status` (`draft`, `published`), `category_id` or `uncategorized`
- Optional `limit` (1–50, default 20) and `page` (default 1)

#### **kb_get_article**

Get a full Help Center article and its translation content.

**Key Features:**

- Required `article_id`; optional `locale`

**Revisions**

#### **kb_list_article_revisions**

List revision summaries for an article.

**Key Features:**

- Required `article_id`; optional `locale`, `limit` (1–50, default 20), and `page` (default 1)

#### **kb_get_article_revision**

Get a revision's full content.

**Key Features:**

- Required `article_id` and `revision_id`

#### **kb_restore_article_revision**

Restore an article translation from a prior revision.

**Key Features:**

- Required `article_id` and `revision_id`; optional `expected_version`
- Saves the current translation as a revision before restoring

**Improvement proposals (read-only)**

#### **kb_list_proposals**

List Help Center improvement proposals.

**Key Features:**

- Optional `status` (`pending`, `needs_decision`, `published`, `dismissed`, `ignored`); ignored proposals are omitted by default
- Optional `limit` (1–50, default 20) and `page` (default 1)

#### **kb_get_proposal**

Get full details for an improvement proposal.

**Key Features:**

- Required `proposal_id`

**Articles**

#### **kb_create_article**

Create a Help Center article.

**Key Features:**

- Requires at least one `translations` entry with `locale`, `title`, and Markdown `content`; `excerpt` and `slug` are optional
- `excerpt` is also used as the meta description
- Translation entries optionally accept `seo_title` (search/social title; the page heading stays `title`), `noindex` (hide from search engines), `canonical_url` (absolute HTTP(S) URL), and `og_image_url` (HTTPS social preview image)
- Optional `category_id`, `position`, and `status` (`draft` or `published`)

#### **kb_update_article**

Update a Help Center article.

**Key Features:**

- Required `article_id`; optional `expected_version`, `category_id`, `position`, `status`, and `translations`
- Translation entries accept the same optional SEO fields as `kb_create_article`; `excerpt` is also used as the meta description
- Send `category_id: null` to uncategorize; omit it to keep the current category
- Content edits to an existing locale save a revision; SEO-only edits do not. A new locale needs `title` and `content`

#### **kb_publish_article**

Publish an article immediately.

**Key Features:**

- Required `article_id`; optional `expected_version`

#### **kb_unpublish_article**

Return a published article to draft.

**Key Features:**

- Required `article_id`; optional `expected_version`

#### **kb_delete_article**

Soft-delete a Help Center article.

**Key Features:**

- Required `article_id`; optional `expected_version`

#### **kb_delete_article_translation**

Delete one translation from an article.

**Key Features:**

- Required `article_id` and `locale`; optional `expected_version`
- The article must retain at least one translation

#### **kb_reorder_articles**

Set the order of all articles in one category group.

**Key Features:**

- Required `ids` containing all article UUIDs in the desired order
- Optional `category_id`; omit it or send `null` for uncategorized articles

**Categories**

#### **kb_create_category**

Create a Help Center category.

**Key Features:**

- Required `name`; optional `slug` and `position`

#### **kb_update_category**

Update a Help Center category.

**Key Features:**

- Required `category_id`; optional `name`, `slug`, and `position`

#### **kb_delete_category**

Delete a Help Center category.

**Key Features:**

- Required `category_id`; its articles become uncategorized

#### **kb_reorder_categories**

Set the display order of all Help Center categories.

**Key Features:**

- Required `ids` containing all category UUIDs in the desired order

#### Help Center behavior

- New articles are drafts by default. Set `status: published` on create or update, or use `kb_publish_article`, to publish immediately; there is no approval step.
- Search first with `kb_search_articles`; update an existing article instead of creating a duplicate.
- `kb_get_article` returns a `version`. Pass it as `expected_version` to update, publish, unpublish, delete, delete a translation, or restore. A stale version returns `PRECONDITION_FAILED` with `current_version`; re-read and retry. If the article lock cannot be acquired within its wait period, the tool returns `ARTICLE_LOCKED`; retry shortly. Other error codes include `SLUG_TAKEN`.
- Locales match by language: `en`, `en-US`, and `en-GB` all match English content.
- Articles created over MCP have source `mcp`; each revision records the user and MCP client that made it.
- Help Center settings, logo and image uploads, GitHub sync, and publishing or dismissing improvement proposals are not available over MCP; use the Weav app.

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
- **`PRECONDITION_FAILED`**: The article changed since its `expected_version`; re-read it and retry with `current_version`.
- **`ARTICLE_LOCKED`**: The article lock wait timed out; retry shortly.
- **Tool errors**: Confirm the IDs you pass belong to the connected workspace.

### Troubleshooting Tips

- Restart the AI agent after configuration changes
- Confirm the URL is `https://mcp.weav.com/mcp` (including `/mcp`)
- Check the browser for OAuth consent errors
- Disconnect and reconnect from **Settings → Developers → MCP connections** if a client is stuck
