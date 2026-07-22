# PowerSquad — Identity & Tenant Strategy

The operational backbone of the squad. Empirically validated across all builders,
the Canvas Authoring MCP, and the `power-apps`/`pac`/`dataverse`/`az` CLIs.

## Auth cheat-sheet — the FOUR independent sessions (all as native admin)
Each tool keeps its OWN auth session. Authenticate each separately to the **demo tenant** as the
native admin (`admin@<tenant>`). They do NOT share tokens.

| Tool | Used by | Auth command (demo tenant, as admin) | Notes |
|---|---|---|---|
| **pac CLI** | Black Ranger/Green Ranger (writes), Red Ranger model-driven, Blue Ranger deploy, Zord solution | `pac auth create --environment ${DEMO_ORG_URL}` (interactive) | `pac auth list` / `pac org who` to verify. Dataverse audience only. |
| **dataverse CLI** | Dataverse Skills (metadata/data/query) | `dataverse auth create -env ${DEMO_ORG_URL} -n psq-demo --deviceCode` | Token ~1h; re-run when it lapses. Don't kill the command before it persists. |
| **power-apps CLI** | Red Ranger code apps | implicit on `npx power-apps init`; if wrong tenant → **`npx power-apps logout`** then re-init and sign in as admin | Its own cached session, separate from pac. |
| **Azure CLI (`az`)** | Blue Ranger (Power Pages): integrate-webapi, setup-datamodel, add-sample-data, **activate** (BAP), and direct Dataverse Web API calls | **`az login --tenant ${TENANT_ID} --allow-no-subscriptions --use-device-code`** | See the `az` section below — this one bit us; document carefully. |
| **Canvas Authoring MCP** | Red Ranger canvas | `connect(..., auth_flow=broker, login_hint=admin@<tenant>)`; retry on timeout | Broker user session tied to the live Studio tab. |
| **Browser (Playwright)** | any portal/Studio UI | corporate `@${CORP_DOMAIN}` (Windows SSO) + **Switch directory → demo tenant** | Only browser identity; can't switch users. |

### The `az` (Azure CLI) story — READ THIS (learned the hard way)
Power Pages (Blue Ranger) is the one builder that depends on **Azure CLI tokens**, because its
Dataverse-OData and BAP (api.powerplatform.com) operations get their bearer token from `az`.
- **The default `az` session is the CORPORATE tenant** (`${CORP_TENANT_ID}`). Against the demo org it
 fails: `power-apps init`-style "Environment not found" / `verify-dataverse-access` errors /
 activate 404. You MUST switch `az` to the demo tenant.
- **`az login` via the broker HANGS** in this environment — it printed "Select the account…" and
 never returned, and no browser window appeared. Do NOT rely on it.
- **WORKING command (use this):**
 ```
 az login --tenant ${TENANT_ID} --allow-no-subscriptions --use-device-code
 ```
 It prints a device code → user opens https://login.microsoft.com/device, enters the code, signs
 in as `admin@${ENVIRONMENT_NAME}` + MFA. Then `az` prompts to pick the subscription
 (`ME-${ENVIRONMENT_NAME}-...`) — press Enter to accept the default. This is a **MANUAL human
 step** (device code + MFA) — Zord's prereq gate must flag it.
- **Verify it worked:**
 ```
 az account show --query "{user:user.name, tenantId:tenantId}" -o json # tenantId must be ${TENANT_ID}...
 node <power-pages-plugin>/scripts/verify-dataverse-access.js ${DEMO_ORG_URL}
 ```
 The verify script returns a JSON token scoped to the demo org on success.
- **What the demo-tenant `az` token unlocks (all of Blue Ranger's remaining steps):** integrate-webapi,
 setup-datamodel, add-sample-data, **activate-site** (BAP provisioning), and direct
 `az account get-access-token --resource <org>` calls to read/write adx_* site config + site
 settings + table permissions via the Dataverse Web API.
- **`az` token != pac token:** pac's Dataverse token does NOT work for `api.powerplatform.com`
 (different audience) — that's why activate needs `az`, not pac.

## DECISION (made by the user) — hybrid, admin-first
**Use the native tenant admin (`admin@<tenant>`) for everything that can authenticate
independently of the browser; use the corporate `@${CORP_DOMAIN}` (guest) ONLY for browser
navigation, via Switch directory.**


Why this split (the real architecture):
- The **Playwright browser is bound to Windows SSO = `@${CORP_DOMAIN}`** and **cannot switch
 users**. So all browser/UI work runs as that corporate account.
- the user **added `@${CORP_DOMAIN}` as a GUEST in the demo (admin) tenant**, so **Account manager →
 Switch directory → demo tenant** works and the browser can reach the demo org env.
- **CLIs and the Canvas Authoring MCP authenticate independently of the browser** (device code /
 broker), so they sign in as the **native admin** — which is what resolves the tenant correctly.

Concrete allocation:
- **Admin (native)** → `pac`, `dataverse`, `power-apps` CLIs; Canvas Authoring MCP `connect`;
 all headless builders; resource ownership. Anything not strictly browser-bound = admin.
- **`@${CORP_DOMAIN}` guest (browser only)** → Playwright navigation in the maker portal /
 Studio, always via **Switch directory** to the demo tenant. Can't be avoided; it's the only
 browser identity available.
- **Canvas = both at once** (proven): the Studio coauthoring TAB runs as the guest browser
 session, while the **MCP `connect` authenticates as admin** and pushes into that tab; then
 **Share the app to the admin** so ownership is correct.

Implications already encoded in the squad: Red Ranger per-type prereqs, Zord's prereq gate (flags the
admin sign-ins / MFA + the browser Switch-directory step), and the guardrails in README/SKILL.

