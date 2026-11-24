# RyvSoft AI — Release Notes

## 1.5.0.3 (November 2026)

### AI Bot access by security role

You no longer need a setup row for every user. Give tools, agents, tables and knowledge base documents to a **security role** once, and every user who holds the role gets them. New employees get access as soon as they get the role, and a user who changes jobs gets the access of their new role.

**Navigation:** RyvSoft AI > Setup > AI Bot access setup (formerly *User setup*)

- **One list, two kinds of rows:** **User** rows (as before) and **Role** rows. Use **Show: All / Users / Roles** to filter.
- **New** opens a short dialog: choose the **Setup type**, then the user or the security role. The role lookup lists only roles that have no access setup yet.
- **Both kinds have the same tabs:** Tools, Agents, Tables, Knowledge, with **Set as default** on the Agents tab. Connection and Memory stay on User rows only: a model connection or an MCP credential belongs to one person and is never shared through a role.
- **Effective access** tab on a User row: the tools, agents, tables and documents the user really gets, which row each one comes from, and in which companies.
- **Users with this role** tab on a Role row: who holds the role and for which companies.
- **Tables** tab: the table allow-list now has its own tab, for users and for roles.

**How a user's access is decided**

| User's own row | Result |
|---|---|
| Exists, **Use role access** = No | The user's own lists decide, exactly as in 1.4. Every existing row works this way after the upgrade |
| Exists, **Use role access** = Yes, or no own row | The Role rows of all roles the user holds are combined |

**A user row and Role rows are never mixed.** Access always comes from one place, so you can always tell where it came from:

- A user with their own row and **Use role access** = No gets only that row's lists, even if they also hold roles that have Role rows. Their roles add nothing.
- With **Use role access** = Yes the roles decide; the user's row adds no tools, agents, tables or documents. It keeps only the user's personal settings: model connection, MCP credentials, memories and default agent.
- **Enabled** = No on the user's own row blocks that user in both cases, even when their roles would give access. Use it to stop one person without changing their roles.
- To give one user a little more than their role, create a role for the extra access, set up a Role row for it and assign that role to the user.

When roles are combined:

- **Tools and agents:** the union of what the roles list. A role that lists no tools or agents gives none, so an empty role never opens everything.
- **Companies:** an agent that comes from a role assigned for some companies only (a role assignment with organizations) works only in those companies. In other companies it is not offered, and a direct request is refused.
- **Tables and knowledge:** if any of the user's roles has no list, the user is not restricted; otherwise the union of the lists applies.
- **Default agent:** the user's own choice in My setup while it is still allowed; otherwise the default of the role with the lowest **Priority** number.
- The agent's own tool list and every F&O security check still apply. A Role row never gives a right the user does not have in F&O.
- Role assignments that are disabled or outside their valid dates are ignored. A change to a user's roles takes effect within 5 minutes; a change in AI Bot access setup takes effect immediately.

**Who may set up roles:** a system administrator may set up any role. Anyone else with access to the form may set up only the roles they hold themselves; on other roles' rows the line buttons are disabled.

**New security role:** **AI Bot access administrator** maintains AI Bot access setup, without the rest of the AI Bot administration.

**Activity log:** a refused request is logged with the reason, for example *no access setup for the user or any of their roles* or *agent X is not allowed in company Y*.

### Agent library: export and import as JSON

Move agents and skills between environments as one file, with full instruction texts of any length. Data management (DMF) text imports cut a text field at 32,768 characters, so a long agent instruction could arrive cut off; the library file has no such limit. The DMF entities are still there and work as before.

**Navigation:** RyvSoft AI > Periodic tasks > Agent library > **Export library** / **Import library**, or the **Library** tab on the Agents and Skills forms.

- **Export library** writes all shared agents (with their tools and skill links) and all skills (with their required tools) to one UTF-8 JSON file. Run in batch, the file is attached to the batch job.
- **Import library** checks the whole file before it writes anything. It creates what is missing and updates what exists, by agent ID and skill ID, in one transaction. It never deletes, and it skips rows that have not changed.
- **Preview only** shows what the import would change, without writing anything. Run it first.
- **Rights:** export is in the agent and skill View and Maintain privileges, import only in Maintain.

### Initialize Knowledge runs in batch

**Initialize Knowledge** (the AI writes the environment knowledge from your metadata) can take several minutes. It is now a batch job and runs in batch by default, so nobody has to keep a session open. It is also under RyvSoft AI > Periodic tasks.

### Standard F&O reports from the agent

The agent can now run standard F&O (SSRS) reports and attach the result to the chat as PDF, Excel, Word or CSV, for example *"send me the customer aging report for USMF as Excel"*. It works in the chat and in batch runs.

- The report runs with the user's own rights: the user needs access to the report's menu item, and F&O security applies as for a report started from the menu.
- The agent fills the report's parameters and ranges by name. If a parameter is unknown or the report refuses to run, the agent gets the reason and can correct the request.
- There is no new tool to grant: reports go through the existing file export tool (`start_export`).

### AI workspace

**Share pages.** A page owner can share a page with users, security roles or everyone, from the new **Access** panel. Viewers find it in their launcher read-only and see only the data their own rights allow. A viewer can **make a copy** to change it. Sharing needs the new privilege **Share AI workspace pages**.

**Buttons.** A new Button widget can:

- open a form or report with the record selected in another widget, also in another company;
- ask the agent in a new chat, with fields of the selected record filled into the question;
- export a widget to CSV (all rows, not only the visible page);
- go to another page, open a web link, or refresh the page.

**New widgets:**

