# Setup Gate — run this FIRST on activation

Before the squad builds anything, the coordinator (**Zord**) must confirm the
environment is configured. This is a hard gate: **no specialist starts until it
passes.**

## The check

1. **Does `.env` exist at the repo root?**
 - If **no** → tell the user PowerSquad isn't configured yet and point them to
 `setup/SETUP.md` (run `pwsh ./setup/setup-powersquad.ps1` or copy
 `setup/.env.example` → `.env`). **Stop here** until it exists.
2. **Are the REQUIRED values filled** (non-empty, not still the example
 placeholder like `orgXXXXXXXX` / `00000000-...` / `YourDemoEnvironmentName`)?
 Required: `DEMO_ORG_URL`, `ENVIRONMENT_ID`, `ENVIRONMENT_NAME`, `TENANT_ID`,
 `ADMIN_UPN`, `DEMO_MCP`.
 - If any are missing/placeholder → list exactly which, and ask the user to
 complete them (re-run the helper or edit `.env`). **Stop** until fixed.
3. When the required set is present, **load `.env`** and use those values
 wherever the specs reference a `${PLACEHOLDER}` (org URL, tenant/env IDs,
 admin UPN, MCP name, etc.). Optional values (`CORP_*`, `CRM_SALES_HOST`,
 `CONTENT_LIBRARY_HOST`, `USER_EMAIL`) are used only by the agents that need
 them; their absence is fine.

## Passing the gate

Once `.env` is present and complete, confirm to the user in one line
(e.g. *"Environment configured: <ENVIRONMENT_NAME> — ready to build."*) and
proceed to the normal flow (Intake → Scope + gates → Build → Package).

## Notes

- This gate is about **environment configuration**, and is separate from Zord's
 two build gates (the **autonomy gate** and the **prerequisite gate**), which
 still run in Phase 2. Order on a fresh clone: **setup gate → autonomy gate →
 prerequisite gate → build.**
- Never print secret-like values back to the user in full; a short confirmation
 (env name) is enough.
- `.env` is git-ignored — never write real values into any committed file.
