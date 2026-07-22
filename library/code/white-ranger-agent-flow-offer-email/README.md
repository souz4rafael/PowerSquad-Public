# Agent flow — "PSQ Draft Offer Email" (White Ranger)

A Copilot Studio **agent flow** (modern cloud flow, `category=5`, `modernFlowType=1`) used as a **tool**
by the **PSQ Recruiting Assistant** agent. Demonstrates the squad's cleanest **agent → flow → AI → result**
pattern: the user asks the agent to "draft an offer email for <candidate>", the agent fills the inputs from
its Dataverse grounding, calls this flow, and returns the generated email in chat.

## Shape
- **Trigger:** `When an agent calls the flow` (Request / kind `Skills`) with 3 string inputs:
 `CandidateName` (→ `text`), `Position` (→ `text_1`), `AnnualSalary` (→ `text_2`).
- **Action `Run a prompt`:** AI Builder custom prompt **`PSQ Draft Offer Email`**
 (msdyn_aimodel `${MODEL_RECORD_ID}`, model **GPT-4.1 mini**), inputs bound to the trigger.
- **Action `Respond to the agent`:** output `OfferEmailDraft` (String) =
 `outputs('Run_a_prompt')?['body/responsev2/predictionOutput/text']`.

## The AI Builder prompt instruction (recreate if importing elsewhere)
> You are an HR recruiter at PSQ Recruiting. Write a warm, professional job offer email addressed to the
> candidate **[CandidateName]**, who applied for the position of **[Position]**. The annual salary offered is
> **[AnnualSalary]** USD per year. Include congratulations, the role and a PSQ sign-off.

Inputs are inserted in the prompt editor with `/` → **Text** (name + sample data). Sample data used:
`Sophia Johansson` / `Account Executive` / `11000`. Test cost ≈ **0.2 Copilot credits**, ~3.8s.

## Files
- `workflow.json` — Logic Apps definition (trigger + Run a prompt + Respond). The prompt is referenced by
 `recordId` (the msdyn_aimodel GUID) — **the prompt object is NOT in this JSON**; it must exist in the target env.
- `metadata.yml` — workflow metadata (category 5, scope 4 = Organization, stateCode 1 = activated).

## Env / IDs (demo org)
- Org `${DEMO_ORG_URL}`, solution `psq_demo`.
- Agent `PSQ Recruiting Assistant` = `${AGENT_ID}`.
- Flow (workflow) = `${WORKFLOW_ID}` (solution componentType **29**).
- Prompt (msdyn_aimodel) = `${MODEL_RECORD_ID}` (solution componentType **401**).

## How to reuse
- **Portal-first** authoring: in Copilot Studio open the agent → **Tools → Add tool → Agent flow → New** (or the
 env **Flows** tab → **New agent flow**). Build trigger inputs, add **Run a prompt → New custom prompt**, map the
 trigger inputs, then **Respond to the agent** = the prompt's `Text` output. Publish, then add it as a tool.
- **Headless maintenance:** after the portal build, `pac copilot clone --bot <id>` pulls the flow into
 `workflows/<FlowName>-<id>/` (this `workflow.json` + `metadata.yml`); edit + `pac copilot push`. The custom
 prompt itself is an AI Builder object (portal-authored), referenced by GUID — recreate it in a new env first.
- **ALM:** add BOTH the workflow (ct 29) and the prompt msdyn_aimodel (ct 401) to the delivery solution, then publish.
