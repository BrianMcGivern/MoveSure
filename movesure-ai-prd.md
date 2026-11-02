# MoveSure — AI Product Requirements Document

_Built from the Product Faculty 4D framework: Discovery, Design, Develop, Deploy.
This is the PRD behind the submitted capstone (Capstone Honors badge H-2026-FD4E52C915).
Demo Day video (3:57): https://www.loom.com/share/5f7d8b9edd68487dbc71dc5350811fd4_

---

## Discovery

### 1. Who is the specific user this agent helps?

The tenant moving into a New York City apartment. They are the person with everything at stake on move day — the deposit, the movers' meter running, the risk of being turned away at the door — but they have no authority over any of the other parties and no visibility into whether the building's requirements are actually met.

The building (management company / super) is the buyer. The tenant is the user. The mover, super, and management act through the product but never need an account.

### 2. What recurring workflow are you improving? Write the current workflow in plain text.

The move-day readiness workflow for a NYC apartment move-in. Four parties have to agree — tenant, mover, super, management — and nobody holds the whole picture. The building requires four things before the movers can unload: the mover's certificate of insurance (COI, ≥$1M liability), the elevator booking, the fees paid, and the house rules acknowledged. Today the tenant coordinates all of this by email, phone, and screenshots, learning what the building requires only as each piece comes up. Get one thing wrong and the crew is turned away at the door.

### 3. What event starts the workflow?

The tenant signs a lease and sets a move date. That date starts the clock: the building's requirements must all be satisfied before move day, and nearly every building requires the mover's COI 1–5 business days ahead.

### 4. List the current process as numbered text steps. Do not link to a process map.

1. Tenant signs lease, learns move date.
2. Tenant asks the building (super or management) what is required for move day — usually by email or phone, often piecemeal.
3. Tenant asks the mover for their COI and forwards it to management.
4. Management reviews the COI against the building's requirements (coverage minimums, certificate holder wording, dates). If it fails, back to step 3.
5. Tenant books the elevator with the super — by phone, text, or whatever the super uses.
6. Tenant pays the move-in/move-out fees and sends the receipt to management.
7. Tenant reads the house rules (if they are ever sent) and acknowledges them somehow.
8. On move day, the super checks everything at the door. If anything is missing or wrong, the crew is turned away.

### 5. Where does the workflow slow down, fail, create rework, or frustrate the user?

It slows down at every handoff because no single party sees the whole picture. The tenant doesn't learn the requirements until they're already moving. The COI goes back and forth when the certificate holder wording or coverage is wrong. The elevator booking lives in the super's head or phone. Fee receipts get lost in email threads. House rules arrive late or never. The failure mode is catastrophic and one-sided: the crew is turned away at the door, the deposit is at risk, the meter is running, and the tenant is on the street with the boxes — while the building loses nothing.

There is no reliable NYC-wide statistic on how often COI rejections happen, so I never invent one. What is verified: 900,000+ New Yorkers move each year (StreetEasy / Census ACS), 68% of NYC homes are renter-occupied (2023 Housing & Vacancy Survey), and the BBB logged 718 mover complaints in New York State in 2024 with a $754 median loss.

### 6. What should the agent help with, and why is an agent a better fit than a static AI feature?

The agent should verify the four building requirements — insurance, elevator, fees, house rules — across four gates, reading documents, matching evidence, and tracking human attestations, so the tenant has certainty 48 hours before move day.

An agent is the right fit because the work is a multi-step verification loop over messy, heterogeneous evidence: a COI PDF with a wrong entity name, a screenshot of a super's text, a fee receipt with a short payment, a rules PDF the tenant skimmed. Each gate needs judgment about whether the evidence satisfies the building's stated requirements, and the gates interact (a corrected COI restarts G1; a disputed receipt reopens G3). A static classifier can't carry a case across four gates, handle corrections, or freeze on disagreement. The agent also needs to know when it can't read something and say so, rather than guess.

