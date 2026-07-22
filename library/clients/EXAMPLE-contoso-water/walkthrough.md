# Example walkthrough — Contoso Water Utility (fictional)

> **This is a worked example, not real data.** Contoso Water Utility is an
> invented customer. Every name, table, and number below is fabricated to show
> how the squad flows end to end. No real client data, GUIDs, org URLs, or
> credentials appear anywhere in this file. Use it as a template for what a real
> engagement looks like — your own runs go in `library/clients/<your-client>/`
> (git-ignored).

## The ask (what the user brought to Zord)

> "I'm meeting Contoso Water Utility next week. They're a regional water
> provider drowning in manual invoice and field-work paperwork. Finance can't
> spot duplicate supplier invoices, and field crews log meter jobs on paper.
> I want a demo that shows Dynamics + Power Platform + Copilot cleaning this up.
> I have the opportunity in CRM and a couple of discovery-call notes."

---

## Phase 0 — Setup gate

Zord checks the repo: `.env` exists and the required values are filled
(`DEMO_ORG_URL`, `ENVIRONMENT_ID`, `ENVIRONMENT_NAME`, `TENANT_ID`, `ADMIN_UPN`,
`DEMO_MCP`). Green. Zord loads it and confirms:

> *"Environment configured: **CWU-Demo** — ready to build. Let's scope the
> Contoso Water Utility demo."*

---

## Phase 1 — Intake · driver: Zord → **Gold Ranger**

Gold Ranger pulls the opportunity + discovery notes (read-only context) and
writes a **Context Dossier**. Distilled:

- **Who:** Contoso Water Utility — regional utility, ~1,200 staff, ~90k customer
 meters. Finance + Field Operations are the pain owners.
- **Pains:**
 1. **Duplicate/near-duplicate supplier invoices** slip through AP → overpayment.
 2. **Field meter jobs** captured on paper → re-keyed → days of lag, errors.
 3. **No single view** of a supplier's invoices, disputes, and jobs.
- **"Wow" candidates:** Copilot that *explains* why two invoices look like
 duplicates; a field app that turns a photo of a meter into a logged reading.
- **Low-hanging fruit:** the duplicate-invoice detector (high value, small build).

Dossier lands in `library/clients/contoso-water/dossier.md` (git-ignored in a
real run).

---

## Phase 2 — Scope (the gate) · driver: Zord, with the user

Zord turns the dossier into a **Scope Proposal**:

| | |
| --- | --- |
| **Problem** | AP overpays on duplicate invoices; field jobs lag on paper. |
| **Personas** | *Priya* (AP Analyst), *Marcus* (Field Supervisor). |
| **Build** | Data model → seed data → model-driven app + a canvas field app → a duplicate-detection flow → a Copilot agent that explains duplicates → deck. |
| **Before → after** | 3-day invoice review & manual dup-hunting → same-day, auto-flagged with an explanation. Paper meter logs → photo-to-record in the field. |
| **Storyline** | "A duplicate slips in. The flow flags it in seconds. Copilot explains *why*. Priya approves the hold in one click." |

**Autonomy gate** → user picks **Per-artifact** (show each plan before building).
**Prerequisite gate** → Zord lists the manual steps: sign in to the demo org as
the native admin, open a model-driven Studio tab. User clears them.

Zord creates/identifies the delivery solution **`cwu_demo`** and broadcasts the
unique name to every builder.

---

## Phase 3 — Build (dependency order)

### Black Ranger — Data Modeler *(first; everyone depends on the model)*
Creates, all inside `cwu_demo`, each custom table with an **icon**:
- `cwu_supplier` (Supplier) — name, category, risk band.
- `cwu_invoice` (Invoice) — supplier (lookup), amount, invoice #, date, PO #, status.
- `cwu_duplicatealert` (Duplicate Alert) — **two lookups** to `cwu_invoice`
 (suspect ↔ original) + score, confidence, reason.
- `cwu_meterjob` (Meter Job) — meter #, reading, photo, crew, status.

Adds a `cwu_triage` formula column on `cwu_invoice` (🟢/🟡/🔴 by amount + status),
and **backfills every existing row** so no field shows blank.

### Green Ranger — Data Seeder *(after the model)*
Seeds **synthetic** story-supporting data — never real client data:
- 12 suppliers, 45 invoices (with **one deliberate duplicate pair**: INV-2041 and
 INV-2041-A, same supplier "Blue Ridge Pumps", 82,400, two days apart, same PO).
- 16 meter jobs across 3 crews, a couple still "on paper / pending".

### Red Ranger — Power Apps · Yellow Ranger — Automate · White Ranger — Copilot Studio *(parallel now)*

- **Red Ranger** builds:
 - A **model-driven app** "CWU Finance Center" (sitemap: Invoices, Suppliers,
 Duplicate Alerts, Meter Jobs) — publishes as the owning admin.
 - A **canvas field app** "CWU Meter Capture": crew picks a job, snaps the
 meter photo, enters the reading, Patches it back to `cwu_meterjob`.

- **Yellow Ranger** builds the **"CWU Detect Duplicate Invoices"** cloud flow:
 trigger *When a row is added or modified* on `cwu_invoice` (scope: Organization);
 a deterministic pre-filter + score (vendor .30 / amount .30 / date .15 / PO .15 /
 number .10; flag at ≥ 0.50 with 2+ strong signals); on a hit it writes a
 `cwu_duplicatealert` (suspect → original) and notifies Priya.
 *Test:* inserting INV-2041-A → an alert appears in ~20s at 90% / High. ✅

- **White Ranger** builds the **"CWU Finance Assistant"** Copilot Studio agent:
 grounded on the invoice + alert tables; a topic **"Explain this duplicate"**
 that takes an alert and returns a plain-language rationale ("same supplier,
 amount identical, 2 days apart, identical PO — very likely the same invoice
 re-submitted"). An **agent flow** drafts the supplier hold-email.

### Silver Ranger — logger *(throughout)*
Records every decision, the solution name, and the tested outcomes into the
session log so the run is reproducible.

---

## Phase 4 — Package · driver: Zord → **Pink Ranger**

Zord runs the **solution audit**: everything (tables, app, flow, agent) is
confirmed inside `cwu_demo` — no orphans in the Default solution.

Pink Ranger produces the **only non-solution deliverables** (saved to
`library/clients/contoso-water/`, not the org):
- **Business-value deck** — the before/after, the "duplicate caught + explained"
 money slide, the field photo-to-record moment, and a rollout path.
- **Presentation script** — the exact click path and talk-track for the live demo,
 timed to ~8 minutes.

---

## The delivery pack

What ships to the user:
1. `cwu_demo` solution (importable) — model, apps, flow, Copilot agent, seed data.
2. Deck + presentation script.
3. A short "what to say / what to click" runbook.

**Net story:** a vague "clean up our paperwork" ask became a packaged, live-
demoable solution — a flagged-and-explained duplicate invoice and a photo-logged
meter reading — assembled by the squad in one governed pass.
