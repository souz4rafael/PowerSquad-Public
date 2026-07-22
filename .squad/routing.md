# Work Routing

How to decide who handles what.

## Routing Table

| Work Type | Route To | Examples |
| ----------------------------------------------------- | ------------------- | ---------------------------------------------------------------------------- |
| Scope, gating, orchestration, solution governance | 🏗️ Zord | New client/demo intake, autonomy gate, prerequisite gate, solution audit |
| Client/opportunity context, business-value framing | 🔍 Gold Ranger | CRM Sales/WorkIQ/web research, dossier, low-hanging-fruit identification |
| Dataverse data model | 🔧 Black Ranger | Tables, columns, relationships, choices, form/view wiring |
| Demo data population | 🌱 Green Ranger | Seeding realistic rows, lookups, bulk inserts |
| Power Apps (canvas / model-driven / code apps) | ⚛️ Red Ranger | Building or editing an app over the Dataverse model |
| Power Automate cloud flows | 🔁 Yellow Ranger | Notifications, approvals, orchestration, agent-callable flows |
| Power Pages sites | 🌐 Blue Ranger | Classic portal or code-site SPA over the data model |
| Copilot Studio agent | 💬 White Ranger | Topics, knowledge grounding, actions calling flows/Dataverse |
| Business-value deck + presentation script | 🎤 Pink Ranger | Final client-facing narrative, after the build is summarized |
| Session logging | 📋 Silver Ranger | Automatic after substantial work |

## Multi-Agent Routing

| Scenario | Agents | Why |
| ------------------------------------------------ | ---------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| New demo/PoC request end to end | Zord → Gold Ranger → Black Ranger → Green Ranger → {Red Ranger, Yellow Ranger, Blue Ranger, White Ranger} → Pink Ranger → Silver Ranger | Full lifecycle: intake → scope/gate → data model → seed → parallel build → package |
| Data-model-only change | Zord + Black Ranger + Silver Ranger | Zord confirms the target solution before Black Ranger touches schema |
| App-only change on an existing model | Zord + Red Ranger + Silver Ranger | Zord confirms solution + identity before Red Ranger builds |
| New flow that an agent will call | Zord + Yellow Ranger + White Ranger + Silver Ranger | Yellow Ranger builds the flow, White Ranger wires it as an agent tool (the White Ranger↔Yellow Ranger bridge) |
| Client discovery only (no build yet) | Zord + Gold Ranger + Silver Ranger | Dossier + business-value hypotheses precede any scope commitment |
| Packaging an already-built demo | Zord (solution audit) + Pink Ranger + Silver Ranger | Audit first, then storytelling — Pink Ranger never starts before the audit |

## Rules

1. **Zord gates everything.** No specialist starts before Zord's autonomy gate and prerequisite gate are cleared.
2. **Dependency order is fixed:** Gold Ranger (context) → Zord (scope) → Black Ranger (data model) → Green Ranger (seed data)
 → Red Ranger/Yellow Ranger/Blue Ranger/White Ranger (build, parallel once data exists) → Pink Ranger (package, only after Zord's solution audit).
3. **Every Dataverse artifact must land in the delivery solution.** Zord owns and broadcasts the solution name.
 **Pink Ranger is the only exception** — her deliverables save outside the solution, to `library/clients/<client>/`.
4. **Native tenant admin identity only.** Cross-tenant guest sessions break the toolchain — flagged by Zord's prereq gate.
5. **Silver Ranger always runs after substantial work.**
6. **Direct invocation is allowed.** Any agent can be addressed by name to run a scoped task without going through
 Zord (per `squad.config.json` → `directInvocation`). They still check their own prerequisites first.