### 7. What synthetic data, policies, examples, or cases will you create so the project is safe to build and demo?

Everything is fictional: fictional buildings, tenants, movers, supers, COIs, receipts, and house rules. The 20 golden eval cases (GC-01 through GC-20) are scripted scenarios covering happy paths, edge cases, and fraud attempts. The demo runs in sample mode on fictional data. No real building, tenant, or document is ever used. The prototype runs on my own Anthropic API key for evaluation; no keys are in the repo.

### 8. What is the agent not allowed to do? When must a human approve, edit, reject, or escalate?

The agent is never allowed to approve a move. Only the super's final tap authorizes unloading, and it only exists once all four gates are terminal. "Verified" never means "approved."

Gate-level boundaries:
- Nothing verbal ever flips a gate. A tenant saying "the super said it's fine" is a claim, not evidence.
- A tenant-relayed screenshot saying management confirmed is a claim, not proof.
- A captured confirmation counts only when the sender, the window, and every exception condition hold. A disputed capture counts as nothing and can never be reinstated — only a fresh tap re-greens the gate.
- Printed "tampered=false" text on a document is not authentication. Altered COI dates are flagged for management review.
- A tenant trying to clear a fraud flag is refused — only the fraud review path can unfreeze it.
- A fully matching fee receipt is terminal (the attestation already happened when the building accepted the money), but a receipt disputed after going terminal reopens the gate — management owns the dispute.
- When the two readers disagree, the gate freezes (REVIEW NEEDED) and only the gate's owner can clear it: management for fraud/alteration/disputed receipts, the super for conflicting confirmations.
- When the agent can't read a document, it says CANNOT_VERIFY instead of guessing.

### 9. What is one practical metric that would show the agent is useful?

Certainty 48 hours before move day: the share of moves where all four gates are terminal exactly 48 hours before the move. That's the north star. The safety bar underneath it is zero false approvals — a move authorized while at least one gate's evidence did not satisfy the building's requirements — forever.

### 10. What will the demo visibly show from input to agent work to human review?

