# Code interpreter demo — candidates.csv (White Ranger)

Sample structured-data file for the **Copilot Studio code interpreter** demo on the **PSQ Recruiting Assistant**.
Validated the agent ran Python (pandas) over this CSV and returned deterministic analytics
(avg AnnualComp by SeniorityBand, count per Stage, IQR salary-outlier detection) — see
`PowerSquad/agents/08-white-ranger-copilot-studio.md` "Code interpreter" VALIDATED note.

## Enable + run
- Agent **Settings → Generative AI → File processing**: **File uploads ON** + **Code interpreter ON** → Save → Publish.
- Test pane: **paperclip (Upload file)** → pick this CSV → ask e.g.
 *"Analyze this candidates CSV with code: average AnnualComp by SeniorityBand, count per Stage, flag salary outliers."*
- The agent shows a **Code (Preview)** tool card (Python), returns tables, and offers chart/export.
- Limits: 16 MB/file, max 10 files; premium generative tool (Copilot credits).

## The data
Generated from the real `psq_candidate` rows (demo org). Columns: FullName, Email, MonthlySalary, AnnualComp, Stage,
SeniorityBand. Contains a deliberate **outlier** (Ava Chen, AnnualComp 540,000) so the IQR step has something to flag.
Regenerate from Dataverse if the demo data changes (select psq_fullname/psq_email/psq_salary/psq_annualcomp/psq_stage,
derive SeniorityBand from salary band).
