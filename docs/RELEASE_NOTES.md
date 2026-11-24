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

### Upgrade steps

1. Run a database synchronization (new fields and a new unique index on the access setup table).
2. Nothing else is required. Existing user rows keep their own lists, so behavior does not change until you add Role rows or switch a user's **Use role access** to Yes.
3. To move a team to role-based access: create a Role row for their role, fill its Tools and Agents tabs, check the **Effective access** tab of one user, then set **Use role access** = Yes on the users' own rows (or delete those rows).

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