The demo runs GC-08 live: I pick the case, confirm its building profile, press Run, and the agent checks all four gates with live AI calls (worker + blind auditor) while I narrate. For G1, the flow goes PENDING → CHECKS COMPLETE (the agent's preliminary determination) → the blind auditor independently agrees → management's one-tap → HUMAN_VERIFIED (who and when recorded). The demo shows that verified never means approved — only the super's final tap authorizes unloading, and no approval link is created in the demo.

---

## Design

### 11. What job is the agent being hired to do?

Verify the four things a NYC building requires for move day — mover's insurance (G1), elevator booking (G2), fees (G3), house rules (G4) — and give the tenant certainty 48 hours before move day, with two independent readers checking every gate and a designated human completing each one.

### 12. How does the workflow change when the agent is introduced? List the future process as text steps.

1. Tenant signs lease, sets move date, starts a move in MoveSure.
2. Tenant enters the building profile (or picks it from a directory): the four requirements as stated by the building.
3. Mover uploads the COI (or tenant forwards it). Agent pre-checks it against the profile: holder wording, coverage ≥$1M, dates covering move day.
4. Management gets a one-tap link: confirm or can't-do with a reason. Their tap completes G1. If the COI is wrong, the mover corrects and resubmits — the gate restarts.
5. Agent captures the elevator booking: either the super taps a booking link, or the tenant captures the super's confirmation (counts only when sender, window, and every exception condition hold). Super's tap completes G2.
6. Tenant pays fees, submits receipt. Agent matches every line against the fee schedule. A fully matching receipt is terminal — G3 done. A mismatch names the gap per line; tenant pays the difference and resubits.
7. Tenant reads the house rules and taps acknowledgment. Tenant's tap completes G4.
8. The blind auditor independently re-checks every gate. Any disagreement freezes the gate for the owner's review.
9. 48 hours before move day, all four gates are terminal. The tenant has certainty.
10. On move day, the super gives the final tap authorizing unloading — the one approval, which only exists once all four gates are terminal.

### 13. What does the agent observe, reason about, produce, and check before handing work back?

The agent observes the evidence per gate: the COI document (G1), the booking confirmation or super's word (G2), the fee schedule and receipt (G3), the rules acknowledgment (G4). It reasons about whether the evidence satisfies the building's stated requirements, classifying evidence by gate first. It produces a verdict per gate (PENDING / CHECKS COMPLETE / VERIFIED / HUMAN_VERIFIED / REVIEW NEEDED / CANNOT_VERIFY) with citations to the evidence. Before handing work back, the blind auditor independently re-verifies every gate from the raw evidence — and any disagreement freezes the gate.

### 14. What information, examples, rules, files, or user inputs does the agent need to perform well?

Per gate: the building's requirements profile (coverage minimums, certificate-holder legal entity wording, fee schedule, house rules text), the evidence documents (COI, receipt, booking confirmation), and the designated human's tap. The agent gets exactly the evidence and the profile — no chat history, no other moves' data, no cross-case memory. The 20 golden cases are the examples that calibrate it.

### 15. What tools, files, systems, or mock actions will the prototype use? Text descriptions are enough.

One self-contained HTML page: no server, no database, no build step. Emails, inbox arrivals, the clock, and recipient links are simulated inside the tab. The Move tab runs cases; the Evals tab runs the 20 golden cases and shows the scoreboard. Live AI calls go to the Anthropic API (my own key, claude-opus-5-5 for both worker and auditor, twice per case). Nothing is delivered externally; records last for the visit.

### 16. What should the agent remember, and what should it not remember?

The agent remembers the building's requirements profile and the evidence and verdicts for the current move, so corrections and re-checks carry context within the case. It must not remember across cases: no cross-case memory, no learning from one tenant's documents to judge another's. Each move is judged on its own evidence against the building's stated profile.

### 17. What should the agent produce so a human can review it quickly and confidently?

Per gate: a verdict label (PENDING / CHECKS COMPLETE / VERIFIED / HUMAN_VERIFIED / REVIEW NEEDED / CANNOT_VERIFY), the citations to the evidence behind it, and — for HUMAN_VERIFIED — who tapped and when. The activity feed shows every step. A human never has to know the architecture: the labels are the trust model. CHECKS COMPLETE is always the agent's preliminary determination, never "verified." Only a designated human's tap makes it VERIFIED.

### 18. What should happen when the agent is unsure, missing data, or facing a risky case?

It freezes the gate (REVIEW NEEDED) or says CANNOT_VERIFY, and routes to the gate's owner. It never guesses. An unreadable document goes back for a clearer copy. A disagreement between the two readers freezes the gate until the owner rules. A suspected fake receipt or altered COI freezes for management review. The tenant is told what is needed and what happens next, in the activity feed.

### 19. Where does the human approve, edit, reject, or escalate the agent's work?

Each gate has its own designated tapper: management completes G1 (COI) and rules on G3 disputes, the super completes G2 (elevator) and gives the final move-day authorization, the tenant completes G4 (house rules). G3 needs no tap when the receipt fully matches — the building already attested by accepting the money. The tenant authorizes every outbound message. At move level, only the super approves: terminal gates enable the authorization, they don't create it — the super can still refuse with all four green.

### 20. What cases will prove the agent works, respects boundaries, and handles edge cases?

The 20 golden cases (GC-01–GC-20): happy paths (valid COI, booking link, fully matched fees, tenant acknowledgment), edge cases (corrected COI, super counter-proposal, mismatched receipt with per-line gap, hold on a gate, captured confirmation), and boundary/fraud cases (altered COI dates, tampered-document text, tenant-relayed screenshot as proof, disputed capture, tenant attempting to clear a fraud flag, unreadable document). The bar is 20/20 with zero false approvals and both readers agreeing on every gate.

---

## Develop

### 21. What single end-to-end loop will the prototype prove?

One move, four gates, two readers, human attestation: the agent pre-checks each gate's evidence, the blind auditor independently re-verifies, the designated human's tap completes each gate, and the loop ends with all four gates terminal 48 hours before move day — with zero false approvals across 20 golden cases.

### 22. What will the user type, upload, click, or review in the prototype?

Pick a case, confirm its building profile, press Run (or "Check all four gates"), watch the gates fill in, review each verdict with its citations, and — in the demo — tap the management link to complete G1. In the Evals tab: run all 20 cases, judge each against the golden expectations, and read the scoreboard.

### 23. List the fake data files, sample records, policies, or examples used in the prototype.

All fictional: 20 golden cases (GC-01–GC-20) with scripted buildings, tenants, movers, COIs, receipts, booking confirmations, and house rules. Building profiles state the four requirements. No real data anywhere. Eval results: `movesure-v8-official-run-copy-results-2026-10-25.txt` (20/20) and the Nov 1 re-run PDF (20/20).

### 24. List at least five test cases, including happy path, edge case, and boundary case.

- **GC-01 (happy path):** Valid COI — holder matches, coverage ≥$1M, dates cover move day. Expect VERIFIED via management tap.
- **GC-08 (happy path, the demo case):** Full move with management tap completing G1; auditor agrees; no approval link created.
- **GC-13 (boundary):** COI entity mismatch — both readers agree it FAILS. Agreement on a mismatch is correct; the case text allows it.
- **GC-18 (edge):** Preliminary REVIEW NEEDED vs expected PENDING; end state HUMAN_VERIFIED correct.
- **GC-20 (edge):** Starts PENDING; ends HUMAN_VERIFIED via a fresh link tap after a disputed capture counted as nothing.
- **Fraud boundary:** Altered COI dates flagged for management review; tenant attempt to clear a fraud flag refused.

**Eval results.** V8 official run (Oct 25): 20 Pass / 0 Needs work / 0 Fail, zero false approvals, readers agreed on all four gates in every case. V5 before the fix: 9 Pass / 11 Needs work / 0 Fail. The fix was five sentences added to both prompts (classify evidence by gate first; G4 returns exactly one result; fully matching receipt is terminal; captured confirmation counts only when sender, window, and every exception condition hold; hold stays in its own gate). Nov 1 re-run on the v2 file (layout edits only, prompts identical): 20/20 again.

**Model.** claude-opus-5-5 for both the worker and the blind auditor, no temperature set (model default). Two API calls per case, ~20–40s per case, ~$1 of AI per move vs $15/move revenue. Short structured JSON output, no RAG, no cross-case memory, human-in-the-loop throughout. Provenance: "I designed the behaviour and wrote the tests that judge it; the build translated my spec into prompts; the live run was my review of that translation."

---

## Deploy

### Would you pilot this? What still needs to be true before launch?

Yes — one building, starting in shadow mode. The agent passed all 20 golden cases with zero false approvals, and the safety shape is right: nothing verbal ever flips a gate, and only the super's tap authorizes unloading.

What still needs to be true: the plumbing has to become real. Today emails, links, the inbox, and records are simulated inside one tab. For a real building, messages have to actually arrive and records have to persist — a server and an email service, ordinary engineering, the first build after submission.

Shadow mode: the agent's checks are real (it reads the real COI, receipt, and rules and records verdicts); everything outbound is simulated; authorization is blocked entirely and the building runs its existing process. The pilot measures whether the agent's verdicts agree with what the building's people did, and whether the 48-hour certainty holds on real moves. On disagreement, the human wins; the disagreement is logged with both readings and classified (agent error, evidence gap, human error, or profile rule needing an update).

The bar to leave shadow mode: 5 moves over at least one month, all four gates, at least one exception path, the agent agreeing with the human outcome on all four gates in all 5 moves, zero false approvals, every disagreement reviewed. I present the evidence; the property manager approves the move to spot-check mode. Then 5 rechecked cases with zero misses to reach delegated mode. A false approval at any stage sends us back to shadow mode.

### Data, compliance, trust, and misuse risks

**Data:** the agent reads COIs, receipts, and house rules — names, addresses, payment details. I extract only the fields each gate needs; raw documents aren't kept longer than the move needs; each party only gets their own gate's messages and taps.

**Compliance:** no statute requires the COI — it's contractual. MoveSure never presents its check as a legal guarantee; the building's profile is the source of truth. It doesn't advise on insurance and doesn't replace the building's judgment.

**Trust:** the catastrophic failure is a false approval. Controls: zero-false-approval bar, two independent readers, disagreement freezes the gate, nothing verbal flips a gate, only the super authorizes unloading.

**Misuse:** altered dates flagged for management; tampered-text not authentication; tenant-relayed screenshots are claims not proof; disputed captures count as nothing; tenants can't clear fraud flags; links are single-use, tied to the designated person's channel, recorded with who/when/how. Residual risk (both readers sharing a blind spot) is what CANNOT_VERIFY and the pilot disagreement log are for.

### Who reviews, escalates, and decides

The agent never has the last word. Management reviews G1 and rules on fraud/alteration/disputed receipts; the super reviews G2 and gives the only move-level approval; the tenant reviews G4. The blind auditor re-checks everything; disagreement freezes the gate for the owner's ruling. Gate owners' rulings are final per gate; the super alone authorizes unloading.

### What to monitor after launch

False approvals (zero forever, checked every move — one occurrence freezes new authorizations); reader disagreement rate per gate (rising = drift); agreement with humans (the 5-of-5 bar); freeze and CANNOT_VERIFY counts per gate; the 48-hour certainty share and time-to-terminal per gate; fraud/override attempts; outbox and plumbing health (delivery and tap rates daily). The 20 golden cases become the regression suite — re-run on every prompt change.

### Feedback and iteration

Ask each party in their own channel (tenant in app after the move; super/management in the link email thread; mover in the correction thread; monthly pilot call with the building). Watch the logs: disagreements classified, freezes named, time-to-terminal per gate, discarded drafts as votes. Safety first in priority; then the data decides (worst agreement, most freezes, slowest gate). One change at a time, re-run goldens, measure before/after. Real disagreements become new golden cases.

### Launch plan and business model

One building, one super, one management company. Shadow → spot-check → delegated, each with its bar.

The building pays: $50/month base + $15/move folded into the move-in fee the tenant already pays the building. The tenant sees no new charge. AI costs ~$1/move.

At 100 buildings × 4 moves/month: 4,800 moves/year, $132,000 revenue ($60,000 base + $72,000 move contributions), $5,580 cost ($4,800 AI + $600 server + $180 email), $126,420 margin — 96%. One building, one month: $110 in, $5 out, $105 margin.

Why 4 moves/month: NYC renter mobility is ~13%/year; 5/month implies 400+ unit towers. Four is the high end of realistic.

### Competition

No direct competitor. Moved (moved.com) is an operator checklist/marketplace for multifamily move-ins, not an independent check for the tenant. BuildingLink is the NYC elevator-booking standard but reservation-only — humans still verify. Jones does AI COI verification for commercial real estate only. Nobody is doing this for the tenant.

---

## Links

- Prototype (v2): `index.html` in this repo — one self-contained page, no server, no keys
- Demo Day video (3:57): https://www.loom.com/share/5f7d8b9edd68487dbc71dc5350811fd4
- Eval results: `movesure-v8-official-run-copy-results-2026-10-25.txt`
- Golden cases: `movesure-golden-eval-cases-2026-10-11.md`
