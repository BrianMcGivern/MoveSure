# MoveSure

Capstone grading prototype — the agent loop and the eval harness, in one page. A scale model proving the verification logic works; not the tenant product UX.

## What this is

Moving into a New York apartment is a coordination problem. Four parties have to agree — the tenant, the mover, the super, and management — and nobody holds the whole picture. The tenant, the one with everything at stake, has no authority over any of them and can't see whether any of it is actually done. You don't learn what a building requires until you're already moving. Get one thing wrong and the crew is turned away, the deposit is lost, the meter is running, and you're on the street with the boxes.

MoveSure verifies the four things your building requires — insurance, elevator, fees, house rules — and gives you certainty 48 hours before move day, so one missed document can't leave you on the street with the boxes.

`index.html` is the entire thing: one self-contained HTML page (no server, no database, no build step, no API keys anywhere in the repo — verified 2026-10-13). It walks one NYC move-in through four verification gates (G1 mover insurance, G2 elevator booking, G3 fees, G4 house rules) with agents verifying and humans approving via one-tap links. The agent pre-checks and captures; the designated human's word completes each gate. Nothing verbal flips a gate, and "verified" never means "approved" — only the super's tap authorizes unloading.

Live demo: **capstone.productfaculty.com/movesure**

Demo Day video (3:57): https://www.loom.com/share/5f7d8b9edd68487dbc71dc5350811fd4

## How to run it

1. Open `index.html` in a browser (Chrome recommended).
2. The end-to-end walkthrough runs from the main view, one click per case chip.
3. That's it. Nothing to install, no credentials needed.

## How to run the eval harness

1. Open `index.html` → go to the **Evals** tab.
2. Each golden case has its own **Run** button. Run a case, read the verdict and the trace it produced.
3. Compare against the expected verdict — the expected answer for every case was written by the author *before* any run (see the testing report). Ask one question: *is this the answer I wrote down, reached the way I said it should be reached?*
4. Pick the human verdict: **Pass / Needs work / Fail**. The agent never grades itself; the verdict is the human's judgment.
5. A case passes only if the verdict matches AND every trace field is in the run log.
6. The scoreboard at the top tracks passes; the improvement card (before / change / after) and the limitations live below.

**Two modes:** sample mode shows prepared examples, labelled as samples — it proves the harness wiring works, no key needed. Live-AI mode hands each case to a real model; the page asks for your own key in the key box (it lives in that tab only and is never stored in the repo).

**No LLM judge by design:** every golden-case criterion is exactly checkable (exact name match, coverage ≥ minimum, trace steps present or not), so the harness asserts verdict and trace deterministically rather than asking a model to grade a model.

## Eval results — the official run

**V8, October 25, 2026: 20 Pass / 0 Needs work / 0 Fail, zero false approvals.**

Run locally, on the author's own Anthropic API key, with live AI (Claude Opus 5.5) on both prompts — the worker and the independent blind auditor. All 20 golden cases, run 15:53–16:17 ET. The two readers agreed on all four gates in every case. The full judged export is `movesure-v8-official-run-copy-results-2026-10-25.txt`.

**The before:** V5, run the same day on the same key, scored 9 Pass / 11 Needs work / 0 Fail, also zero false approvals. The dominant failure was the two readers disagreeing in 8 cases, mostly on G4. The full judged export is `movesure-official-run-copy-results-2026-10-25.txt` (judgments at the bottom of the file).

**What changed between V5 and V8:** five sentences added to both prompts (worker and auditor) — classify every artifact by which gate it belongs to before judging it; G4 has one result (a readable matching rules document is CHECKS COMPLETE, only the tenant's own tap completes it); a fully matching receipt is terminal, so G3 never waits for a second step; a captured confirmation counts only when the sender, the window, and every exception condition all hold; a hold stays in its own gate. The export was also made to print both readers' verdicts per gate, so a disagreement is visible, and the GC-20 replay was made to create the fresh booking draft when none exists. Every one of the 11 needs-work cases flipped.

**The safety bar:** zero false approvals. A false approval means a truck turned away at the door. When the two readers disagree, the gate freezes (REVIEW NEEDED) instead of passing — that is the system working, not failing.

## Where the results are

- The page's **Copy results** button gives the eval table, the improvement card, and the limitations as text.
- **Download the log (CSV)** downloads `run-log.csv` — the full run history.
- The per-case written record lives in the testing report in the capstone goal files (one row per golden case: expected result vs. the human's verdict).
- The golden set itself: `movesure-golden-eval-cases-2026-10-11.md` — GC-01–GC-20, each declaring fixture, policy versions, expected verdict, and required trace. The full pass rule is defined there.
- Official run exports in this repo: `movesure-v8-official-run-copy-results-2026-10-25.txt` (20/20) and `movesure-official-run-copy-results-2026-10-25.txt` (V5, 9/11).

## Business model

The tenant is the user and the building is the buyer. The cost is shared: the building pays a $50 monthly base and the tenant contributes $15 per move, folded into the move-in fee they already pay. The cost per move is about $1 of AI calls plus hosting and email. At 8 moves a month for one building, revenue is $170, cost about $58, margin about $112. The same four gates run for every building, so it scales without adding staff.

## Design decisions

- **Model:** Claude Opus 5.5 for both prompts. The expensive model only where the input is messy (the COI); the cost is about $1 per move against $15 per move in revenue. Both readers need to be strong, or the disagreement freeze fires on capability gaps rather than real disagreements.
- **Temperature:** not set anywhere in the build; runs use the model default. The prompts are tight and the disagreement freeze is the safety net that makes that safe.
- **Output:** short, structured JSON (verdicts plus citations), not free-form prose. Structured output is what makes the worker-vs-auditor comparison and the citation guard possible.
- **RAG:** none. The kit hands the model exactly what it needs — the case packet, the building profile, the five policies — and nothing else. No chat history, no other moves' data.
- **Memory:** none across cases. Each move is isolated. The system keeps an audit trail (who tapped what, when) but raw documents aren't kept longer than the move needs.
- **Guardrails:** (1) citation guard — every gate must cite one of five exact policy keys or the result is withheld; (2) two-reader disagreement freeze — if worker and auditor disagree, the gate freezes instead of passing; (3) nothing verbal flips a gate — only a designated human's tap or a terminal receipt can complete it; (4) verified never means approved — only the super's tap authorizes unloading.
- **Human-in-the-loop:** the agent pre-checks, the human completes. Never fully automated approvals.

## Repo rules (per reviewer guidance)

- **No API keys in the repo.** Verified 2026-10-13 — none in the deliverable or the build sources; any runtime key is pasted into the browser tab only.
- **README says how to run the harness and where the results are** — that's this file.

## Limits

- A prototype: evidence is made up and arrives in one click, not over days.
- No real inbox, links, or emails leave the page. Sends are recorded in the run log only.
- The AI reads only what is on the page: one building profile and the policy sections.
- Live AI can misread documents; both checkers may share a blind spot. Only fictional evidence should be uploaded.

---
*MoveSure is the capstone for the Product Faculty AI PM fellowship (Cohort 11). Repository goes public Nov 1, 2026, after capstone submission.*
