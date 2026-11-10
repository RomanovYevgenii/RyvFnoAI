# RyvSoft AI — Installation

## 1. Deploy the package

RyvSoft AI ships as a standard D365FO **deployable package** (`AXDeployablePackage_*.zip`).

**Sandbox / production (LCS or Power Platform admin center):**
1. Upload the package to the Asset library (LCS) or the environment's package deployment page.
2. Apply it to the target environment (*Maintain → Apply updates*).
3. Wait for the servicing operation to finish — database synchronization runs automatically.

**Cloud-hosted / one-box developer VM:**
1. Unpack the zip to a local folder.
2. From an elevated prompt in that folder run:
   ```
   AXUpdateInstaller.exe generate -runbookid="RyvSoftAI" -topologyfile="DefaultTopologyData.xml" -servicemodelfile="DefaultServiceModelData.xml" -runbookfile="RyvSoftAI-runbook.xml"
   AXUpdateInstaller.exe import -runbookfile="RyvSoftAI-runbook.xml"
   AXUpdateInstaller.exe execute -runbookid="RyvSoftAI"
   ```
3. Run a full database synchronization if the runbook did not, then `iisreset`.

## 2. Assign security

Add the duties to roles of your choice:

| Duty | For |
|---|---|
| AI Bot administrator duty | Administrators who configure the AI |
| AI Bot clerk duty | Users who chat with the AI and build their own workspace pages |
| View the AI workspace | Users who only open AI workspace pages |
| Maintain AI Reporting definitions | Users who design AI Reporting reports |
| Run AI Reporting reports | Users who run published AI Reporting reports |
| AI Bot external access | The F&O user an external system signs in as (see [External API](EXTERNAL_API.md)) |

## 3. Configure the provider

**RyvSoft AI → Setup → AI Bot parameters**

- Enable AI Bot = Yes
- AI Provider: Azure OpenAI / OpenAI / Anthropic Claude
- Endpoint URL, API key (stored encrypted), Model ID, Max tokens
- Azure OpenAI only: Deployment name, API version
- Click **Test Connection**

## 4. Initialize users

**RyvSoft AI → Periodic tasks → Initialize AI Bot user setup** — creates the user setup with all tools and agents.
Then review **RyvSoft AI → Setup → User setup** and narrow down tools and agents per user.

To give users the ready-made agents, import the [Agent Library](AGENT_LIBRARY.md) package through Data management first.

Run **Refresh label index** (**AI Bot parameters → Additional configuration**) once after installation and again after a deployment that adds or changes tables, enums, menu items or their labels, so the agent can find tables, fields and menu items by their labels.

## 5. Open the chat

On any page, open the **settings (gear) menu → AI Bot**. The chat opens as a side panel next to your form; pick an agent and start typing.

Optional next steps: MCP servers, Knowledge Base, custom agents, scheduled agent runs, AI Reporting, session clean-up batch job — see the [Functional guide](FUNCTIONAL_GUIDE.md).
