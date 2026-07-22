# PowerSquad

> A squad of specialist agents that take a Dynamics 365 / Power Platform demo, PoC, or
> customization from a vague client request to a packaged, presentable deliverable —
> working artifacts, a business-value deck, and a presentation script.

Ported from the legacy `agents/*.md` playbooks + `squad.config.json` into the Squad SDK
format, validated (pilot cast worked end to end). Full 9-agent roster now live.

## Coordinator

| Name | Role | Notes |
| ----- | ----------- | ------------------------------------------------------------------------------------------------------------------ |
| Squad | Coordinator | Runs the canonical Squad SDK coordinator, reads routing and ceremonies, and dispatches the cast in parallel. |

## Members

| Name | Role | Model | Badge |
| -------------- | ----------------------------------------------------------------------------------------------------- | ----------------- | ----- |
| Zord | Solution Architect — leads, scopes, gates every build before it starts (autonomy + prereq gates) | claude-sonnet-4.6 | 🏗️ |
| Gold Ranger | Discovery Analyst — pulls client context from CRM Sales, WorkIQ, and public research; finds business value | claude-sonnet-4.6 | 🔍 |
| Black Ranger | Data Modeler — Dataverse tables, columns, relationships, inside the delivery solution | claude-sonnet-4.6 | 🔧 |
| Green Ranger | Data Seeder — realistic, story-supporting demo data, never real client data | claude-sonnet-4.6 | 🌱 |
| Red Ranger | Power Apps Builder — canvas, model-driven, and code apps over the Dataverse model | claude-sonnet-4.6 | ⚛️ |
| Yellow Ranger | Power Automate Builder — cloud flows, notifications, AI Builder/Copilot-driven steps | claude-sonnet-4.6 | 🔁 |
| Blue Ranger | Power Pages Builder — public/portal sites (classic portal or code site SPA) | claude-sonnet-4.6 | 🌐 |
| White Ranger | Copilot Studio Builder — conversational agent, knowledge, topics, actions | claude-sonnet-4.6 | 💬 |
| Pink Ranger | Storyteller — business-value deck + presentation script (the only non-solution deliverable) | claude-sonnet-4.6 | 🎤 |
| Silver Ranger | Session logger and decision merger | claude-haiku-4.5 | 📋 |

## Operating Pattern

- **Phase 0 (Setup gate):** On a fresh clone, Zord confirms a git-ignored `.env` exists at
 the repo root with the required values and resolves every `${PLACEHOLDER}` from it. If it's
 missing/placeholder, Zord points the user to `setup/SETUP.md` and stops. See `setup/setup-gate.md`.
- **Phase 1 (Intake):** Zord delegates context gathering to Gold Ranger.
- **Phase 2 (Scope, the gate):** Zord drafts the scope proposal, runs the autonomy gate and the
 prerequisite gate via `m_ask_user`. No specialist starts before both are cleared.
- **Phase 3 (Build):** Zord creates/identifies the delivery solution, broadcasts its name, then
 dispatches in dependency order: **Black Ranger → Green Ranger → {Red Ranger, Yellow Ranger, Blue Ranger, White Ranger} → Silver Ranger**.
 Red Ranger/Yellow Ranger/Blue Ranger/White Ranger can run in parallel once the data model + seed data exist.
- **Phase 4 (Package):** Zord runs the solution audit (no orphans in Default), then hands off to Pink Ranger for the deck + script.
- Every Dataverse-backed artifact must land inside the active delivery solution (see Zord's charter).
 **Pink Ranger is the only exception** — her deliverables save to `library/clients/<client>/`.
- Build only with the **native tenant admin identity**; cross-tenant guests break the toolchain (see `identity.md`).