- **Trend** by day, week, month, quarter or year; periods with no data show as zero.
- **Pie / donut.**
- **Pivot** with heat map and totals.
- **Record card** with totals from related data.
- **KPI** gains a period, the change against the previous period, a sparkline and a target.

**Click to filter.** Clicking a bar, a slice or a pivot cell now filters the linked widgets by that value.

**Fixes:**

- F&O table rights are checked for every table a widget reads, and record level security always applies.
- A KPI counts all rows; grouping without a measure counts rows.
- With nothing selected in the driving widget, a linked widget shows a message instead of filtering on a blank value.
- Saving a widget with joined data sources no longer overwrites a joined source.
- Removed delivered templates are disabled instead of deleting the widgets users built from them.

### AI Reporting: published reports keep their macros

Publishing a report now freezes the formula macros it uses, including macros called by other macros, into the published version. Editing, disabling, or deleting a macro no longer changes the output of reports that are already published; Preview still uses the current macros, and the report shows that it has unpublished changes until it is published again. Reports that were published before this update and use macros show unpublished changes until they are published again, because their published version does not yet include the macros. The help for the output file name now lists only the supported placeholders ({{ReportId}} and {{RunDate}}), and the unsupported file type message now includes CSV and Text.

### AI Reporting and AI workspace agents rebuilt as core plus skills

The AI Reporting assistants and the AI workspace designer now keep a short core instruction and load the details only when needed: 12 reporting skills (datasets, SQL sources, formulas, variables and macros, Excel, XML/JSON and CSV/TXT templates, mappings, publish and derive, run and debug, e-invoice XML) and 9 workspace skills. Their instructions were checked against the product and corrected. Answers are more precise, and each request uses fewer tokens. Get them by importing the agent library file.

### Upgrade steps

1. Run a database synchronization (new fields and a new unique index on the access setup table, and a new table for shared workspace pages).
2. To share workspace pages, give the privilege **Share AI workspace pages** to the roles that should share.
3. To get the rebuilt AI Reporting and AI workspace agents, import the 1.5 agent library file (delivered with the subscription) with **Import library**, with **Preview only** first.
4. Publish again the AI Reporting reports that use macros, so that their published version includes the macros.
5. Existing user rows keep their own lists, so AI Bot access does not change until you add Role rows or switch a user's **Use role access** to Yes.
6. To move a team to role-based access: create a Role row for their role, fill its Tools and Agents tabs, check the **Effective access** tab of one user, then set **Use role access** = Yes on the users' own rows (or delete those rows).

## 1.4.0.1 (November 2026)

### Agent activity log: who did what

Every session is now logged: you can see who asked the agent what, what it did, and what changed. This covers chats, scheduled runs, External API calls and the AI workspace.

**Navigation:** RyvSoft AI > Setup > Agent activity log

- **One row per event:**
  - each request;
  - each tool call, with its duration and the tables it used;
  - each tool the agent was not allowed to use, with the reason;
  - each change it proposed, and what happened to it: confirmed, cancelled, rejected, failed or expired.
- **Columns:** user, session title and ID, agent, calling agent (for sub-agents), schedule, source, company, event type, tool, tables, summary, records and duration. Each change row shows what changed, for example `CustGroup = 30, Name = Wholesale`.
- **Filters:** date range, user, agent, event type, source, company, and **Changes only**.
- **Open session** shows the whole session in order, read-only, with a header: title, user, agent, source, schedule and start time.
- **Open record** opens the changed record in its own F&O form, in its company. It follows the admin's own menu item security.
- The log is for administrators only: it is in the **AI Bot administrator** duty. Users still see only their own chat sessions.

**Settings:** AI Bot parameters > Activity log

| Field | Default | Description |
|-------|---------|-------------|
| Log agent activity | Yes | Record a row for every request, tool call, denial and change made through the agent |
| Store request text | Yes | Store the user's message text in the log. Tell your employees that their requests to the agent are logged |
| Retention (days) | 365 | How long the log is kept. 0 = keep it for ever |

Business data is not copied into the log. A change row identifies the record and the changed fields; for a deleted record, only its key is kept.

### Knowledge base access per user

- **User setup** has a new **Knowledge** tab. It works like the Agents tab: **Initialize documents**, **Add document**, **Remove document**, **Enable** and **Disable** for the marked rows.
- **Empty list:** the user can use every document, as before.
- **List with rows:** the user can use only the enabled documents. A document added later must be added to the list.
- The filter applies to the automatic context and to the `search_knowledge` tool, in the chat, scheduled runs, the External API and sub-agents.
- Clerks no longer open the **Knowledge base** form; they use the knowledge base through the agent. The form stays for administrators.
- Deleting a document also deletes its access rows.

### New batch job: Activity log clean-up

**Navigation:** RyvSoft AI > Periodic tasks > Clean up agent activity log

- Deletes log rows older than the retention period.
- Leave **Retention (days)** in the dialog at 0 to use the value from AI Bot parameters. Then you schedule the job once and change the retention in the parameters only.
- When the retention in AI Bot parameters is 0 too, nothing is deleted.

### Upgrade steps

1. **Switch the log on.** After the upgrade, an existing environment has the log off: Log agent activity = No, Store request text = No, Retention = 0. Either:
   - open **AI Bot parameters > Activity log** and set Yes / Yes / 365 days; or
   - run **Initialize AI Bot user setup**. It sets Yes / Yes / 365 when all three are still unset, and leaves the row alone when an administrator has changed any of them.
2. **Schedule the clean-up.** On each environment, schedule **Clean up agent activity log** as a recurring batch job, for example daily. The existing **AI Bot session cleanup** job no longer cleans the activity log.
3. **Knowledge access:** nothing to do. Existing users have no rows on the Knowledge tab, so they keep every document until an administrator fills their list.
