# PowerSquad — Reuse Library

First-class, growing collection of assets that make each new demo start ahead.
Zord proposes promoting generic assets here after every engagement.

## Folders
- `deck-themes/` — reusable deck templates / themes for Pink Ranger (PPTX + HTML).
- `table-patterns/` — reusable Dataverse schema patterns by industry/use case (Black Ranger).
- `flow-snippets/` — reusable Power Automate flow patterns (Yellow Ranger).
- `copilot-agents/` — reusable Copilot Studio topics / agent templates (White Ranger).
- `code/` — **any code we write** (Code apps, PCF, React/Vite, form-scripting JS, Power Fx snippets, plug-ins,
 embed snippets). One subfolder per artifact + a row in `code/INDEX.md`. **Standing rule:** always keep a copy here —
 some code (esp. **Code apps**) can't be pulled back from Power Platform once deployed.
- `script-templates/` — reusable presentation script skeletons (Pink Ranger).
- `clients/` — **private** per-client working folders. NOT shareable.

## Per-client folder (`clients/<client>/`)
Created during an engagement. Typical contents:
- `dossier.md` — Gold Ranger's context (PRIVATE: real client data, never ships to client)
- `scope.md` — Zord's approved scope
- `manifest.md` — everything built (solution name, tables, apps, flows, pages, agent) + reset/redeploy steps
- `<solution>.zip` — exported managed solution
- `deck.pptx` / `deck.html` + `script.md` — Pink Ranger's deliverables

## Promotion rule
An asset graduates from a client folder into a shared library folder only after being
**genericized** (no client-private data, parameterized names, documented). Zord proposes;
the user approves.