### Future optimization (not adopted yet)
For fully unattended headless runs, a **Service Principal (SPN)** can replace interactive admin
auth on `pac`/`power-apps` (`--applicationId/--clientSecret/--tenant`) — good for CI/CD and the
Dataverse builders. Canvas stays human-in-the-loop (live Studio tab). Revisit when scaling.

## (Background) Why native identity matters — evidence
Demo work happens in a **demo org inside its own tenant** (`${ENVIRONMENT_NAME}`,
tenant `${TENANT_ID}...`). A corporate `${CORP_EMAIL}` account is only a **guest** there. Guest
identities break the toolchain in three distinct ways:

1. **Canvas Authoring MCP `connect` → HTTP 404.** The MCP has no tenant parameter; broker auth
 yields a token in the guest's HOME tenant, so the authoring service can't resolve the app.
2. **`power-apps init` → "Environment not found".** Same wrong-tenant resolution.
3. **Ownership orphaning.** Apps created in the browser as a guest are guest-owned and don't
 appear in the admin's "My apps"; the admin can't manage/package them without a Share.

The **native tenant admin** (`admin@${ENVIRONMENT_NAME}`) resolves all three. Confirmed:
- Canvas MCP `connect` succeeds as admin (broker; flaky → retry, succeeds 2nd–3rd try).
- `power-apps init` resolves the demo org env after `logout` + sign-in as admin.
- Code app deployed as admin is admin-owned (a guest opening the play URL sees "Request access").
- pac/dataverse headless builders already auth as admin (device code) and work.

## Open questions still to resolve (per client/org)
- **Licensing:** does the admin have the needed Power Apps / Dataverse licenses in each demo org?
 (Native admin did here; verify per client org.)
- **Per-client portability:** each engagement may use a different the demo org/demo org with its own admin
 + tenant policies, and the corporate `@${CORP_DOMAIN}` must be added as a **guest** in that tenant
 for the browser Switch-directory to work. Re-run the identity setup per org; document org URL +
 tenant id + env id + admin UPN.
- **Guest-only fallback (degraded mode):** if only a guest is available and the corporate account
 can't be made a native admin, model-driven + code apps can still work if the CLI session is
 coaxed to the right tenant, but **canvas authoring needs admin MCP auth** and ownership needs
 post-build Shares.

## Manual actions the squad must surface (Zord's prereq gate)
For any build, flag explicitly which of these the human must do, and which are already satisfied:
- [ ] Browser: signed in as `@${CORP_DOMAIN}` + **Switch directory** to the demo tenant (the only
 browser identity; requires `@${CORP_DOMAIN}` to be a **guest** in the demo tenant).
- [ ] `pac auth` / `dataverse auth` / `power-apps` sessions on the demo org **as native admin** (re-auth if lapsed).
- [ ] Canvas MCP `connect` **as admin** (broker; expect 1–2 sign-in popups + MFA; retry on timeout).
- [ ] Tenant setting toggles when unavoidable (e.g. "guests can create canvas apps" — only if
 forced to build canvas as guest; prefer admin instead).
- [ ] Canvas: create a **blank app**, enable **Coauthoring**, **keep the Studio tab open**; Share
 to admin afterward.
- [ ] Install host prereqs: **.NET 10 SDK** (canvas MCP), **Node v22+** + **Git** (code apps).

## Reference identifiers (this environment)
- Demo tenant (Contoso): `${TENANT_ID}` (`${ENVIRONMENT_NAME}.onmicrosoft.com`)
- Corporate tenant (guest home): `${CORP_TENANT_ID}` (`microsoft.com`)
- the demo org env id: `${ENVIRONMENT_ID}` · org URL `${DEMO_ORG_URL}`
- Native admin: `${ADMIN_UPN}`
