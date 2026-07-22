# PowerSquad — Execution surface

The concrete tools each agent uses. This is what makes the squad *build* things instead
of just describing them. Every environment-specific value below is a `${PLACEHOLDER}`
resolved from your `.env` (see `setup/SETUP.md`).

## 1. CRM Sales (client context — READ ONLY)
Your Dynamics 365 Sales tenant. Used by **Gold Ranger** to read account /
opportunity / contact context for the client.

```
copilot -p "Query CRM for opportunity <number>: account, contacts, stage, notes. Use dataverse_query / get_opportunity_details." --allow-all-tools
```

Never write to CRM from the squad. It is a source of truth about the customer, not a build target.

## 2. WorkIQ (meetings, chats, transcripts)
Used by **Gold Ranger** to understand what was actually discussed with the client.

```
${SCOUT_HOME}\.copilot\bin\workiq.cmd ask -q "Summarize my meetings and chats about <client> in the last 60 days, including any transcript highlights."
```

Also: `workiq_*` tools directly (list emails, chats, events, search people).

## 3. Demo org (THE BUILD TARGET — Dataverse read/write)
- URL: `${DEMO_ORG_URL}`
- Environment: `${ENVIRONMENT_NAME}`
- MCP: `${DEMO_MCP}`

Read (schema + data) is available directly via the demo-org MCP (`describe`, `read_query`).
Writes (create tables, columns, rows) go through the Copilot CLI, which has full tools on the org:

```
copilot -p "In the demo Dataverse org, create entity_set='...' rows / metadata via dataverse_write." --allow-all-tools
```

Used by **Black Ranger** (metadata), **Green Ranger** (data), and read by **Red Ranger / Blue Ranger**.

## 3b. Identity & authentication (CRITICAL)
**The single most important operational fact: build with the tenant's NATIVE identity.**
A demo org typically lives in a **separate tenant** (`${TENANT_ID}`) from your corporate
account (`${CORP_TENANT_ID}`). A corporate user is only a **guest** in the demo tenant, and
guest tokens repeatedly break tooling:

| Surface | As guest (`${CORP_EMAIL}`) | As native admin (`${ADMIN_UPN}`) |
|---|---|---|
| Canvas Authoring MCP `connect` | **HTTP 404** (token → home tenant) | ✅ connects (broker; retry on timeout) |
| `power-apps init` (code apps) | **"Environment not found"** | ✅ resolves the demo env |
| Browser-created app ownership | guest-owned → invisible to admin "My apps" | admin-owned, visible/manageable |
| pac / dataverse CLI (headless) | n/a (we auth these as admin) | ✅ device-code as admin |

**Auth profiles to keep straight:**
- **pac CLI** → `pac auth list` should show `${ADMIN_UPN}` active on the demo org.
- **dataverse CLI** (`dataverse auth`) → a separate profile created via **device code**
 as admin (token expires ~hourly; re-run `dataverse auth create` when it lapses).
- **power-apps CLI** → its OWN cached session (separate from pac). Fix wrong-tenant with
 `npx power-apps logout` then re-init signing in as **admin**.
- **Browser (playwright)** → reach the demo tenant via **Account manager → Switch directory →
 <demo tenant>**, approve MFA; stay on `make.powerapps.com`. The browser SSO defaults to the
 corporate account, so canvas Studio sessions are guest-owned unless you sign in as admin.

