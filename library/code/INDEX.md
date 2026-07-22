# PowerSquad — Code Library (index)

**Standing rule (the user): whenever the squad writes ANY code, keep a copy here for reuse.**
Code apps, PCF components, React/Vite projects, form-scripting JS web resources, Power Fx snippets, plug-ins,
helper scripts, embed snippets (e.g. M365 Agents SDK Web Chat), etc. — drop a copy under `library/code/` and add
a row to the index below so we can find it later.

Why: some artifacts **can't be pulled back** from Power Platform once deployed — notably **Power Apps Code apps**
(`pac code` is push-only, no pull/clone; lose the local folder → the source is gone). Keeping a versioned copy here
is the safety net + the reuse catalog.

## How to use this library
- One **subfolder per code artifact**: `library/code/<kebab-name>/` with the actual source (or a faithful copy).
- Include a short `README.md` in each subfolder: what it is, which app/agent it came from, env/IDs, how to reuse,
 any secrets to replace (never commit real secrets/tokens — use placeholders).
- **Add a row to the index table below** every time you add or update an artifact.
- Prefer Git/source control for anything substantial; this folder is the squad-visible catalog + fallback copy.

## Index
| Artifact | Type | Owner agent | Source app/agent | Path | Added | Notes |
|---|---|---|---|---|---|---|
| PSQ Draft Offer Email | Agent flow (modern cloud flow + AI Builder prompt) | White Ranger | PSQ Recruiting Assistant | `white-ranger-agent-flow-offer-email/` | | `workflow.json`+`metadata.yml` from `pac copilot clone`; prompt referenced by GUID (recreate prompt in new env). See subfolder README. |
| candidates.csv (code interpreter demo) | Sample structured data (CSV) | White Ranger | PSQ Recruiting Assistant | `white-ranger-code-interpreter-candidates/` | | 9 candidate rows incl. a deliberate outlier (Ava Chen 540k) for the code-interpreter IQR demo. Regenerate from psq_candidate. |
| Candidate Adaptive Card | Adaptive Card (schema 1.5) JSON | White Ranger | PSQ Recruiting Assistant | `white-ranger-adaptive-card-candidate/` | | Static + templated card shown via topic "Show Candidate Card" (Send message > Adaptive card modality). See subfolder README. |

> Keep this table current — it's the thing we consult to answer "have we built this before?".
