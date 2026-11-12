# RyvSoft AI — Functional Guide

## Overview

RyvSoft AI adds an AI-powered assistant to Dynamics 365 Finance & Operations. Users interact with the AI through a chat interface embedded in D365FO forms. The AI can query data, create and update records, generate reports, navigate forms, and execute complex multi-step workflows — all controlled by per-user permissions. This guide describes release 1.4.

---

## Getting Started

### Prerequisites

- Dynamics 365 Finance & Operations 10.0.45 or later
- System administrator role for initial setup
- An API key from one of the supported AI providers

### Supported AI Providers

| Provider | Description |
|----------|-------------|
| **Azure OpenAI** | Microsoft-hosted, deployment-based, supports API versioning |
| **OpenAI** | Direct OpenAI API access with your own key (any current GPT model) |
| **Anthropic Claude** | Claude API with optional workspace support |

---

## Configuration

### Step 1: AI Bot Parameters

**Navigation:** RyvSoft AI > Setup > AI Bot parameters

| Field | Description |
|-------|-------------|
| Enable AI Bot | Master on/off switch for the entire feature |
| AI Provider | Select: Azure OpenAI, OpenAI, or Anthropic Claude |
| Endpoint URL | Provider API base URL |
| API Key | Provider API key (stored encrypted) |
| Model ID | Model identifier as your provider names it (e.g., `claude-sonnet-5-5`, `claude-opus-5-5`, or the GPT model name from your OpenAI account) |
| Max Tokens | Maximum tokens per AI response |
| Compact after messages | Number of messages before context compaction (0 = disabled, default 20) |
| Model response timeout (seconds) | How long to wait for the model's answer; raise it for slower local models |