**Manual human actions this implies** (Zord's prereq gate must flag these): admin browser
sign-in, MFA approval, possibly a tenant setting toggle (guest app-creation), device-code auth.
**See `identity.md`** for the full native-admin identity strategy.

## 4. pac-mcp (Power Platform CLI via MCP)
Solution ALM, environments, app listing, publishing. Used by **Black Ranger** (create solution),
**Red Ranger**, **Yellow Ranger**, **Blue Ranger**, and Zord at packaging time.

Typical operations: create/select solution, add components, export (managed/unmanaged),
import, list environments & apps.

## 4b. CLI-native build stack — PREFERRED (Claude Code / GitHub Copilot CLI plugins)
Microsoft ships official agent plugins that let a CLI build Power Platform by natural
language. **This is the squad's primary build mechanism.**

**(a) Power Platform Skills marketplace** — `github.com/microsoft/power-platform-skills`
Installed via the installer script (`scripts/install.js`), which also sets up `pac`/Azure CLI
and registers the marketplace. Plugins:
- **power-pages** (Code Sites / SPAs: React/Angular/Vue/Astro) → **Blue Ranger**
- **code-apps-preview** (Power Apps code apps: React + Vite + TS, deployed via PAC CLI) → **Red Ranger**
- **canvas-apps** (PA YAML via Canvas Authoring MCP server; needs .NET 10 SDK) → **Red Ranger**
- **model-apps** (model-driven generative pages: React + TS + Fluent via PAC CLI) → **Red Ranger**
- **mcp-apps** (MCP-connected app scaffolding) → **Red Ranger**
Get started in a session, e.g. `/power-pages:create-site`.

**(b) Dataverse Skills** — `github.com/microsoft/Dataverse-skills` (`dataverse@awesome-copilot`)
Skills: **dv-overview** (tool routing, loaded first), **dv-connect** (one-time setup + auth +
registers the Dataverse MCP), **dv-metadata** (tables/columns/relationships/forms/views),
**dv-solution** (create/export/import/promote), **dv-data** (CRUD + bulk CSV import +
AI-generated sample data), **dv-query** (read/filter/aggregate, pandas), **dv-admin** (env
settings, bulk delete, retention, PPAC allowlist), **dv-security** (roles, users, app users, BUs).
First run: ask the agent **"Connect to Dataverse"** → dv-connect registers a `dataverse-<org>`
MCP and `pac auth` against the env.
- **Black Ranger** → dv-metadata + dv-solution.
- **Green Ranger** → dv-data ("AI-generated sample data — one prompt"; CSV import).
- **Zord** → dv-solution (create/export); dv-admin / dv-security for env setup.

**(c) eval-guide** — `github.com/microsoft/eval-guide`
Evaluates Copilot Studio agents (plan/generate/run/interpret/triage; interactive HTML
dashboards). Install for GitHub Copilot: `npx skills add microsoft/eval-guide`. → **White Ranger** (QA).

**(d) Power CAT Copilot Studio Kit** — `github.com/microsoft/Power-CAT-Copilot-Studio-Kit`
Solution-based toolkit (model-driven + code apps) to develop, govern, and test Copilot Studio
agents → **White Ranger**. Highlights: batch **Testing** (response/topic/attachment/multi-turn/generative
+ user-defined **rubrics**), **Agent Library** (pre-built agent templates — feeds the reuse
`library/`), **Component Library**, **Prompt Advisor**, **Agent Debugger**, **Agent Insights
Hub** + **Conversation KPI**, **Compliance Hub** + **Power Shield** (DLP/connector governance),
**SharePoint sync** for knowledge. Imported into the demo org as a managed solution.

Usage pattern (one prompt; the plugin orchestrates pac/MCP/SDK/Web API):
```
copilot -p "Using Dataverse Skills against the demo org, in solution '<solution>': create tables [...] with lookups, then load 50 realistic sample records." --allow-all-tools
```
Raw `dataverse_write` / demo-org MCP and maker-portal playwright remain fallbacks.

## 4c. Copilot Studio via CLI (for White Ranger)
There is a Copilot Studio plugin for Claude Code / GitHub Copilot CLI. **White Ranger**
should prefer it for agent/topic/knowledge operations when available, use **eval-guide**
for QA, and fall back to playwright on the maker portal for anything not yet covered.

## 5. playwright (maker portal — where APIs are thin)
Browser automation against the maker portals. Used where there is no clean API:
- Power Apps Studio fine-tuning — `make.powerapps.com`
- Power Automate designer — `make.powerautomate.com`
- Power Pages design studio — `make.powerpages.microsoft.com`
- Copilot Studio (topics, knowledge, publish) — `copilotstudio.microsoft.com`

## 6. filesystem + skills (deliverables)
- **pptx** skill → PowerPoint deck (script in slide notes).
- **web-artifacts-builder** skill → interactive HTML deck.
- **excalidraw** skill → architecture diagrams for the deck.
Used by **Pink Ranger**.

## Decision: solution as the unit of delivery
Every demo lives in a Dataverse **solution** named per client (e.g. `cwu_<client>`).
Build unmanaged; export managed to hand off. This keeps demos isolated, versioned, and
cleanup-able, and feeds the reuse `library/`.
