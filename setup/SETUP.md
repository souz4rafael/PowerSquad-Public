# PowerSquad — Setup

PowerSquad builds Dynamics 365 / Power Platform demos in **your own** demo
environment. Before the squad can build anything, it needs a few details about
that environment (org URL, tenant, admin identity, etc.). You provide them
**once**; they're stored in a **git-ignored `.env`** so your tenant details
never get committed.

## 1. Prerequisites

- A **Power Platform demo environment** you have **admin** on (build target).
- The **native tenant admin** account of that environment (see `../identity.md`
 — building as a cross-tenant guest breaks the tooling).
- **Microsoft Scout** with the tools the squad uses (a Dataverse MCP server,
 Playwright, the file/shell tools). See `../execution-surface.md`.
- **PowerShell 7+** (`pwsh`) to run the setup helper.

## 2. Configure (choose one)

### Option A — interactive helper (recommended)
```pwsh
pwsh ./setup/setup-powersquad.ps1
```
It asks for each value (showing any current value as the default) and writes
`.env` at the repo root. Re-run it any time to update.

### Option B — manual
```pwsh
copy setup\.env.example .env
```
Then edit `.env` and fill in your values. Every field is documented inline.

## 3. What each variable is

| Variable | What it is | Where to find it |
| --- | --- | --- |
| `DEMO_ORG_URL` | Your demo org URL | Power Apps → Settings → Session details |
| `DEMO_ORG_HOST` / `DEMO_ORG_ID` | Host + short id | derived from the URL (the helper fills these) |
| `ENVIRONMENT_ID` | Environment GUID | Power Platform Admin Center → Environments |
| `ENVIRONMENT_NAME` | Environment display name | same place |
| `TENANT_ID` | Azure AD tenant GUID | Entra ID → Overview |
| `ADMIN_UPN` | Native admin of the demo tenant | your admin account, e.g. `admin@…onmicrosoft.com` |
| `ADMIN_USER` | Admin short name (cosmetic) | optional |
| `USER_EMAIL` | Your work email (notifications/test sends) | optional |
| `USER` / `SCOUT_HOME` | OS username + home dir (path resolution) | `C:\Users\<USER>` |
| `CORP_TENANT_ID` / `CORP_DOMAIN` / `CORP_EMAIL` | Your corporate/home tenant | only for guest sign-in scenarios (CRM Sales / content library) |
| `CRM_SALES_HOST` | CRM Sales host for opportunity context | optional |
| `CONTENT_LIBRARY_HOST` | Enablement/content library host | optional |
| `DEMO_MCP` | Name of the Dataverse MCP server registered in Scout | your Scout MCP config |

Anything marked optional can be left blank if it doesn't apply to you.

## 4. Activate

Open a session in Scout and say **"activate PowerSquad"** (or name an agent,
e.g. *"Black Ranger, add a status column"*). On activation the coordinator
(**Zord**) runs the **setup gate**: it checks that `.env` exists and that the
required values are filled. If anything is missing, it points you back here
before any build starts.

## 5. Security notes

- **`.env` is git-ignored** — your real tenant details never leave your machine.
- The committed docs/specs only contain `${PLACEHOLDER}` references — no real
 org URLs, tenant IDs, emails, or GUIDs.
- Client data is private: the squad **never** puts real customer data into
 shareable artifacts (decks, seed data, screenshots) — it synthesizes instead.
- Build only in your **demo** org. Any customer/opportunity source is
 **read-only** context, never a build target.
