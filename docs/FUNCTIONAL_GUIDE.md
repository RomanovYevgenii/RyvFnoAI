# RyvFnoAI — Functional Guide

## Overview

RyvFnoAI adds an AI-powered assistant to Dynamics 365 Finance & Operations. Users interact with the AI through a chat interface embedded in D365FO forms. The AI can query data, create and update records, generate reports, navigate forms, and execute complex multi-step workflows — all controlled by per-user permissions.

---

## Getting Started

### Prerequisites

- D365FO 10.0.x environment (PU 50+)
- System administrator role for initial setup
- An API key from one of the supported AI providers

### Supported AI Providers

| Provider | Description |
|----------|-------------|
| **Azure OpenAI** | Microsoft-hosted, deployment-based, supports API versioning |
| **OpenAI** | Direct OpenAI API access (GPT-4o, GPT-4, etc.) |
| **Anthropic Claude** | Claude API with optional workspace support |

---

## Configuration

### Step 1: AI Bot Parameters

**Navigation:** System administration > Setup > RYV AI Bot > AI Bot Parameters

| Field | Description |
|-------|-------------|
| Enable AI Bot | Master on/off switch for the entire feature |
| AI Provider | Select: Azure OpenAI, OpenAI, or Anthropic Claude |
| Endpoint URL | Provider API base URL |
| API Key | Provider API key (stored encrypted) |
| Model ID | Model identifier (e.g., `gpt-4o`, `claude-sonnet-4-20250514`) |
| Max Tokens | Maximum tokens per AI response |
| Compact after messages | Number of messages before context compaction (0 = disabled, default 20) |

**Azure OpenAI additional fields:**

| Field | Description |
|-------|-------------|
| Deployment Name | Azure deployment resource name |
| API Version | API version string (e.g., `2024-02-01`) |

**Anthropic additional fields:**

| Field | Description |
|-------|-------------|
| Workspace ID | Organization workspace (for org-level API keys) |

### Step 2: User Setup

**Navigation:** System administration > Setup > RYV AI Bot > User Setup

Each user who will use the AI Bot needs a setup record.

1. Click **New** to create a record — UserId defaults to the current user
2. Set **Enabled** = Yes
3. **Tools tab** — click **Add tool**, select tools from lookup, enable/disable individually
4. **Agents tab** — click **Add agent**, select agents from lookup, enable/disable individually
5. Table-level restrictions are enforced at runtime (admin UI for the table allow-list is planned for 1.1)

**Alternative:** Run the **Initialize AI Bot user setup** batch job (System administration > Periodic tasks) to auto-create the current user's setup with all tools and agents pre-populated.

#### Adding Tools

1. Click **Add tool** button on the Tools tab
2. A new row appears in the grid
3. Click the lookup arrow on the **Tool Name** field
4. The lookup shows all available tools not yet assigned to you (built-in + MCP server tools)
5. Select a tool — Description auto-fills
6. Toggle **Enabled** as needed

#### Adding Agents

1. Click **Add agent** button on the Agents tab
2. A new row appears in the grid
3. Click the lookup arrow on the **Agent ID** field
4. The lookup shows all enabled agents not yet assigned to you
5. Select an agent — Description auto-fills
6. Toggle **Enabled** as needed

### Step 3: MCP Servers (Optional)

**Navigation:** System administration > Setup > RYV AI Bot > MCP Servers

MCP (Model Context Protocol) servers extend the AI with external tool capabilities (e.g., Jira, Confluence, GitHub, custom APIs).

1. Click **New** to create a server record
2. Fill in: Server ID, Name, URL
3. Set authentication type: None, Bearer Token, or API Key
4. Provide credentials if required
5. Click **Test Connection** to verify connectivity
6. Click **Sync Tools** to import available tool definitions from the server
7. Enable/disable individual tools on the **Tools** tab

MCP tools automatically appear in the User Setup > Add Tool lookup after syncing.

