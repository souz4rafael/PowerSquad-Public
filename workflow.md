# PowerSquad — Workflow

The end-to-end flow. **Zord** (Solution Architect) owns this orchestration and is the only
agent that talks to you across all phases. Specialists are invoked by Zord.

```
INTAKE ──> SCOPE (gate) ──> BUILD ──> PACKAGE ──> DELIVER (the delivery pack)
```

## Phase 0 — Setup gate (first run per clone)
Before anything else, Zord confirms the environment is configured: a git-ignored `.env`
must exist at the repo root with the required values (`DEMO_ORG_URL`, `ENVIRONMENT_ID`,
`ENVIRONMENT_NAME`, `TENANT_ID`, `ADMIN_UPN`, `DEMO_MCP`). If it's missing or still on the
example placeholders, Zord points you to `setup/SETUP.md` (`pwsh ./setup/setup-powersquad.ps1`)
and **stops** until it's complete. Full rules in `setup/setup-gate.md`. Once green, Zord loads
`.env` and resolves every `${PLACEHOLDER}` in the specs from it, then proceeds to Intake.

## Phase 1 — Intake
**Driver: Zord → delegates context gathering to Gold Ranger.**

You provide whatever you have:
- Client / account name
- Client ID or opportunity number (for CRM Sales)
- Which meetings you had, when (for WorkIQ + transcripts)
- Any group chats / colleague conversations about the client

Gold Ranger produces a **Context Dossier** in `library/clients/<client>/dossier.md`:
- Who the client is, industry, the people involved
- The opportunity / deal context (from CRM Sales)
- What was discussed in meetings & chats (from WorkIQ) — pains, asks, constraints
- Candidate use cases + identified low-hanging fruit

## Phase 2 — Scope (the gate)
**Driver: Zord, with you.**

Zord turns the dossier into a **Scope Proposal**:
- Business problem + target persona
- Proposed solution shape (which artifacts: apps / flows / pages / Copilot agent / data / customizations)
- Business value framing: before → after, metrics, "wow" moment
- Low-hanging fruit vs. strategic bets
- Demo storyline (the narrative the demo will tell)

**Autonomy gate (mandatory).** Zord assesses how clear the scope is and asks you via
`m_ask_user` which mode to run in:
- **Full auto** — build everything, only report at milestones
- **Per-artifact** — show each artifact plan before building it
- **Co-pilot** — confirm every significant step

No specialist starts until the scope and autonomy level are agreed.

### Prerequisite gate (end of Phase 2)
After scope + autonomy, Zord compiles and presents the **prerequisites checklist** for the chosen
artifacts and **flags every manual human action** (admin browser sign-in + MFA, tenant settings,
creating a blank canvas app + Coauthoring + keeping the Studio tab open, installing .NET 10 / Node,
CLI auth to the demo tenant). Each builder spec owns its prereq list (e.g. Red Ranger's per-app-type
prereqs in `agents/05-red-ranger-power-apps.md`). The build does not start until the user confirms the
pending manual steps are done. **Golden rule:** build with the **native tenant admin** identity —
cross-tenant guests cause 404s, "environment not found", and orphaned ownership.

## Phase 3 — Build
**Driver: Zord, delegating to specialists in dependency order.**

Zord first creates or identifies the delivery **solution** (`psq_<client>_<date>`; **for a quick demo,
a single reusable `psq_demo`**) in the demo org, and **broadcasts its exact unique-name
to every builder**. Then dispatches:

```
Black Ranger (data model) ─┐
 ├─> all build into the SAME solution (the one Zord broadcast)
Green Ranger (data) ───────┤
Red Ranger (apps) │
Yellow Ranger (flows) │
Blue Ranger (pages) │
White Ranger (copilot) ────┘
```

**Solution rule (whole squad):** every Dataverse-backed artifact MUST be created inside (or added to)
that solution. **Only exception: Pink Ranger** (Phase 4) — her deck/script save to the PC/`library/`, never
the solution. Power Pages sites ARE solution-aware (enhanced data model) — Blue Ranger sets the Current
solution before creating components and adds the site via Solutions → Add existing → Site.

Dependencies:
- **Green Ranger** waits for **Black Ranger** (need tables before data).
- **Red Ranger / Blue Ranger / White Ranger** wait for **Black Ranger** (and usually some **Green Ranger** data).
- **Yellow Ranger** can run in parallel once the tables it touches exist.

Each specialist reports back to Zord: what it built, where, **and confirms it's a member of the
delivery solution**. Before packaging, **Zord runs a solution audit** (lists components, backfills any
orphan that fell into Default).

### Build-time gotchas (squad-wide, the demo org — validated 2026-06)
- **Publish/activate lock:** `pac solution publish` (and `WorkflowSetState`) often fail with `another
 [Import]/[Uninstall] running` because the demo org auto-installs first-party solutions in waves — **not a real error**.
 Use a **retry-until-success loop** (~30s backoff); inspect `importjobs` (completedon null) / `asyncoperations`
 (statecode 0) to see what's holding the lock.
- **Client IndexedDB cache:** freshly published changes (site map, theme, New look, forms, custom pages) may not
 appear in the running app until the **browser IndexedDB cache** rebuilds — the **server descriptor is already
 correct**. Force fresh via Studio **Play (F5)**, clear IndexedDB + reload, or wait. Verify server-side (the
 relevant entity's XML/metadata) rather than trusting the cached runtime.
- **Seeded rows + BPF:** API/SDK-created rows do **not** auto-instantiate a BPF — Green Ranger must create the BPF
 instance per seeded row (see `agents/04-green-ranger-data-seeder.md`). A cross-agent dependency Zord should flag.

## Phase 4 — Package
**Driver: Pink Ranger, co-designed with Zord + Gold Ranger.**

- Zord exports the solution (managed) via pac-mcp into `library/clients/<client>/`.
- **Zord summarizes** everything that was built (solution audit + each specialist's report).
- **Zord + Gold Ranger + Pink Ranger discuss together** and elaborate **what Pink Ranger should deliver** —
 format(s), depth, narrative, audience. Gold Ranger brings the **business-value findings**, Zord brings
 the **built artifacts**. Only after this co-design does Pink Ranger build. **The format is decided here
 (post-delivery), not upfront** — any of **PPTX / HTML / Word / .txt**, or a combination.
- **Pink Ranger** builds the **business-first** presentation (business value, before/after, the artifacts as
 value moments; a few light technical/solution slides are OK but secondary) + the **presentation script**
 (scene-by-scene, mapped to the live demo click-path; in slide notes when PPTX). May pull approved
 messaging/templates from **your enablement/content library** (`${CONTENT_LIBRARY_HOST}`, @${CORP_DOMAIN} sign-in).

## Deliver — the delivery pack
The packaged bundle:
- Managed solution (importable)
- Deck + script
- Context dossier (private — not for the client)
- A `manifest.md` listing everything built and how to reset/redeploy

## Reuse loop
After delivery, Zord proposes promoting any generic asset (a table pattern, a flow,
a Copilot agent template, a deck theme) into `library/` so the next demo starts ahead.