The **Connection** tab also has **Test Connection** to check the provider settings, the limits for [scheduled runs](#scheduled-agent-runs), and **Allow script injection** (leave it off unless you trust the model to script forms through the task recorder).

The **Knowledge** tab holds **Initialize Knowledge**, which scans your environment's metadata into instructions for the agent (with **Include standard models in scan** to cover Microsoft models too). The **Additional configuration** tab holds **Enable preview widget types** (see [AI workspace](#ai-workspace-preview)), **Refresh label index** (see [Label index](#step-5-label-index)) and the switches for the [External API](EXTERNAL_API.md).

The **Activity log** tab sets the [agent activity log](#agent-activity-log): **Log agent activity**, **Store request text** (tell your employees that their requests are logged) and **Retention (days)** (default 365; 0 = keep for ever).

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

**Navigation:** RyvSoft AI > Setup > User setup

Each user who will use the AI Bot needs a setup record.

1. Click **New** to create a record — UserId defaults to the current user
2. Set **Enabled** = Yes
3. **Tools tab** — click **Add tool**, select tools from lookup, enable/disable individually
4. **Agents tab** — click **Add agent**, select agents from lookup, enable/disable individually
5. **Knowledge tab** — the knowledge base documents this user may use. Empty list: every document. With rows: only the enabled ones; a document added later must be added here. **Initialize documents** adds every document the user does not have yet
6. Table-level restrictions are enforced at runtime (an admin UI for the table allow-list is planned)

Use **Enable selected** / **Disable selected** to switch several tools at once. **Initialize Tools** and **Initialize Agents** in the action pane reset the user's tool or agent list to everything available.

**Connection tab:** a user can get their own model connection instead of the system one. Set **Use custom connection** and fill in **Connection settings** (provider, endpoint, key, model); otherwise the settings in AI Bot parameters apply.

**Memory tab:** what the agent remembers about this user (see [Memory](#use-memory)).

**Alternative:** Run the **Initialize AI Bot user setup** batch job (RyvSoft AI > Periodic tasks) to auto-create the current user's setup with all tools and agents pre-populated.

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

**Navigation:** RyvSoft AI > Setup > MCP servers

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

**Navigation:** RyvSoft AI > Setup > Knowledge base

The knowledge base enables RAG (Retrieval-Augmented Generation) — the AI searches your documents to answer questions with context-specific information.

#### Adding Documents

| Source Type | How to Add |
|-------------|------------|
| Manual | Type or paste text directly |
| File | Attach a document file |
| URL | Enter a URL — content is fetched automatically; optionally include the pages it links to on the same site |

#### Indexing

1. Create one or more documents
2. Click **Index** to chunk the document and generate embeddings, or **Index All** for every document; indexing runs in batch
3. Status transitions: Draft -> Indexing -> Active (or Error); the **Statistics** tab shows the size of the document and any error

#### Enabling RAG

In AI Bot Parameters, go to the **Semantic Search** tab:

| Setting | Default | Description |
|---------|---------|-------------|
| Enable Semantic Search | Yes | Master RAG switch |
| Embedding Endpoint URL | — | Embedding API URL |
| Embedding Model ID | — | Embedding model name |
| Auto-retrieve | Yes | Automatically search knowledge on every user message |
| Top results | 5 | Number of chunks to retrieve per query |
| Min similarity score | 0.3 | Minimum cosine similarity threshold (0-1) |
| Max context tokens | 2000 | Token budget for retrieved context |
| Chunk size | 500 | Tokens per chunk during indexing |
| Chunk overlap | 50 | Overlapping tokens between adjacent chunks |

### Step 5: Label index

The agent finds tables, fields, enums and menu items by their labels, using a label index built per language. Build it with **Refresh label index** in **AI Bot parameters > Additional configuration**; it runs as a batch job. Run it again after a deployment adds or changes objects or their labels. If the index has never been built, the first search that needs it queues the build automatically.

### Moving setup between environments

Agents (with their instructions), the tools each agent may use, and the AI Bot parameters can be exported and imported with **Data management**. API keys and the system prompt are never included. The agents entity carries the agents' instructions, so it is available to administrators only. When importing, import the agents before their tool access.

---

## Extending RyvSoft AI with your own tools

RyvSoft AI is open for extension. Partners and customers can build company-specific tools, for example "check credit exposure", "find duplicate vendors" or a call to an internal service, **without changing RyvSoft AI code**. There are two ways:

**1. X++ tools in your own model**
- Implement the RyvSoft AI tool interface: a name, a description, a parameter schema and an execute method.
- Register the tool through the **published extension point** that RyvSoft AI raises while it builds its tool list.
- Your tools live in your model, so they survive RyvSoft AI updates.

**2. MCP servers, in any language**
- Build a Model Context Protocol server (C#, Python, Node.js, …) or use an existing one (Jira, Confluence, GitHub, internal APIs).
- Add it under **MCP Servers**, click **Sync Tools** (see Step 3), and its tools become available.

In both cases the tools appear in **User Setup → Add tool**, can be included in an agent's tool set, and follow the same **per-user tool permissions** as the built-in tools.

The X++ interface reference and a sample tool are provided with the package.

## Forms Reference

The chat (**AI Bot**) opens from the settings (gear) menu on any page. Everything else is in the **RyvSoft AI** module:

| Form | Where | Purpose |
|------|-------|---------|
| **AI Bot** | Settings (gear) menu | Chat side panel — type messages, receive AI responses, use tools; can read the open form and selected record |
| **AI workspace** | Workspaces | Your own pages built from F&O data (see [AI workspace](#ai-workspace-preview)) |
| **Reports** | AI Reporting (Preview) | Reports that fill your own templates (see [AI Reporting](#ai-reporting-preview)); data sets, templates and macros are under **Advanced** |
| **Runs** | Inquiries and reports | Every AI Reporting run with the file it produced |
| **Chat sessions** | Inquiries and reports | Your conversation history |
| **Agent activity log** | Setup | Who did what through the agent, for all users (administrators); see [Agent activity log](#agent-activity-log) |
| **AI Bot parameters** | Setup | Global configuration: provider, API keys, RAG settings |
| **Agents** | Setup | Define AI personas with custom instructions and tool permissions |
| **AI Bot schedules** | Setup | Run an agent on a schedule or on demand through the batch queue (see [Scheduled agent runs](#scheduled-agent-runs)) |
| **User setup** | Setup | Per-user tool, agent and connection settings (administrators) |
| **My setup** | Setup | Your own default agent and memories |
| **User memories** | Setup | What the agent remembers, for all users (administrators) |
| **Knowledge base** | Setup | Manage and index documents for RAG |
| **MCP servers** | Setup | Configure external MCP server connections |

---

## Using the AI Bot

### Starting a Chat

1. On any page, open the **settings (gear) menu** in the top bar and choose **AI Bot**. The chat opens as a **side panel** next to the current form, so you can keep working in it.
2. Select an agent from the dropdown (or use the default agent)
3. Type your message and press Enter

### Attaching files and voice input

- **Attach a file** to your question: images (PNG, JPG, GIF, WEBP), PDF, Word (DOCX), Excel (XLSX) and text files such as TXT, CSV, JSON, XML, Markdown, log or HTML. A long file is read in parts, so the agent can work through all of it.
- **Speak instead of typing**: click the microphone button. The panel uses your browser's speech recognition and types what you say into the message box, so you can check it before you send.

### What the AI Can Do

#### Query Data
- "Show me all open sales orders for customer US-001"
- "How many vendor invoices were posted this month?"
- "Look up item BOM details for product D0001"

#### Create and Modify Records
- "Create a new customer with name Contoso Ltd"
- "Update the payment terms on PO-000123 to Net 30"
- "Delete the draft journal JV-0045"

Data modifications are never written by the agent directly. The agent prepares the change and the panel shows a card with the exact preview; only your **Confirm** click saves it (Cancel discards it). The check is done in X++: if the record was changed after the preview, nothing is saved and the agent prepares a fresh preview. Several pending changes can be confirmed or cancelled together.

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

The AI maintains persistent memory across chat sessions. Each memory has a category (Preference, Fact, Pattern or Procedure). You can review and delete your own memories in **My setup**; administrators see everyone's in **User memories**.

### Agent Selection

Different agents can be configured for different purposes:

- **Default agent** — general-purpose D365FO assistant
- **Custom agents** — specialized personas (e.g., AP specialist, inventory analyst) with tailored instructions and restricted tool sets

Agents are configured by administrators in the Agents form. Each user can pick their own default agent among the agents they have access to in **RyvSoft AI > Setup > My setup** (**Set as default**).

### Chat Sessions

- Each conversation is saved as a session
- Sessions are private — each user sees only their own
- Sessions started by your [schedules](#scheduled-agent-runs) or through the [External API](EXTERNAL_API.md) are saved the same way
- **RyvSoft AI > Inquiries and reports > Chat sessions** lists your sessions, newest first; **Activate** opens the selected session in the AI Bot, **Delete** removes it
- Old sessions can be cleaned up via the Session Cleanup batch job

### Agent activity log

**Navigation:** RyvSoft AI > Setup > Agent activity log (administrators)

Every session is logged, whatever started it: the chat, a schedule, the External API or the AI workspace. One row per event:

| Event type | When |
|------------|------|
| Request | A user (or a schedule or external system) asked the agent something |
| Tool call | The agent ran a tool; with its duration and the tables it used |
| Tool denied | The agent wanted a tool it is not allowed to use; with the reason |
| Change proposed | The agent prepared a change for a Confirm |
| Change confirmed / cancelled / rejected / failed / expired | What happened to that change |

- Columns: user, session title and ID, agent, calling agent, schedule, source, company, event type, tool, tables, summary, records, duration. A change row's summary shows the changed fields, for example `CustGroup = 30, Name = Wholesale`; for a delete, only the record's key.
- Filters: date range, user, agent, event type, source, company, **Changes only**.
- **Open session** shows the whole session in order (read-only), with its title, user, agent, source, schedule and start time.
- **Open record** opens the changed record in its own form and company, subject to your own menu item security.
- No business data is copied into the log beyond what identifies the record and the change.

### Background runs and the plan card

A long task can move to a standard F&O batch job after you confirm, so you can close the browser and come back to the result in the same session. Large data is worked through in portions, so the size of the data is not limited by what the model can read at once.

When a background run will change data, the agent first shows a **plan card**: the steps, the companies, the files it will produce (read only) and, for each record, the change it will make; the full list of records can be downloaded as CSV. The plan card is the only approval a background run gets, and nothing outside the plan is changed: a step the plan does not cover is not run and is reported as a proposal in the final answer.

---

## AI workspace (preview)

**Navigation:** RyvSoft AI > Workspaces > AI workspace

The AI workspace lets each user build their own F&O pages from blocks: data grids, KPI and number tiles, bar charts, work lists, text and quick links. Describe a block in plain words, check the real rows it returns and accept it; from then on it runs from live data with your own permissions. Blocks can be linked, so selecting a row in one block filters the blocks linked to it.

**AI blocks** (the agent writes the answer or prepares the query) and **action widgets** (a button that runs a tool) are preview features. They are hidden until an administrator sets **Enable preview widget types** in **AI Bot parameters > Additional configuration**. An action widget never writes without confirmation.

---

## AI Reporting (preview)

**Navigation:** RyvSoft AI > AI Reporting (Preview) > Reports

AI Reporting fills your own file templates with F&O data: Excel workbooks, XML, JSON, CSV and text files. A report brings together three parts, all reachable from the report:

- **Data** (*Advanced > Data sets*) — what is read: tables and their joins, data providers or SQL queries, columns, filters, **Questions** (the parameters asked at run time) and the companies to read.
- **Template file** (*Advanced > Templates*) — the file to fill. Upload it, then **Read file structure**; every new upload is kept as a file version.
- **Layout** — which value goes into each cell or element: a column, a question, a constant or a formula. **Match by name** fills every cell whose name matches a column or question; visibility rules hide parts of the output; **Preview** shows the result. Reusable formula pieces are kept as **Macros** (*Advanced > Macros*).

The **Readiness** tab lists what is still missing and offers **Fix**. A report starts as **Draft**: test it with **Preview on a record**, then **Publish** to freeze the version that people and agents run (**Unpublish** returns it to Draft). A **derived report** (**Create derived report**) takes its whole format from another report and keeps only what it changes.

Every run — a preview, an on-demand run or a batch run — is listed with its file in **RyvSoft AI > Inquiries and reports > Runs**.

**AI Reports button on F&O forms.** When a report takes one of its questions from the open record (**Take from the open record**), users who may run reports get an **AI Reports** button on the F&O forms for that record type. It runs the published report for the selected record without retyping it.

The AI Reporting agents of the [Agent Library](AGENT_LIBRARY.md) help design and debug reports in the chat.

---

## Agent Configuration

**Navigation:** RyvSoft AI > Setup > Agents

| Field | Description |
|-------|-------------|
| Agent ID | Unique identifier (read-only after creation) |
| Name | Display name shown in the chat agent selector |
| Description | Brief description of what this agent does |
| Instructions | System prompt — defines the agent's behavior, expertise, and personality |
| Default Agent | If Yes, this agent is auto-selected when chat opens, unless the user picked another default in My setup |
| Enabled | Master enable/disable for this agent |

**Tools tab** — the tools this agent may use. **Add tool** / **Remove tool**, **Initialize Tools** (adds the tools the agent does not have yet and keeps existing settings), **Enable selected** / **Disable selected**. A tool must be enabled both for the agent and for the user.

**Tool Configuration tab** — **Max Tool Iterations**: how many tool-call steps the agent may take for one request. Simple assistants need few; long agentic workflows need more.

---

## Batch Jobs

### Initialize AI Bot User Setup

**Navigation:** RyvSoft AI > Periodic tasks > Initialize AI Bot user setup

- Creates or finds the User Setup record for the current user
- Populates all missing tools (built-in registry + MCP server tools)
- Populates all missing agents

### Scheduled agent runs

**Navigation:** RyvSoft AI > Setup > AI Bot schedules

A schedule runs one agent with one instruction in one company, through the batch queue, with nobody in the chat. Each run creates a normal AI Bot session that you can open and continue.

| Field | Description |
|-------|-------------|
| Name | What the run is for, e.g. "Morning collections call list" |
| Agent ID | The agent that does the work, with its own instructions and tools |
| Instruction | The task in plain words, as you would type it in the chat |
| Company | The company the run works in |
| Mode | **Read-only** — the agent reports and never changes data; proposed changes are listed in the answer. **Propose** — the agent prepares changes as Confirm cards and stops; nothing is written until a person confirms them |
| Enabled | Switches the schedule on or off |
| Notify | The user who gets a notification with a link to the session (the owner by default) |

Buttons:
- **Recurrence** — when the schedule runs (for example, weekdays at 08:00).
- **Run now** — queue one run at once.
- **History** — the runs of this schedule, with status and the number of proposals.
- **Open session** — open the session of the selected run in the AI Bot.

A scheduled run gets the same rights as the owner in the chat: the same roles, company access and AI Bot user setup (tools, agents, tables). It can never write without a Confirm.

### Activity log clean-up

**Navigation:** RyvSoft AI > Periodic tasks > Clean up agent activity log

| Parameter | Default | Description |
|-----------|---------|-------------|
| Retention (days) | 0 | Keep this many days of log. 0 = use **Retention (days)** from AI Bot parameters |

- Schedule it once as a recurring batch job; then change the retention in AI Bot parameters only
- When the retention in AI Bot parameters is 0 too, nothing is deleted

### Session Cleanup

**Navigation:** RyvSoft AI > Periodic tasks > AI Bot session cleanup

| Parameter | Default | Description |
|-----------|---------|-------------|
| Retention days | 30 | Delete sessions older than this many days |

- Can be scheduled as a recurring batch job
- Deletes only sessions older than the retention period
- Also expires changes proposed by scheduled runs that nobody confirmed in time, and removes memories of users whose AI Bot user setup is gone or disabled

---

## Security

### Who Can Access What

Six duties come with RyvSoft AI; add them to roles of your choice.

| Duty | For | Access |
|------|-----|--------|
| **AI Bot administrator duty** | Administrators | Configure the AI Bot: parameters, agents, user setup, schedules, knowledge base, MCP servers, user memories, batch jobs, data entities; read the agent activity log |
| **AI Bot clerk duty** | Users | Use the chat; maintain their own My setup (default agent, memories) and their own AI workspace pages; read the configuration; use the knowledge base through the agent only |
| **View the AI workspace** | Users who only look at workspace pages | Open the AI workspace |
| **Maintain AI Reporting definitions** | Report designers | Create and change data sets, templates and reports; run them and see the run history |
| **Run AI Reporting reports** | Report users | Run published reports |
| **AI Bot external access** | The F&O user of an external system | Call the agent through the [External API](EXTERNAL_API.md) |

### User-Level Controls

Beyond role-based security, each user's access is controlled through their User Setup record:

- **Tool Access** — enable/disable individual tools per user
- **Agent Access** — control which agents a user can select
- **Knowledge Access** — control which knowledge base documents the agent may use for the user (empty list = all)
- **Table Access** — runtime-enforced table allow-list (an admin UI is planned)

### Data Protection

- API keys are encrypted at rest and excluded from data entities
- Chat sessions are private: each user sees only their own; administrators see who did what in the agent activity log
- Data entities exclude sensitive fields such as API keys and system prompts; the agents entity carries agent instructions and is for administrators only

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| "AI Bot is not configured" | Set all required fields in AI Bot Parameters: Provider, Endpoint URL, API Key, Model ID |
| AI Bot not appearing on forms | Check: Enable AI Bot = Yes, User Setup exists with Enabled = Yes, user has the AI Bot clerk duty or AI Bot administrator duty |
| "Tools and agents initialized for 0 user(s)" | Run the Initialize batch job as the user who needs access |
| "Access denied to agent: X" | Add the agent to the user's Agent Access in User Setup |
| MCP tools not in Add Tool lookup | Verify MCP server is Enabled, click Sync Tools, verify individual tools are Enabled |
| Chat session not loading | Clean up old sessions via Session Cleanup batch job |
| Tool execution errors | Check: table access granted, tool enabled for the user (User setup > Tools) and for the agent (Agents > Tools) |
| The agent says the label index is not built or out of date | Run **Refresh label index** in AI Bot parameters > Additional configuration |
| The model answer times out | Raise **Model response timeout (seconds)** in AI Bot parameters |
| No AI block or action widget in the workspace palette | Set **Enable preview widget types** in AI Bot parameters > Additional configuration |
| No AI Reports button on a form | A report question must be taken from the open record, and the user needs the Run AI Reporting reports duty; see the report's Readiness tab |
