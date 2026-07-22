# PowerSquad — Frameworks (from the Agentic Transformation Patterns Playbook)

Source: Microsoft *Agentic Transformation Patterns Playbook* (PowerPnP Guidance Hub,
Apr 2026) + the June 2026 Business Apps "What's New" deck. These frameworks give the squad
a shared language for **scoping** and **business-value framing**. Zord uses them in Phase 2;
Pink Ranger uses them in the deck.

## 1. The core narrative: Assist → Execute
Agents are moving from **assisting** humans (human decides + executes, agent supports) to
**executing** work (agent acts across systems, human oversees outcomes). This shift creates
four new demands every demo should speak to: **Ownership, Risk, Lifecycle, Governance.**
Great demos show an agent that *does* something, not just chats.

## 2. The six transformation patterns (classify every demo into one)
Zord tags the engagement with the pattern(s) it targets. Patterns are design choices, not
stages — most clients run 2–3 at once.

| # | Pattern | What agents do | Demo angle |
|---|---|---|---|
| 1 | **Employee AI Enablement** | Personal productivity; human stays accountable | Assistants embedded in daily work |
| 2 | **Business Expert Empowerment** | Scale an expert's judgment as on-demand guidance | Policy/standards Q&A with escalation |
| 3 | **Workplace & IT Services** | Run internal services end-to-end (HR/IT/Finance) | Self-service that *executes* requests |
| 4 | **Core Business Processes** | Agents woven into business-critical flows | Higher-risk workflow automation |
| 5 | **External Engagement** | Customer/partner-facing agents | Power Pages + Copilot agent at the trust boundary |
| 6 | **AI-first Business Capabilities** | Net-new capabilities impossible before AI | "Different everything" innovation |

> Mapping to our builders: pattern 5 leans on **Blue Ranger** (Pages) + **White Ranger** (Copilot);
> patterns 3–4 lean on **Yellow Ranger** (flows) + **Black Ranger/Green Ranger** (process data); patterns 1–2
> lean on **White Ranger** (knowledge agents). **Red Ranger** (apps) supports all.

## 3. Maturity model (frame value + next steps)
Five capability drivers, scored across five levels (100 Initial → 200 Repeatable →
300 Defined → 400 Capable → 500 Optimized):

1. **AI Strategy & Experience** — how deliberately AI is planned/invested in
2. **Business Strategy** — how deeply AI is integrated into processes + outcome metrics
3. **Governance & Security** — risk, compliance, monitoring, responsible AI
4. **Technology & Data** — platforms, architecture, data quality, telemetry
5. **Organization & Culture** — adoption, skills, AI-positive culture

**Key idea — the scale-breaker:** your *weakest* driver is your ceiling. Don't lift every
driver; find the ONE that breaks scale first and fix it. Each pattern needs different depth
(e.g. Employee Enablement needs Culture 300; Expert Empowerment needs Governance + Tech/Data 300;
External Engagement needs the deepest Governance & Security). Ref: https://aka.ms/AgentMaturityModel

## 4. Operating model (the "after the demo" story)
- **Center of Excellence (CoE)** turns intent into repeatable, trusted execution.
 Structures: **Centralized** (control/regulated), **Hybrid** (central standards + local build),
 **Federated** (BUs own outcomes; CoE governs by exception).
- CoE roles: Executive Sponsor, Business Owner, CoE Lead/AI Program Mgr, Agent Product Owner,
 Platform & Operations, Security/Risk/Compliance. **Makers** build agents in Copilot Studio /
 Agent Builder within guardrails; **Domain Experts** own knowledge quality.
- **90-day play:** pick your pattern → name an owner → find your scale-breaker → start Monday.

## 5. Product landscape notes (June 2026 — keep current)
- **Copilot for Sales / Service / Finance** are no longer standalone — folded into
 **Microsoft 365 Copilot** (agents). Position demos accordingly.
- **Agent 365** and **Microsoft 365 E7** are GA.
- **Copilot Cowork** moves from conversation → action with reusable **"Cowork Skills"**.
- **Agent Builder** (in M365 Copilot) is a lighter maker surface alongside **Copilot Studio**.
- **Dataverse = the agent data platform.** Dataverse in M365 Copilot grounds answers on
 business data; **Dataverse "business skills"** describe a process (steps + data + rules)
 that agents discover via the Dataverse MCP server and execute to org standards.
- **Power Apps MCP server** + **Agent feed** + **closed-loop learning**: agents (e.g. a
 data-entry agent) learn from user corrections — memory-based + Genetic-Pareto prompt
 optimization — improving automatically in production.
- **Computer-using agents (CUA)** are GA in Copilot Studio (agents that operate UIs).
- Stay current via the D365 release plans / roadmap: https://dynamics.microsoft.com/en-us/roadmap/overview/

## 6. Demo "wow moment" ideas (Assist→Execute, grounded in current capabilities)
- An agent that **executes** a process end-to-end (not just answers) via Dataverse business
 skills + a Yellow Ranger flow — pattern 3/4.
- A **closed-loop learning** moment: correct the agent once (Agent feed), show it apply the
 rule on the next record automatically — pattern 1/2.
- An **External Engagement** combo: Blue Ranger portal + White Ranger Copilot agent at the trust
 boundary, with table-permission security as a talking point — pattern 5.
- A **CUA** demo: the agent driving a UI to complete a task no API exposes — pattern 4/6.

## 7. Distribution & scale — the Agent Store (for "after the demo")
Source: Copilot Studio Blog, "4 ways to build a curated Agent Store" (May 2026) + 2026 Work
Trend Index (58% of employees produce work they couldn't a year ago). The **Agent Store** is a
curated hub in **Microsoft 365 Copilot** to discover/install/use agents in the flow of work
(Teams, Outlook, Word, Excel, PowerPoint), built by Microsoft, trusted partners, and your teams.
It addresses the three adoption blockers: **where do we start, what can we trust, how do we
drive adoption.** Four pathways to populate a curated store (Pink Ranger/Zord use this as the
"scale" next-step):
1. **Deploy pre-built agents** from Microsoft / verified partners — fastest, no dev.
2. **Build & distribute via Agent Store** using Copilot Studio (fast) or the M365 Agents
 Toolkit + Agents SDK (pro-code).
3. **Bring your own agents** (Foundry, third-party, custom code) onboarded via the M365
 Agents Toolkit/SDK so they're discoverable in the Store.
4. **Integrate with the Microsoft Agent 365 SDK** for enterprise capabilities (Entra identity,
 governed M365 data access, observability, cross-surface notifications) without a full rebuild.
Ref: Administering and Governing Agents whitepaper — https://aka.ms/AgentGovernanceAndSecurity

## How the squad uses this
- **Zord (scope):** classify the demo into pattern(s); name the target maturity + likely
 scale-breaker; frame Ownership/Risk/Lifecycle/Governance; pick an "Execute" wow-moment.
- **Pink Ranger (deck):** structure value around Assist→Execute, the chosen pattern, the
 maturity/scale-breaker, and a 90-day next-steps slide (incl. CoE shape).
- **White Ranger (Copilot Studio):** for Expert Empowerment patterns, build escalation + knowledge
 quality controls, not just Q&A.
