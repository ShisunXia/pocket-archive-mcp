<p align="center">
  <a href="https://www.pocketarchive.io"><img src="assets/logo.png" alt="Pocket Archive" width="96" height="96"></a>
</p>

# Pocket Archive MCP Server

Connect Claude, ChatGPT, Cursor, VS Code, Codex or any MCP app to your [Pocket Archive](https://www.pocketarchive.io), the AI document organizer for receipts, bills, contracts, warranties and tax documents. Your agent can search your documents, read their text, upload new files, track deadlines and organize records into Smart Collections, with read-only or read-and-write access you approve.

[![Install in Cursor](https://cursor.com/deeplink/mcp-install-dark.svg)](https://cursor.com/en/install-mcp?name=pocket-archive&config=eyJ1cmwiOiJodHRwczovL2FwaS5wb2NrZXRhcmNoaXZlLmlvL21jcCJ9)
[![Install in VS Code](https://img.shields.io/badge/VS_Code-Install_Server-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=pocket-archive&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fapi.pocketarchive.io%2Fmcp%22%7D)

**Server URL:** `https://api.pocketarchive.io/mcp` · Streamable HTTP · OAuth sign-in · [Setup guide on pocketarchive.io](https://www.pocketarchive.io/ai-agents)

This is a hosted (remote) MCP server; there is nothing to install or run locally. This repository contains its connection details, a reusable Skill and optional client adapters.

## Before you start

You need a Pocket Archive account. [Create one for free](https://www.pocketarchive.io/sign-up). MCP access is included on every plan, including Free; uploads still count toward your plan's text recognition and storage limits.

## Set up your app

Every app uses the same server URL: `https://api.pocketarchive.io/mcp`. The first time your app connects, it opens Pocket Archive in your browser. Check the app name, then choose read-only or allow changes.

### Claude Code

```sh
claude mcp add --transport http pocket-archive https://api.pocketarchive.io/mcp
```

Then run `/mcp` in Claude Code, choose `pocket-archive`, and sign in.

### Claude (web and desktop)

Open **Settings → Connectors** and choose **Add custom connector**. Name it Pocket Archive, paste the server URL, and select **Connect**.

### Codex

```sh
codex mcp add pocket-archive --url https://api.pocketarchive.io/mcp
codex mcp login pocket-archive
```

### Cursor

Add Pocket Archive to `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "pocket-archive": {
      "url": "https://api.pocketarchive.io/mcp"
    }
  }
}
```

Cursor opens your browser to sign in the first time it connects.

### VS Code

Add Pocket Archive to `.vscode/mcp.json` in your workspace, or run **MCP: Open User Configuration** from the Command Palette to add it for all workspaces:

```json
{
  "servers": {
    "pocket-archive": {
      "type": "http",
      "url": "https://api.pocketarchive.io/mcp"
    }
  }
}
```

Start the server from the file or the Extensions view; VS Code opens your browser to sign in.

### ChatGPT

Turn on developer mode in **Settings → Apps & Connectors → Advanced settings**. Create a connector with the server URL and choose OAuth sign-in. Developer mode is a beta feature, and availability depends on your ChatGPT plan.

## Connection details

For other MCP apps, or to check compatibility:

| Setting | Value |
| --- | --- |
| Server name | `pocket-archive` |
| Endpoint | `https://api.pocketarchive.io/mcp` |
| Transport | Streamable HTTP |
| Authentication | Browser OAuth authorization code with PKCE S256 |
| Registration | Dynamic public-client registration (DCR), `token_endpoint_auth_method=none` |
| Resource | `https://api.pocketarchive.io/mcp` |
| Scopes | `archive:read`, optionally `archive:write` |

Use the client's connect/login action. It discovers OAuth metadata, registers its own name and callback URL, opens browser authorization, and exchanges the approved code for MCP credentials. The authorization page displays the registered client's name and callback host; each approved connection is granted and revoked independently.

If you are already signed in to Pocket Archive on the web in the same browser, the authorization page reuses that sign-in. Otherwise you sign in and return to the authorization request. The agent receives its own MCP tokens; it never sees your web session. Read-only connections expose read tools only. **Settings → Connected apps** in Pocket Archive lists and revokes connections.

A client needs remote Streamable HTTP and this OAuth flow to connect directly. A client supporting only stdio, legacy HTTP+SSE, confidential-client authentication or Client ID Metadata Documents needs an adapter. Not every agent product has been tested; compatibility is defined by these capabilities, not by product names.

## Tools

Read-only connections see only the read tools. Allowing changes adds the write tools.

### Documents

| Tool | Access | What it does |
| --- | --- | --- |
| `search_documents` | Read | Search by keyword with document type, tag, date, vendor and amount filters; returns snippets and citations |
| `list_documents_page` | Read | List documents with filters, sorting and pagination |
| `list_documents` | Read | List documents as a single array (legacy) |
| `get_document` | Read | Get a document's fields, OCR text and processing state |
| `list_document_pages` | Read | Read OCR text page by page, with page numbers for citations |
| `download_original_document` | Read | Get short-lived download links for the original files |
| `get_document_upload_status` | Read | Check an upload before and after it is finalized |
| `create_upload_session` | Write | Reserve a signed upload URL for a PDF, JPEG, PNG or HEIC file up to 50 MiB |
| `create_document` | Write | Finalize an upload and start OCR, analysis and indexing |
| `update_document` | Write | Update a document's metadata or extracted fields |
| `delete_document` | Write | Move a document to trash |
| `restore_document` | Write | Restore a document from trash |
| `retry_document_processing` | Write | Retry a failed processing stage |

### Attention items (deadlines, renewals and follow-ups)

| Tool | Access | What it does |
| --- | --- | --- |
| `list_attention_items` | Read | List Attention items (the Action Inbox) across the archive or for one document |
| `get_attention_item` | Read | Get an Attention item and its related information |
| `create_attention_item` | Write | Create an Attention item, optionally linked to a document |
| `update_attention_item` | Write | Update an item's details, dates or status |
| `delete_attention_item` | Write | Delete an Attention item and its reminders |

### Smart Collections

| Tool | Access | What it does |
| --- | --- | --- |
| `list_collection_templates` | Read | List collection templates and their setup questions |
| `list_collections` | Read | List collections |
| `get_collection` | Read | Get a collection, its readiness and details |
| `list_collection_documents` | Read | List a collection's documents and suggestions |
| `list_collection_requirements` | Read | List a collection's checklist |
| `list_collection_requirement_documents` | Read | List the documents matched to a checklist item |
| `list_collection_attention_items` | Read | List Attention items for a collection |
| `create_collection` | Write | Create a collection from a template |
| `update_collection` | Write | Update, archive or restore a collection |
| `delete_collection` | Write | Delete a collection (its documents are kept) |
| `refresh_collection` | Write | Re-evaluate a collection and its suggestions |
| `include_collection_document` | Write | Add a document to a collection |
| `decide_collection_document` | Write | Accept or exclude a suggested document |
| `remove_collection_document` | Write | Remove a document from a collection |
| `create_collection_requirement` | Write | Add a checklist item |
| `update_collection_requirement` | Write | Update or complete a checklist item |
| `delete_collection_requirement` | Write | Delete a checklist item |
| `confirm_collection_requirement_document` | Write | Confirm that a document satisfies a checklist item |
| `decide_collection_requirement_document` | Write | Accept or reject a suggested match for a checklist item |
| `remove_collection_requirement_document` | Write | Clear a document from a checklist item |

## Example prompts

- "Find my Costco receipts from last year and total them."
- "Which warranties expire in the next three months?"
- "Upload this PDF to my archive and tell me what it is once it's processed."
- "Make a tax collection for 2026 and show me which documents are still missing."
- "List everything with a deadline this month and add a reminder for the lease renewal."

## Use the Skill

[`skills/pocket-archive/SKILL.md`](skills/pocket-archive/SKILL.md) contains agent-independent archive workflows: finding records, uploading files and waiting for processing, and downloading originals. An agent that supports `SKILL.md` can load this folder through its own skill installation mechanism. Other frameworks can use the instructions as task guidance alongside the discovered MCP tool schemas. MCP clients can call the tools without installing the Skill.

```sh
# Claude Code
mkdir -p ~/.claude/skills/pocket-archive && curl -fsSL https://raw.githubusercontent.com/ShisunXia/pocket-archive-mcp/main/skills/pocket-archive/SKILL.md -o ~/.claude/skills/pocket-archive/SKILL.md

# Codex
mkdir -p ~/.codex/skills/pocket-archive && curl -fsSL https://raw.githubusercontent.com/ShisunXia/pocket-archive-mcp/main/skills/pocket-archive/SKILL.md -o ~/.codex/skills/pocket-archive/SKILL.md
```

In the Claude app, download [pocket-archive.zip](https://www.pocketarchive.io/skills/pocket-archive.zip) and upload it under **Customize → Skills**. Code execution must be turned on.

The Skill provides workflow guidance; the host supplies MCP discovery, OAuth, credential storage, tool calls and any binary upload/download capability. If the host cannot perform binary HTTP transfers, the agent directs the user to upload on the web rather than claiming that a local file path was uploaded.

## Files

- [`mcp.json`](mcp.json): [Agent Plugins MCP configuration](https://agent-plugins.org/schemas/1.0.0/mcp.schema.json) with a `streamable-http` server. Use it in hosts that support this format; other clients should configure the endpoint in their native format.
- [`skills/pocket-archive/SKILL.md`](skills/pocket-archive/SKILL.md): shared workflow instructions.
- [`.codex-plugin/plugin.json`](.codex-plugin/plugin.json) and [`.mcp.json`](.mcp.json): optional Codex plugin adapter. The `http` type in this adapter maps to the same Streamable HTTP endpoint. Package this repository, including its dotfiles, as one plugin root following the [Codex plugin format](https://developers.openai.com/plugins/build/plugins).
- [`skills/pocket-archive/agents/openai.yaml`](skills/pocket-archive/agents/openai.yaml): optional OpenAI display and dependency metadata; other agents can ignore it.

- [`server.json`](server.json): entry for the [official MCP Registry](https://registry.modelcontextprotocol.io).
- [`glama.json`](glama.json): maintainer claim for [Glama](https://glama.ai/mcp/servers).

A configuration file is not installed into an agent automatically. Include the adapter and Skill your client needs when packaging for it. This repository contains no credentials.

## Privacy and security

Authentication follows the [MCP authorization specification](https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization). Document text is your data and agents are instructed to treat it as data, never as instructions. See the Pocket Archive [privacy policy](https://www.pocketarchive.io/privacy) for how your data is handled.

## Support

- Setup guide: [pocketarchive.io/ai-agents](https://www.pocketarchive.io/ai-agents)
- Help and contact: [pocketarchive.io/contact](https://www.pocketarchive.io/contact)
- Bugs or questions about this repository: [open an issue](https://github.com/ShisunXia/pocket-archive-mcp/issues)
- [Terms](https://www.pocketarchive.io/terms) · [Privacy](https://www.pocketarchive.io/privacy)

## License

The files in this repository are released under the [MIT License](LICENSE). The Pocket Archive name and logo are not covered by this license. Use of the Pocket Archive service is governed by its [Terms](https://www.pocketarchive.io/terms).