### Step 4: Knowledge Base (Optional)

**Navigation:** System administration > Setup > RYV AI Bot > Knowledge Base

The knowledge base enables RAG (Retrieval-Augmented Generation) — the AI searches your documents to answer questions with context-specific information.

#### Adding Documents

| Source Type | How to Add |
|-------------|------------|
| Manual | Type or paste text directly |
| File | Attach a document file |
| URL | Enter a URL — content is fetched automatically |

#### Indexing

1. Create one or more documents
2. Click **Index** to chunk the document and generate embeddings
3. Status transitions: Draft -> Indexing -> Active (or Error)

#### Enabling RAG

In AI Bot Parameters, go to the **Semantic Search** tab:

| Setting | Default | Description |
|---------|---------|-------------|
| Enable Semantic Search | No | Master RAG switch |
| Embedding Endpoint URL | — | Embedding API URL |
| Embedding Model ID | — | Embedding model name |
| Auto-retrieve | No | Automatically search knowledge on every user message |
| Top results | 5 | Number of chunks to retrieve per query |
| Min similarity score | 0.7 | Minimum cosine similarity threshold (0-1) |
| Max context tokens | 2000 | Token budget for retrieved context |
| Chunk size | 500 | Tokens per chunk during indexing |
| Chunk overlap | 50 | Overlapping tokens between adjacent chunks |

---

## Extending RyvFnoAI with your own tools

RyvFnoAI is open for extension. Partners and customers can build company-specific tools, for example "check credit exposure", "find duplicate vendors" or a call to an internal service, **without changing RyvFnoAI code**. There are two ways:

**1. X++ tools in your own model**
- Implement the RyvFnoAI tool interface: a name, a description, a parameter schema and an execute method.
- Register the tool through the **published extension point** that RyvFnoAI raises while it builds its tool list.
- Your tools live in your model, so they survive RyvFnoAI updates.

