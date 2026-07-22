# Adaptive Card — Candidate card (White Ranger)

A rich **Adaptive Card** (schema 1.5) shown by the **PSQ Recruiting Assistant** via a custom **topic**
("Show Candidate Card"). Validated asking *"show me the candidate card"* triggers the topic and renders
the card in the test pane (header with avatar + name + position + stage badge, a FactSet, a summary, and an
**Action.OpenUrl "Open record"** button).

## Files
- `candidate-card-static.json` — the exact JSON used in the demo (hard-coded Sophia Johansson). Robust to paste.
- `candidate-card.json` — **templated** version using `${...}` bindings (CandidateName, Position, Stage, AnnualComp,
 SeniorityBand, Email, Summary, RecordUrl) for when the card is data-bound from a topic/variable or flow output.

## How it was added (portal, validated)
1. Agent → **Topics → Add a topic → From blank**. Name it; set the **trigger** description (generative): e.g.
 *"Shows a rich candidate card. Triggers when the user asks to show/display/see a candidate as a card."*
2. **Add node → Send a message** → on the Message node **Add modality → Adaptive card** → **Edit adaptive card**.
3. In the card designer's **Card payload editor** (Monaco): focus it, **Ctrl+A → Delete**, then **paste** the JSON
 (set the clipboard from the file and Ctrl+V — typing large JSON triggers Monaco auto-bracket issues). The designer
 live-renders the card. **Save** (card designer) → **Close** → **Save** (topic) → **Publish** the agent.
4. Test: *"show me the candidate card"* → topic fires, card renders.

## Notes / gotchas
- **Schema ≤ 1.6** in Copilot Studio (Web Chat 1.6, no `Action.Execute`; Teams/Omnichannel 1.5). Keep cards at **1.5**
 to be safe. The **test pane renders the card**; the **canvas app does not** render agent Adaptive Cards.
- To **show** a card use **Send a message + Adaptive card modality**. To **collect input** use **Ask with adaptive
 card** instead (gives you the submitted values). Give every Submit a **unique id** to avoid cross-card misfires.
- **Data-binding:** use the templated `candidate-card.json` and feed values from topic variables or a flow's output
 (the card supports `${var}` template expressions).
- Authoring/preview also at `adaptivecards.microsoft.com` (or generate the JSON with Copilot), then paste here.
