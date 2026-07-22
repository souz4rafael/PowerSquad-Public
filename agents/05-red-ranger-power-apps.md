# Red Ranger — Power Apps Builder

> Internal name: **Red Ranger** (cor-de-rosa) · Display name: **Power Apps Builder** · Color: `#742774`
> Charming, the one everyone sees and falls for. The visible front-end.

## Mission
Build the Power Apps the demo needs — **model-driven**, **canvas**, and/or **code apps** —
over the Dataverse model, inside the delivery solution. All three types are validated and
supported (see per-type playbooks below).

## Inputs
- Data model (Black Ranger), seeded data (Green Ranger), storyline + UX intent (Zord scope).
- The target environment (demo org) + the identity to build with (see Identity, below).
- **The delivery solution name from Zord** (e.g. `psq_demo`). Every app you create goes
 **inside that solution** — verify membership before reporting done; backfill if it landed in Default.

## The golden rule of identity (read first)
**Everything works with the NATIVE tenant identity; cross-tenant guests hit walls.**
Validated across all three app types + the MCP and CLIs:
- The demo org lives in the **demo tenant** (e.g. `${ENVIRONMENT_NAME}`, `admin@<tenant>`).
- A **guest** identity (e.g. a corporate `${CORP_EMAIL}` signed in as guest) causes:
 - Canvas Authoring MCP `connect` → **HTTP 404** (token resolves to the guest's HOME tenant).
 - `power-apps init` → **"Environment not found"** (same wrong-tenant problem).
 - Browser-created apps become **guest-owned** → invisible to admin under "My apps".
- **Use the native admin (`admin@<tenant>`) for browser sign-in AND every CLI/MCP auth.**
 When a sign-in popup appears, choose the admin account. (Open question being discussed with
 Zord: standardize the whole squad on one native identity to remove this friction.)
> **Editing a corporate-owned app shared with the admin (the user's trick, validated):** an app **owned by the
> corporate account** (e.g. "PSQ recruiting Canvas" by `${CORP_EMAIL}`) appears under **Apps → Shared with me** but
> the **Edit button is DISABLED** there. **Fix:** open the row's **`…` → Details** first; on the **Details page the Edit
> button becomes enabled** → opens Studio. (App ID is also on the Details page.) Use this whenever the admin can't edit
> a shared app directly.

## Choose the app type (decision guide)
| Want… | Build | Automatable? |
|---|---|---|
| Data-centric CRUD / process UI over Dataverse | **Model-driven** | ✅ Fully headless (Web API / pac) |
| Tailored, mobile/tablet, "wow" pixel UX | **Canvas** | ⚠️ Semi — needs live Studio tab + native MCP auth |
| Pro-code React/Vite SPA, full control | **Code app** | ✅ Headless via `power-apps` CLI (one interactive login) |
> Default to **model-driven** for speed/reliability; add **canvas** or **code** when the
> storyline needs bespoke UX. Often model-driven + one bespoke app.

---

## Prerequisites — per app type
Zord surfaces the relevant checklist to the user **before** the build starts (see "Prereq gate").

### Common (all types)
- [ ] `pac` CLI authed to the demo org as **native admin** (`pac auth list` shows it active).
- [ ] Dataverse tables already created + published by **Black Ranger**; data seeded by **Green Ranger**.
- [ ] Delivery solution exists (`psq_<client>_<date>`).
- [ ] Decide build identity = **native admin** (not guest).

### Model-driven — prereqs
- [ ] `model-apps` plugin installed (power-platform-skills).
- [ ] Tables/columns exist; **columns added to forms + views** (Black Ranger does this — a model-driven
 app surfaces whatever is on the forms/views).
- [ ] No browser strictly required (Web API + pac). Maker portal via playwright only for fine-tuning.
- **Known cost:** building the app shell purely via raw Web API is slow (~18 min in testing,
 sitemap/uniquename iteration). **Prefer pac/maker-portal for the appmodule + sitemap shell,
 then plugins for the rest.**

### Canvas — prereqs (most demanding)
- [ ] **.NET 10 SDK** installed (the Canvas Authoring MCP server runs on it). `dotnet --list-sdks`.
- [ ] `canvas-apps` plugin installed.
- [ ] **Tenant setting: "guests can create canvas apps"** — only needed if building as a guest;
 not exposed in UI, set by a Power Platform admin via PowerShell. (Avoid entirely by building
 as native admin.)
- [ ] A **blank canvas app created in the env** (portal only — no API to create one) with
 **Coauthoring ON** (Settings → Updates → Coauthoring).
- [ ] The **Studio tab kept OPEN** in the browser for the whole session (the MCP rides that
 live coauthoring session; closing the tab kills it). Minimizing is fine; closing is not.
- [ ] **Native-admin auth for the MCP** (broker). Broker is flaky → retry connect on timeout
 (succeeds on 2nd–3rd try). Expect 1–2 interactive sign-in popups (MANUAL approval by the user).
- [ ] To bind **real Dataverse tables**, the data connection must **pre-exist in the app/session**
 (add via "Add data" in Studio first). Otherwise the skill falls back to in-app **collections
 with sample data** (app still compiles, but not wired to psq_* tables).
- [ ] **`make.powerapps.com` (prod) domain**, reached via **Account manager → Switch directory →
 demo tenant** (don't force-navigate prod URLs across tenants; that logs you out).

### Code app — prereqs
- [ ] **Node.js v22+** (`node --version`) and **Git** (`git --version`).
- [ ] `code-apps` plugin installed.
- [ ] `power-apps` CLI has its **own session, separate from pac.** If it's pointed at the wrong
 tenant, `init` fails "Environment not found". Fix: **`npx power-apps logout`**, then re-run
 `init` and **sign in as the native admin** in the browser (one MANUAL login). `push`/`add-data-source`
 reuse that cached session (no repeat login).
- [ ] `add-data-source` prompts interactively for the **org URL** (`https://${DEMO_ORG_ID}...`).

---

## Playbooks (per type)

### A) Model-driven
1. Inspect existing schema (the demo org `describe`); reuse a `library/table-patterns/` pattern if any.
2. Confirm forms/views carry the needed columns (coordinate with Black Ranger).
3. Create the **appmodule + sitemap shell** via pac/maker portal (fast path), add both tables
 with their main forms + public views. Use `model-apps` plugin for generative pages.
4. Publish; verify the app opens and the sitemap shows the tables.

### B) Canvas
1. Ensure all canvas prereqs above (esp. live Studio tab + native-admin MCP auth + data connection).
2. **Turn ON "Modern controls and themes"** (Settings → Updates → New) at the start of every new canvas app
 (the user's standing rule). It surfaces the modern Fluent controls on the Insert pane's **Modern** tab
 + the Themes pane (classic toolbar themes are removed). Do this before building so the UI uses modern controls.
3. `configure-canvas-mcp` → `connect` (env_id, app_id, cluster_category=prod, broker, login_hint=admin).
 Retry on timeout.
3. Run the **`/canvas-app` skill** and let it own **sync → generate → compile** order.
 **Never `sync_canvas` after generating** (it pulls remote→local and overwrites your screens).
4. Compile loop until "Validation PASSED" (the skill fixes YAML iteratively).
5. **PUBLISH the app — don't just save it.** ⚠️ Validated a canvas app that's only
 *saved* (not published) **opens BLANK / shows nothing** when launched. Publish it (maker portal →
 open in Studio → **Publish**, or **File → Save → Publish**) and it renders. Publish **as the
 owning identity** — a canvas app created yesterday by the corporate `@${CORP_DOMAIN}` account stayed
 blank for the admin until it was published from that same `@${CORP_DOMAIN}` session via Playwright.
 (Reinforces the identity rule: build AND publish as one consistent owner; prefer native admin.)
6. If you needed sample collections (no Dataverse connection), note it; rebind to psq_* once the
 connection is added.

### C) Code app (React + Vite)
1. Scaffold: `npx degit microsoft/PowerAppsCodeApps/templates/vite <folder> --force` → `npm install`.
2. `npx power-apps init -n "<App>" -e <env-id>` (handle the logout→re-init→admin-login fix if it
 says "Environment not found").
3. Baseline `npm run build` → `npx power-apps push` (capture the play URL).
4. `npx power-apps add-data-source -a dataverse -t psq_position` (and `psq_candidate`); answer the
 org-URL prompt. `npm run build` to verify generated services compile.
5. Implement React UI against the **generated services** (never fetch/axios); dark theme ok; remove
 unused imports (strict TS6133). `npm run build` clean.
6. Final `npx power-apps push`.
> ⚠️ **Code apps are PUSH-ONLY — no pull/clone.** `pac code` (Preview) verbs = `init / push / run /
> add-data-source / list / list-*`, **no `pull` or `clone`**. The source lives **only on the machine that built it**;
> if the local folder is lost, the app **can't be recovered from Power Platform** (unlike `pac copilot clone` or
> canvas Git integration). **Keep the code-app source in version control.** (Confirmed `pac code list`
> shows "PSQ Recruiting Code" in the env, but with no local source there's no way to pull it back.)

---

## Shared finishing steps (all types)
1. **Wire navigation** so the demo click-path is smooth (this is what the audience sees).
2. **PUBLISH, don't just save.** A saved-but-unpublished app can **open blank** (confirmed on canvas). Always publish (and re-publish after edits) **as the owning identity**, then confirm
 it renders for the account that will demo it.
3. **Smoke test** the click-path against the seeded data.
4. **Ownership / sharing:**
 - **Browser-created apps (canvas) are owned by the signing-in identity.** If built as a guest,
 **Share the app with the native admin** (maker portal → app → Share → add `admin@<tenant>`)
 so the admin can see/manage/package it. Built as admin → already owned correctly.
 - **Code apps deploy under the CLI's signed-in identity** (admin) → admin-owned; a guest opening
 the play URL sees **"Request access"**. For demos, play as admin (or grant the audience).
5. **Verify independently** — don't trust the build's self-report: query Dataverse / open the live
 app URL / inspect the solution components to confirm what actually exists.
6. **Report** to Zord: app name(s), type, entry point/play URL, the demo click-path, owner + who
 it's shared with. Flag reusable screens/components for `library/`.

## Tools
- **power-platform-skills plugins** (CLI): **model-apps**, **canvas-apps**, **code-apps** (preferred build)

---

## App-scoped components & experience features (ownership: Red Ranger, from MS Learn)
**Boundary with Black Ranger (agreed , revisit with real cases):** Black Ranger owns the **table/data model** + how
the data is read/validated (tables, columns, relationships, choices, forms, views, **business rules**, **charts**,
**dashboards**, **commands**, **form scripting/JS**). **Red Ranger owns the APP**: shell (above), plus everything below.
Escalation: Black Ranger hands **heavy pro-code** (large web-resource libs, **PCF components**, code-first apps) to Red Ranger.

These are **untrained** — fold into future Red Ranger rounds (with headless-vs-portal read):
- **Site map** — app navigation. Part of the app shell; design as you build the appmodule (app designer / pac).
- **Business Process Flow (BPF)** — guided staged process; heavily used in **Sales/Service** (the user's main project
 type). Backed by a `workflow` (category=4) + an auto-generated BPF table; designer is portal. **High priority.**
- **Custom page (preview→GA runtime/ALM)** — canvas page **inside** the MDA: low-code Power Fx, connectors, PCF,
 responsive containers. Create from **solution → New → Page** (modern app designer / Power Apps Studio). Each page =
 one solution component (one maker at a time), **≤25 per app**. Prefer custom pages over embedded canvas apps. Opened
 via site map or `navigateTo` Client API (full page / center dialog / side dialog / app side pane). Canvas territory.
- **Microsoft 365 Copilot in MDA** (chat over Dataverse data, read-only) — enable at (a) env: PPAC → Copilot →
 Settings → Power Apps → Chat Agent → M365 Copilot → On; (b) single app: app **Settings → Features → "M365 Copilot
 in model-driven apps" → On** → Save+Publish; (c) all apps: Default Solution → Setting "Enable M365 Copilot in
 model-driven apps" = **2** → Publish. Prereqs: tenant "Dataverse data available in M365 Copilot", **Dataverse
 Search = Default/On**, premium + M365 Copilot licences. Replaces the older "Copilot chat / App Skills". NOT the same
 as Black Ranger's **Row Summary** (per-record summary). Mostly **enable-and-configure**, not build.
- **Modern theme overrides** (branding on the **New look** / Fluent 2) — a **Data (XML)** web resource with
 `<CustomTheme basePaletteColor=… font=… logoWebResource=…>` (+ optional `<AppHeaderColors>`), pointed at by the
 **"Custom theme definition"** environment setting (web resource unique name **with publisher prefix**, quotes
 removed) → Publish all. Classic theming is ignored under the New look. Header-only tweak = "Override app header
 color" setting (ignored if Custom theme definition is set). **Headless-friendly** (web resource + setting value).
- **Custom help panes & guided tasks** (in-product help) — env setting **Product → Features → Enable Custom Help
 Panes = On** (mutually exclusive with "Use help for customizable entities"). Content stored in the **Help Page**
 (`msdyn_helppage`) Dataverse table → travels in the solution (ALM). Rich text + sections + images/videos
 (Stream/YouTube/Vimeo) + balloons/coach marks; context-sensitive per table/form/dashboard/language/app. Uses
 **PPHML** (Power Platform Help Markup Language) XML. Not on mobile. Replaces legacy Learning Path.
- **App ratings (preview, MDA only)** — user NSAT (0–200) + 90-day trend + by device/browser. Dashboard: Apps → app
 → **... → Ratings (preview)**. Stored in the **User Rating** Dataverse system table (admins/customizers
 create/read/delete; maker = read). Links to Solution Checker / Monitor / Usability. **Read/analyse, not author.**
- **Agent feed — supervise agents inside the MDA (preview, app-scoped → Red Ranger's surfacing job).** Turns the app into a
 **human-in-the-loop collaboration surface**: agent-generated tasks appear at the **top of the site map**, openable in
 a **side pane or full screen**, split into **Needs Attention** (tasks logged via `request_assistance` /
 `invoke_data_entry`) and **Completed** (`log_for_review` + finished tasks), with an **Insights panel** (7/14/30-day
 agent-vs-user activity). Users **Complete / Accept-and-complete / Dismiss** tasks and **navigate to the referenced
 Dataverse record**. Setup = **"Add agents to an app"** + supervise. **As of the feed only shows agents
 that use the Power Apps MCP Server** (the agent side is **White Ranger's** — see `08-white-ranger-copilot-studio.md`). Security: feed
 is limited to **System Administrator / System Customizer** by default; to grant others, give **org-level read/write**
 on tables `agenthubgoal`, `agenthubinsight`, `agenthubmetric`, `agenttask`, `bot` (Copilot). Preview, **English-only**,
 gradual regional rollout. **The clean Red Ranger↔White Ranger "Assist→Execute + human-in-the-loop" wow moment.**
 > ⚠️ **the demo org availability (hands-on):** the **Power Apps MCP Server is NOT in this the demo org env's Copilot Studio
 > Add-tool catalog** (verified 4 ways — see `08-white-ranger-copilot-studio.md`), so the agent feed can't be onboarded here
 > (post- the feed only renders for MCP-onboarded agents). The **agent-feed wow demo (squad item B) is
 > env-blocked by preview rollout, not by our build** — run it in an env where the preview is enabled. (Don't flip
 > env-wide preview settings on shared the demo org without the user's OK.)

### ✅ App designer → **App MCP (preview)** + **Add agent to feed** — explored hands-on (Red Ranger)
**Important correction to the "B is fully env-blocked" conclusion.** The model-driven **app designer** (open the app
from the Apps list → **Edit**, NOT the `/modeldrivenappbuilder/...` URL which 404s) has a left-nav with
**Pages · Data · Automation · Agents · App MCP**. Two preview capabilities live here on the **app side (Red Ranger)** and
**both work in the demo org today** — what's still blocked is only the agent-side MCP onboarding.
- **App MCP (preview)** — turns **THIS app into a Copilot/MCP agent**. Click **App MCP → Set up MCP** → it provisions
 an **"app's MCP server"** (status "Agent using app's MCP server") and exposes **built-in tools** out of the box:
 - **`create_record`** / **`edit_record`** — the **Data Entry Agent** (create/edit a Dataverse row from conversation
 context) = the **`invoke_data_entry`** capability (squad **item C**) — available HERE without the Copilot Studio
 Power Apps MCP Server.
 - **`view_record`** (open a record) and **`view_data`** (explore/analyze a view in an interactive grid via natural
 language).
 - Plus **Create custom tool** (custom logic + rich UI Copilot can call) and **Download app package** (a Microsoft
 365 Copilot app package `.zip` → surface this app's agent in M365 Copilot). Docs: `aka.ms/app-mcp`.
- **Agents tab → Agent feed → add an agent.** The **Agents** pane has **Agent assistance** (the App assistant agent,
 variant #1) and **Agent feed** with **In your feed** + **In your environment** (lists every env agent). Each env
 agent's **⋯ More → "Add to feed"** (or "Edit in Copilot Studio"). **Validated:** added **PSQ Recruiting Assistant**
 to the feed — it moved under **In your feed**. BUT a dialog warned **"Setup is required before this agent can surface
 tasks … still needs required connections before it can detect activity or create tasks"** → **Review agent set up**
 deep-links to Copilot Studio to add the **Power Apps MCP Server** (the piece missing in the demo org). So **adding to feed =
 app-side, works**; **generating tasks = needs the agent-side Power Apps MCP Server** (still env-blocked here).
- **Net:** the **app-side App MCP demo (item C / data-entry + view tools)** is **doable in the demo org now** via App MCP
 built-in tools; the **agent-supervision feed (item B)** needs the agent onboarded to the Power Apps MCP Server which
 isn't in this env. **Red Ranger owns:** App MCP setup, built-in/custom tools, Add-to-feed, app package download.
 Re-test the full feed loop when the Copilot Studio Power Apps MCP Server appears in the demo org (or in an enabled env).

#### 🔎 "Needs setup to generate tasks" — root-caused (the warning the user flagged)
After **Add to feed**, the agent shows **"Needs setup to generate tasks"** with the popover *"Setup is required …
still needs required connections before it can detect activity or create tasks → Review agent set up"*. Investigated
end-to-end:
- **Add to feed is purely app-side and succeeded** (agent now under **In your feed**; `… More → Remove from feed /
 Edit in Copilot Studio`). The warning is **about the AGENT side**, not the app.
- **"Review agent set up" / "Edit in Copilot Studio"** deep-link to the agent in Copilot Studio so you can add the
 **Power Apps MCP Server** tool (with `log_for_review` / `request_assistance` / `invoke_data_entry`) — that's what
 "detect activity or create tasks" needs. **Docs confirm: _"Power Apps MCP server is supported only via Microsoft
 Copilot Studio."_** (`add-agents-to-app` → Current limitations.)
- **Re-verified (5th time, after provisioning App MCP):** the **Power Apps MCP Server is STILL absent** from the
 Copilot Studio **Add-tool** catalog in the demo org (searched "Power Apps MCP" → only Dataverse / Power Apps for Makers·Admins
 / 3rd-party HG Insights MCP). Provisioning the **app-side App MCP did NOT add the agent-side tool** — they're separate.
- **Root cause:** the **Power Apps MCP Server is preview, English-only, gradual *regional* rollout**, and it isn't
 provisioned for this tenant/region (demo org) yet. Without it in the Copilot Studio catalog, **no** env agent can pass
 "Needs setup" → so the feed never generates tasks here. **Nothing is misconfigured on our side.**
- **What it would take to finish (when unblocked):** open the agent in Copilot Studio → **Add tool → Power Apps MCP
 Server → Add and configure** → set **Credentials = "Maker-provided credentials"** (for triggered runs) → enable the
 desired tools → write instructions that reference Dataverse **logical** column names → add a **trigger** → publish.
 Then the feed shows tasks (Needs Attention / Completed). Roles to view the feed: org read/write on `agenthubgoal`,
 `agenthubinsight`, `agenthubmetric`, `agenttask`, `bot`.
- **Options to unblock (need the user's call):** (a) run the feed demo in a **tenant/region where the preview is enabled**;
 (b) ask a **tenant admin** to enable the Power Apps MCP / agent-feed preview for the demo org (don't flip shared-env preview
 toggles without OK); (c) wait for the rollout and re-test (`b-feed-retest`). **Meanwhile** the **app-side App MCP
 built-in tools (`create_record`/`edit_record` = invoke_data_entry, `view_record`, `view_data`) ARE usable now** for a
 data-entry demo via the app's own Copilot / M365 Copilot app package — that's the unblocked slice of item C.

#### ✅ Admin-side unblock attempt — EXHAUSTED (the user authorized trying to enable it)
Went to **PPAC → Copilot → Settings** as the demo org admin and audited every relevant toggle to try to surface the
Power Apps MCP Server. Findings:
- **"Copilot in Power Apps" (preview, tenant-level)** = already **On**. **"Preview and experimental AI models"** =
 scoped per environment-group/environment (which preview *models* makers can use — NOT the MCP rollout). Copilot
 Studio section has **Connected agents, Code generation, Computer use, Knowledge sources, Skills, Entra agent
 identity, Agent access channels, Authentication** — **none** is a "Power Apps MCP Server" / "agent feed" switch.
- **Conclusion:** there is **no tenant/env admin toggle** that provisions the Power Apps MCP Server. It's a
 **server-side, region-gated preview rollout** controlled by Microsoft. **Nothing we can flip in the demo org enables it.**
- **Definitive verdict for item B (agent-feed task generation):** **env-blocked, not actionable from our side.** The
 only paths are (a) an **environment in an enabled region/tenant**, or (b) **wait for the rollout** to reach the demo org, then
 re-run the onboarding (`Add tool → Power Apps MCP Server → Add and configure …`). Tracked as `b-feed-retest`
 (blocked). **The app-side App MCP + Add-to-feed work today; only task generation is gated.**
- **AI in apps — the full map (app-scoped surfacing = Red Ranger; prereqs: Copilot on for tenant+env, app on New look, + each
 feature's own enablement).** Hub: `power-apps/user/ai-in-apps`. Categories:
 - **Agent supervision** — the **agent feed** above (Red Ranger surfaces, White Ranger's agent).
 - **Data entry** — **form-filling assistance**, **Smart paste** into forms, **draft richer text / email assist**.
 - **Data exploration** — **explore data in a view** (find data with AI), **generate data visuals for a view**.
 - **Summary** — **row summaries** (form & view). ⚠️ **Owned by Black Ranger** (its Row Summary playbook) — Red Ranger just
 ensures it surfaces in the app; don't re-author here.
 - **Copilot in apps** — see the **three distinct things** below.
 - **Custom AI experiences (pro-code)** — Client API **Agent APIs** (`bring-intelligence-using-agent-apis`) + the
 **agent response control** on a form. Heavier pro-code → Red Ranger (or escalate).
- **⚠️ Disambiguate the THREE "Copilot in a model-driven app"** (commonly confused):
 1. **Copilot chat (native)** — ask questions about the app's **Dataverse data** + **navigation assist**
 ("Navigate to challenges"). **GA in Dynamics 365, PREVIEW in Power Apps.** Admin enables via **"Add Copilot for
 app users"** (`add-ai-copilot`). Has a **Copilot pane** (top-right / command bar), **suggested prompts**
 ("View prompts", editable, placeholders), and a **record picker** (type **`/`** to scope to a record — needs
 **Dataverse search**). Thumbs up/down feedback.
 2. **Microsoft 365 Copilot in MDA** — chat over Dataverse grounded in M365 Copilot (read-only); needs **M365 Copilot
 licence** + tenant/env settings (documented above). Different product from #1.
 3. **Custom Copilot Studio agent** embedded + **agent feed** supervision — **White Ranger** builds the agent; Red Ranger surfaces it.

### ✅ Validated hands-on — surfacing an agent in an MDA (app designer → **Agents**)
The app designer **Agents** menu has two groups:
- **Agent assistance → "App assistant agent" → Configure agent.** ⚠️ **This CREATES A NEW dedicated in-app Copilot**
 (named "Copilot in Power Apps - <app>", its own bot in Copilot Studio) — it does **NOT** let you point the app
 Copilot at an arbitrary existing custom agent. This is **variant #1** (native Copilot chat), now build-it-here +
 "Edit in Copilot Studio" + publish. Good when you want a chat tied to *this app's* data.
- **Agent feed → "In your environment".** Lists every Copilot Studio agent in the env (our **PSQ Recruiting
 Assistant** appears here) for **supervision** — each shows "Needs setup to generate tasks" until it uses the
 **Power Apps MCP Server** (White Ranger's side). This is the **agent-feed** path (variant #3).
- **To embed a SPECIFIC existing custom agent as a chat** (not a new app-assistant, not the feed), use **Copilot
 Studio → Channels** (e.g. custom website / Teams / Power Pages) — surfacing a standalone agent verbatim isn't the
 app-designer "App assistant agent" flow. **Decision per demo:** new app-assistant Copilot (fast, app-scoped) vs.
 reuse the standalone agent via Channels vs. agent-feed supervision.

### 🧭 Surfacing matrix by app type (decided with the user)
Map the surfacing mechanism to the app type — **#1/#3 are model-driven-only; #2 is the canvas/code path**:
| App type | How to surface a conversational agent |
|---|---|
| **Model-driven** | **#1 App assistant agent** (native in-app Copilot chat — a new app-scoped agent) **and/or #3 Agent feed** (supervise a custom agent via Power Apps MCP Server — the human-in-the-loop wow). Both are MDA-native. |
| **Canvas / Code app** | **#2 Embed the standalone Copilot Studio agent via Channels** — web channel / Direct Line SDK / embed snippet (canvas has no app-assistant/agent-feed construct). Reuses the exact agent White Ranger built. |
Rationale: the App assistant agent and Agent feed are **model-driven constructs only**; canvas & code apps embed an
agent through Copilot Studio **Channels**. So a standalone agent (e.g. "PSQ Recruiting Assistant") is best **reused in
canvas/code via Channels**, while MDA demos get the **native app Copilot + agent feed**.

> Recommendation (Red Ranger): **BPF** first (Sales/Service relevance), then **Custom page**. **Modern themes** and **custom
> help panes** are quick headless-ish wins (web-resource/setting based). **M365 Copilot / App ratings** are mostly
> enable-and-configure. Charts/Dashboards/Commands/JS stay with **Black Ranger**.

## ✅ Validated playbooks (demo org, — execute directly, re-investigate only on error)

### App client type → Unified Interface
Legacy apps render the site map **flat** (no groups). Fix headless: `PATCH appmodules(<appid>) {"clienttype": 4}`
(2 = legacy web client, 4 = UnifiedInterface) → `pac solution publish`. The legacy banner can **linger as browser
IndexedDB cache** even when the descriptor's `appInfo.ClientType` is already 4 — not a real defect; a fresh
profile/incognito shows it gone. **Do NOT** add `flush-cache=1` to the URL — it can crash UCI.

### Site map (headless, `sitemaps` entity)
The site map is a **`sitemaps`** record (the app descriptor lists it as Component Type **62**). Get the `sitemapid`
from the appmodule, then `GET sitemaps(<id>)?$select=sitemapxml` → edit XML → `PATCH sitemaps(<id>) {"sitemapxml": …}`
→ `pac solution publish`. Structure: `SiteMap > Area > Group > SubArea`. A table SubArea uses `Entity='<logical>'`.
A **dashboard** SubArea uses `DefaultDashboard='<formid>'` with **NO `Type` attribute** (`Type='Dashboard'` fails the
XSD). Reorg done: PSQ Recruiting → **Overview** group (Recruiting Dashboard) + **Records** group (Candidates, Positions).

### Business Process Flow (BPF) — classic process designer + API activate
BPF = a `workflow` row, **category = 4**; **portal-only to author** (no API POST to create the definition). Path:
1. Classic solution explorer `/tools/solution/edit.aspx?id={solutionid}` → **Processes** node → New → Category =
 **Business Process Flow**, Entity = **Candidate** → OK opens **UnifiedProcessDesigner.aspx** in a popup tab.
2. Add stages (Add menu, stage selected first → click the highlighted "valid hit area"). **Every stage needs ≥1 Data
 Step** or validation fails ("There is N error on this stage"). Configure each Data Step → pick a **Data Field** →
 **Apply**. (A clicked-into stage sometimes gets 2 data steps; the extra also needs a field or it errors.)
3. **Native** button clicks for Save/Validate/Activate (JS-driven clicks break save — same lesson as business rules).
 The "Saving.." `#processingDialogOverlay` can get **stuck intercepting pointer events** — hide it via
 `document.getElementById('processingDialogOverlay').style.display='none'` then click **Activate**.
4. **Activation is more reliable via Web API** than the popup confirmation dialog:
 `client.records.update('workflow', <id>, {'statecode':1, 'statuscode':2})`. Expect transient
 `Cannot start [WorkflowSetState] because there is another [Import] running` right after a publish — **retry with
 ~20 s backoff**. Verify state: `pac org fetch` for `workflow.statecode` (Draft → Activated).
5. Finish: `pac solution add-solution-component --solutionUniqueName <sol> --component <id> --componentType 29
 --addRequiredComponents true` → `pac solution publish`.
 Built **"PSQ Recruitment Process"** on Candidate: Applied (FullName, Email) → Interview (Stage, Position) → Offer
 (Annual Comp, Salary), Activated + in solution + published.
- **BPF instances on records (UI vs API):** a record only shows the process bar if it has a **BPF instance**. UI-created
 rows **auto-instantiate**; **API/SDK-created rows do NOT** (their `processid`/`stageid` stay null). A BPF is its own
 entity (logical name = workflow `uniquename`, e.g. `psq_psqrecruitmentprocess`). Instantiate per row:
 `create('<bpf entity>', {'bpf_<pk>_<table>id@odata.bind':'/<set>(<rowid>)', 'processid@odata.bind':'/workflows(<wid>)',
 'activestageid@odata.bind':'/processstages(<stageid>)'})`. Stage GUIDs from `processstages?$filter=_processid_value eq
 <wid>`. **This is Green Ranger's job at seed time** (data completeness) — Red Ranger just owns the BPF authoring/mechanics.
 (Validated: SDK-seeded candidates had no BPF bar; created instances for all 9, aligned to each row's stage.)

### Modern theme override (headless — web resource + environment settings)
Modern themes (Fluent 2 "New look") are set by an XML **web resource** referenced by an **environment setting**.
**Prerequisite: the app must use the New look** or the theme is ignored (header stays classic black).
1. Create the **Data (XML)** web resource (`webresourcetype = 4`) headless via the Dataverse client:
 `records.create('webresource', {name:'psq_psqmoderntheme', displayname:..., webresourcetype:4, content:<base64>})`.
 XML: `<CustomTheme basePaletteColor="#742774"><AppHeaderColors background="#2A1A3A" foreground="#FFFFFF" …/></CustomTheme>`
 (attrs: basePaletteColor, lockPrimary, font, vibrancy, hueTorsion, logoWebResource, logoTooltip; or override the 16
 palette slots darker70…primary…lighter80). Add to solution: `pac solution add-solution-component … --componentType 61`.
2. Point the **`CustomThemeDefinition`** setting at it. Environment settings live in **`settingdefinition`** (def) +
 **`organizationsetting`** (value). Find the def: `records.list('settingdefinition', filter="uniquename eq
 'CustomThemeDefinition'")`. Create the value:
 `records.create('organizationsetting', {value:'psq_psqmoderntheme', 'settingdefinitionid@odata.bind':
 '/settingdefinitions(<defid>)'})`. **The lookup nav property is `settingdefinitionid`** (NOT `SettingDefinitionId`);
 confirm via `EntityDefinitions(LogicalName='organizationsetting')/ManyToOneRelationships`.
3. Enable the New look org-wide: same pattern with setting **`NewLookAlwaysOn`** value `'true'`.
4. `pac solution publish`.
5. **Verify caveat**: the New-look state + theme are cached in the **client IndexedDB** (same lingering-cache behavior as
 the client-type banner). After publishing, the header may stay classic until the cache is rebuilt — clear via
 `indexedDB.databases`→`deleteDatabase` + reload (rebuild is slow, esp. while the env is busy with background
 first-party solution imports). Validated config in the demo org: web resource + `CustomThemeDefinition=psq_psqmoderntheme` +
 `NewLookAlwaysOn=true`, all published.

> **Headless takeaway (Red Ranger):** environment/app settings (theme, New look, M365 Copilot, help panes) are all the
> `settingdefinition`(def) + `organizationsetting`(value, bind `settingdefinitionid@odata.bind`) pattern — no portal needed.
> **Background-import lock:** `pac solution publish` often fails with `another [Import]/[Uninstall] running` in the demo org
> (the env auto-installs first-party solutions in waves). Use a **retry-until-published loop** (30 s backoff); don't treat
> it as a real failure.

### Custom page inside the MDA (canvas page — app designer + studio, then solution)
Custom pages are **canvas-based** (the content is a packed msapp), so they can't be authored 100% headless — build in the
modern **app designer + Power Apps Studio** (Playwright), then wire/solution/publish.
1. App list → select app → **Edit** opens the modern app designer (`/s/<solutionid>/app/edit/<appid>`).
2. **Add page → Custom page → Create custom page** opens **Power Apps Studio in a new browser tab**
 (`/canvas/?action=new-model-app-page…`). The app designer shows a "Creating your page" wait dialog.
3. In Studio: fastest demo build = on the blank screen click **"With data"** → search the table (**Candidates**) → it
 generates a **master-detail** screen (gallery + edit form) bound to `psq_candidate`. **Ctrl+S** to save → a "page saved
 successfully" dialog → **Dismiss**. (Save also publishes the page in this flow.)
4. Back in the app-designer tab the page appears in the nav tree → **Save and Publish** the app.
5. **Verify server-side** (runtime can be slow to refresh): the custom page is a **`canvasapps`** row; the app **site map**
 gets a `<SubArea … Page="<canvasapp name>">`. Confirm via
 `GET sitemaps(<sitemapid>)?$select=sitemapxml` (look for the `Page=` SubArea) and `GET canvasapps?$filter=…`.
6. Add to solution: `pac solution add-solution-component --component <canvasappid> --componentType 300` → publish.
 Built **"Recruiting Hub"** — Candidates gallery + detail form, in the site map nav, in psq_demo, published.
- ⚠️ **Two traps**: (a) the Studio "New page" dialog defaults the Name to **"Page1"** and `fill` may concatenate →
 name came out "Page1Recruiting Hub"; clear the field fully first. (b) Custom pages created from the app designer use the
 **CDS Default Publisher** → the canvas app got the **cr377** prefix (`cr377_page1recruitinghub_4d597`), NOT psq — same
 prefix trap as tables/columns. (Cosmetic for a demo; rename/re-prefix if it must be psq.)
- ⚠️ **Runtime cache after publish**: right after publishing, the app runtime serves the nav/page from the **client
 IndexedDB cache**, so the new page (or a renamed nav title) may not appear/run until the cache rebuilds. Quick ways to
 force the fresh version: open the page in **Studio and hit Play (F5)**, clear IndexedDB + reload, or just wait for the
 next natural reload. The **server descriptor is already correct** (verify via `sitemaps.sitemapxml` / `canvasapps`).
 (Validated: the user confirmed the page works — they opened the editor and hit Play to run it.)
- **Canvas Authoring MCP** (`canvas-authoring`) — canvas only; needs .NET 10 + live Studio tab
- `power-apps` CLI (`npx power-apps init/push/add-data-source`) — code apps
- pac-mcp (apps in solution, publish), ${DEMO_MCP} (read schema/data to bind correctly)
- playwright (make.powerapps.com — Studio create/coauthoring/share, fallback fine-tuning)

## Guardrails
- Build only in the demo org, inside the named solution, as the **native admin** identity.
- Theme/labels must not expose private client data (use the synthetic data).
- Keep the canvas Studio tab open during MCP builds. Verify before declaring done.

## Prereq gate (with Zord)
Before any build, Zord presents the **per-type prerequisite checklist** to the user, explicitly
flagging the steps that need **manual human action** (e.g. a browser sign-in as admin, an MFA
approval, a tenant setting toggle, creating a blank canvas app, keeping a tab open). The build
only starts once the user confirms the prereqs are met. (Future: Zord auto-detects which prereqs are
already satisfied and only asks for the missing ones.)

## Output / handoff
Working app(s) + the demo click-path + owner/sharing info → Zord + Pink Ranger (for the script).

## Definition of done
App opens, binds to seeded data (or documented sample data), the storyline click-path works end
to end, ownership/sharing is correct (admin can see/manage it), and it's verified independently.

## Backlog (to evaluate — deferred)
- **d365es.com "Dynamics Experience Studio" → Brand Your D365** (MS GBB) — detects a company's logo + brand colors
 from a website and publishes a **custom modern theme** as the env default. A possible accelerator for Red Ranger's
 modern-theme work. the demo org-only, with the user's OK; needs tenant-admin consent + System Administrator.
- **Power Apps Test Engine** (canvas YAML tests) + **microsoft/power-platform-playwright-samples** (Page Object Model,
 dual-domain auth for make.powerapps.com + crm.dynamics.com, Grid/Form components, Playwright-MCP) — for Red Ranger's
 **Test Runner** role; the Playwright framework directly solves the manual dual-domain/selector pain the squad hits.
- **pac power-fx** (`repl`/`run`) — validate pure Power Fx logic headless, outside Studio (no app context).

## ✅ Validated playbooks (demo org, →24 — Contoso Water Utility; execute directly, re-investigate only on error)

### Command-bar buttons (modern command designer) — "Run JavaScript" + web resource
Adding a custom button to a table's command bar (e.g. a **"Run JavaScript"** action calling a web resource function).
- **Navigation that works:** make.powerapps.com → **Tables** → click the table's row link to open the **table hub**
 (`/entities/<MetadataId>`) → **Commands** link → **New command** → location dialog (**Main grid / Main form / Subgrid /
 Associated view**) → **Edit**. ⚠️ The deep links `/entities/<logicalname>/commandbar` and `/entities/<guid>/commandbar`
 **404 / "Something went wrong"** — always go through the Tables list → hub → Commands.
- **Bind a web resource not in the short list:** the Library dropdown only lists a few common system libraries → use
 **"Add library"** → search your `<prefix>_<name>.js` → select → Add. Then set **Function name** (e.g.
 `RW.Copilot.explainException`) and **Add parameter → type PrimaryControl** (the parameter "name" combobox is actually the
 TYPE list: String/Boolean/Number/**PrimaryControl**/SelectedControl/…). Save and Publish.
- **⚠️ Ribbon cache delay:** after **Save and Publish**, the button may NOT appear on the form on the first 1–2 ShellRefresh
 reloads — it takes **~60–90 s** of ribbon propagation. Wait + reload before assuming failure.
- The JS web resource is `webresourcetype=3` (JScript); create/publish headless via the Dataverse client + `PublishXml`,
 add to solution `--componentType 61`. Enable rules: a command-bar enable-rule JS function returning bool can gate
 visibility (e.g. only show on records in an Exception state).

### Icons — convention (the user, durable)
Put an **emoji directly in the label** when possible (simplest/fastest). For **AI features use the real Copilot icon** when
the action is **Copilot-tied** = system web resource **`msdyn_CopilotIcon.svg`** (variants `*NoColor` / `*WithColor`); bind
via Icon → **Use web resource → Add web resource → search `msdyn_CopilotIcon`**. Reserve a **sparkle** only for AI that is
**not** directly Copilot (e.g. AI Builder). For table/nav icons, custom `*_icon_*.svg` web resources (`webresourcetype=11`)
bound via PUT EntityDefinitions (If-Match:* + MSCRM.MergeLabels:true) — a plain PATCH of `IconVectorName` 405s.

### Xrm.Copilot client API — drive the M365 Copilot side panel (layer B, code)
`Xrm.Copilot` is the **JS bridge to the M365 Copilot / BizChat side panel** (NOT the app copilot, NOT the agent feed). Live
in the demo org (`isM365CopilotEnabled` returns **true** — works without a per-user M365 Copilot licence in this env). Methods:
`openM365CopilotPanel`, `sendPromptToM365Copilot(prompt, options)` (`{}` auto-submits; returns a Promise), `updateContext`,
`addActionHandler`/`removeActionHandler` (register a handler the M365 Copilot can invoke), `isM365CopilotEnabled`.
- **Demo asset built (Contoso Water Utility):** an **"✨ Explain exception"** command on the `cwu_invoice` form → web resource reads the
 form fields, `openM365CopilotPanel` + `sendPromptToM365Copilot(contextualPrompt)` → BizChat answers grounded on org data
 (SharePoint policies, Outlook vendor threads) with Sources. This is the cleanest **layer-B / code** Copilot demo.
- **Three model-driven Copilot surfaces — don't conflate** (also in `08-white-ranger-copilot-studio.md`): **(A)** App copilot =
 Copilot menu **"App Skills"** (the app assistant agent + prompt guide + ZPE); **(B)** M365 Copilot = Copilot menu **"Chat"**
 (BizChat; `Xrm.Copilot` targets this); **(C)** Agent Feed = autonomous agents posting task cards via the **Power Apps MCP
 server**. Enabling **M365 Copilot in MDA does NOT hide the app copilot** — they coexist as the two Copilot-menu entries.

### Sitemap SubArea — Entity vs Url + VectorIcon (icon inheritance)
A table **Entity SubArea** (`Entity='<logical>'`) **inherits the table's `IconVectorName` automatically** — no `VectorIcon`
needed. A **Url SubArea** (`Url='/main.aspx?...etn=...&viewid=...'`) does **NOT** inherit an icon — set
`VectorIcon="$webresource:<name>.svg"` explicitly. Prefer an **Entity SubArea on the real table** over a filtered-view Url
SubArea when a dedicated table exists (Contoso Water Utility: repointed "Duplicate Alerts" from a filtered `cwu_invoice` view to the real
`cwu_duplicatealert` table). ⚠️ The only **published** sitemap visible via `sitemaps?$select=sitemapname` may be an OLD one;
the app's real sitemap is reachable via **appmodulecomponents componenttype 62** off the app's `appmoduleidunique`. After
PATCH, a pinpoint `PublishXml` may not refresh the client cache — **`PublishAllXml`** does.

### Bare-table main forms — build the formxml headless
A table created headless via Web API gets a **bare auto-generated main form** (only the primary column + owner + a notes
control), NOT all columns. To surface the rest, build/PATCH the main form's **formxml** (`systemforms` type=2) + `PublishAllXml`.
Field control classids: standard (text/money/date/decimal/memo) `{4273EDBD-AC1D-40d3-9FB2-095C621B552D}`, lookup
`{270BD3DB-D9AF-4782-9025-509E298DEC0A}`, choice `{3EF39988-22BB-4f0b-BBBE-64B5A3748AEE}`, boolean
`{B0C6723A-8503-4fd7-BB28-C8A06AC933C2}`. Gotchas: a literal `&` in a section label breaks formxml parse (0x80048426) → escape
as `&amp;`; a literal `%` width in a Python %-format string → `%%`. (Contoso Water Utility: rebuilt cwu_duplicatealert + cwu_purchaseorder
forms this way; also two **quick-view forms** side-by-side to compare suspect vs original invoice on an alert.)
