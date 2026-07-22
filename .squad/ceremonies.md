# Ceremonies

> Team meetings that happen before or after work.

## Scope & Prerequisite Gate

| Field | Value |
| ---------------- | ------------------------------------------------------------------- |
| **Trigger** | auto |
| **When** | before |
| **Condition** | new build request (demo, PoC, or customization) with any ambiguity in scope or prerequisites |
| **Facilitator** | Zord |
| **Participants** | Zord, Gold Ranger, Black Ranger, Green Ranger, Red Ranger, Yellow Ranger, Blue Ranger, White Ranger |
| **Time budget** | focused |
| **Enabled** | ✅ yes |

**Agenda:**

1. Confirm business problem, target persona, and proposed artifacts.
2. Run the autonomy gate (Full auto / Per-artifact / Co-pilot) via `m_ask_user`.
3. Compile the prerequisite checklist and flag every manual human action.
4. Confirm (or create) the delivery solution name and broadcast it to all specialists.

---

## Build Debrief

| Field | Value |
| ---------------- | --------------------------------------------------------- |
| **Trigger** | auto |
| **When** | after |
| **Condition** | the squad completed a build pass (data model and/or app) |
| **Facilitator** | Silver Ranger |
| **Participants** | all-involved |
| **Time budget** | focused |
| **Enabled** | ✅ yes |

**Agenda:**

1. Confirm every artifact landed inside the delivery solution (no orphans in Default).
2. Record reusable patterns, gotchas, and prerequisites for future builds.
3. Confirm the build summary + Gold Ranger's business-value dossier are ready to hand to Pink Ranger for packaging.
