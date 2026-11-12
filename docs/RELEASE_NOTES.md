# RyvSoft AI — Release Notes

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
