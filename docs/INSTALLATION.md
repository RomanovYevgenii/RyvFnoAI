# RyvFnoAI — Installation

## 1. Deploy the package

RyvFnoAI ships as a standard D365FO **deployable package** (`AXDeployablePackage_*.zip`).

**Sandbox / production (LCS or Power Platform admin center):**
1. Upload the package to the Asset library (LCS) or the environment's package deployment page.
2. Apply it to the target environment (*Maintain → Apply updates*).
3. Wait for the servicing operation to finish — database synchronization runs automatically.

**Cloud-hosted / one-box developer VM:**
1. Unpack the zip to a local folder.
2. From an elevated prompt in that folder run:
   ```
   AXUpdateInstaller.exe generate -runbookid="RyvFnoAI" -topologyfile="DefaultTopologyData.xml" -servicemodelfile="DefaultServiceModelData.xml" -runbookfile="RyvFnoAI-runbook.xml"
   AXUpdateInstaller.exe import -runbookfile="RyvFnoAI-runbook.xml"
   AXUpdateInstaller.exe execute -runbookid="RyvFnoAI"
   ```
3. Run a full database synchronization if the runbook did not, then `iisreset`.

## 2. Assign security

Give users one of the duties (via a role of your choice):

| Duty | For |
|---|---|
| `RYVAIBotAdminDuty` | Administrators who configure the AI |
| `RYVAIBotClerkDuty` | Users who chat with the AI |

## 3. Configure the provider

**System administration → Setup → RYV AI Bot → AI Bot Parameters**

- Enable AI Bot = Yes
- AI Provider: Azure OpenAI / OpenAI / Anthropic Claude
- Endpoint URL, API key (stored encrypted), Model ID, Max tokens
- Azure OpenAI only: Deployment name, API version

## 4. Initialize users

**System administration → Periodic tasks → Initialize AI Bot user setup** — creates the user setup with all tools and agents.
Then review **User Setup** and narrow down tools and agents per user.

## 5. Open the chat

On any page, open the **settings (gear) menu → AI Bot**. The chat opens as a side panel next to your form; pick an agent and start typing.

Optional next steps: MCP servers, Knowledge Base, custom agents, session clean-up batch job — see the [Functional guide](FUNCTIONAL_GUIDE.md).