**2. MCP servers, in any language**
- Build a Model Context Protocol server (C#, Python, Node.js, …) or use an existing one (Jira, Confluence, GitHub, internal APIs).
- Add it under **MCP Servers**, click **Sync Tools** (see Step 3), and its tools become available.

In both cases the tools appear in **User Setup → Add tool**, can be included in an agent's tool set, and follow the same **per-user tool permissions** as the built-in tools.

The X++ interface reference and a sample tool are provided with the package.

## Forms Reference

The chat (**AI Bot**) opens from the settings (gear) menu on any page. All setup forms are under **System administration > Setup > RYV AI Bot**:

| Form | Purpose |
|------|---------|
| **AI Bot** | Chat side panel (settings menu) — type messages, receive AI responses, use tools; can read the open form and selected record |
| **AI Bot Parameters** | Global configuration: provider, API keys, RAG settings |
| **User Setup** | Per-user tool/agent/table access management |
| **Agents** | Define AI personas with custom instructions and tool permissions |
| **MCP Servers** | Configure external MCP server connections |
| **Knowledge Base** | Manage and index documents for RAG |
| **Chat Sessions** | View and manage conversation history (filtered to current user) |

---

## Using the AI Bot

### Starting a Chat

1. On any page, open the **settings (gear) menu** in the top bar and choose **AI Bot**. The chat opens as a **side panel** next to the current form, so you can keep working in it.
2. Select an agent from the dropdown (or use the default agent)
3. Type your message and press Enter

### What the AI Can Do

#### Query Data
- "Show me all open sales orders for customer US-001"
- "How many vendor invoices were posted this month?"
- "Look up item BOM details for product D0001"

#### Create and Modify Records
- "Create a new customer with name Contoso Ltd"
- "Update the payment terms on PO-000123 to Net 30"
- "Delete the draft journal JV-0045"

Data modifications use a preview-then-confirm flow: the agent first shows what will change and executes only after you approve.

#### Navigate and Explore
- "Open the sales order form"
- "What fields does the CustTable have?"
- "Show me the enum values for CustPaymentMode"

#### Generate Reports and Files
- "Export all overdue invoices to CSV"
- "Generate a summary report of inventory on hand"

#### Use Memory
- "Remember that our fiscal year ends in March"
- "What do you remember about the AP process?"

The AI maintains persistent memory across chat sessions.

### Agent Selection

Different agents can be configured for different purposes:

- **Default agent** — general-purpose D365FO assistant
- **Custom agents** — specialized personas (e.g., AP specialist, inventory analyst) with tailored instructions and restricted tool sets

Agents are configured by administrators in the Agents form.

### Chat Sessions

- Each conversation is saved as a session
- Sessions are private — each user sees only their own
- Old sessions can be cleaned up via the Session Cleanup batch job
- The chat session list is sorted by last modified date (newest first)

---

## Agent Configuration

**Navigation:** System administration > Setup > RYV AI Bot > Agents

| Field | Description |
|-------|-------------|
| Agent ID | Unique identifier (read-only after creation) |
| Name | Display name shown in the chat agent selector |
| Description | Brief description of what this agent does |
| Instructions | System prompt — defines the agent's behavior, expertise, and personality |
| Max Iterations | Maximum tool call loop iterations (default 5, up to 25 for complex workflows) |
| Enabled Tools | Comma-separated list of allowed tools (empty = all tools available) |
| User ID | Owner — empty means shared across all users |
| Is Default | If Yes, this agent is auto-selected when chat opens |
| Enabled | Master enable/disable for this agent |

### Agent Scope

| User ID | Is Default | Scope |
|---------|------------|-------|
| (empty) | No | Shared agent available to all users |
| (empty) | Yes | System default agent for all users |
| specific user | No | Private agent for that user only |
| specific user | Yes | Default agent for that user (overrides system default) |

---

## Batch Jobs

### Initialize AI Bot User Setup

**Navigation:** System administration > Periodic tasks > Initialize AI Bot user setup

- Creates or finds the User Setup record for the current user
- Populates all missing tools (built-in registry + MCP server tools)
- Populates all missing agents

### Session Cleanup

**Navigation:** System administration > Periodic tasks > Session cleanup

| Parameter | Default | Description |
|-----------|---------|-------------|
| Retention days | 30 | Delete sessions older than this many days |

- Can be scheduled as a recurring batch job
- Deletes only sessions older than the retention period

---

## Security

### Who Can Access What

| Role | Duty | Access |
|------|------|--------|
| Administrator | `RYVAIBotAdminDuty` | Full access — configure, create, update, delete everything |
| Clerk / End User | `RYVAIBotClerkDuty` | View-only — read configuration, use chat |

### User-Level Controls

Beyond role-based security, each user's access is controlled through their User Setup record:

- **Tool Access** — enable/disable individual tools per user
- **Agent Access** — control which agents a user can select
- **Table Access** — runtime-enforced table allow-list (admin UI planned for 1.1)

### Data Protection

- API keys are encrypted at rest and excluded from data entities
- Chat sessions are private: each user sees only their own
- Data entities exclude sensitive fields such as API keys and system prompts

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| "AI Bot is not configured" | Set all required fields in AI Bot Parameters: Provider, Endpoint URL, API Key, Model ID |
| AI Bot not appearing on forms | Check: Enable AI Bot = Yes, User Setup exists with Enabled = Yes, user has RYVAIBotClerkDuty or RYVAIBotAdminDuty |
| "Tools and agents initialized for 0 user(s)" | Run the Initialize batch job as the user who needs access |
| "Access denied to agent: X" | Add the agent to the user's Agent Access in User Setup |
| MCP tools not in Add Tool lookup | Verify MCP server is Enabled, click Sync Tools, verify individual tools are Enabled |
| Chat session not loading | Clean up old sessions via Session Cleanup batch job |
| Tool execution errors | Check: table access granted, tool enabled for user, tool in agent's EnabledTools list |
