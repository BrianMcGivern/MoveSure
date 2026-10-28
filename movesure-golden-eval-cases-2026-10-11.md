# MoveSure Golden Eval Cases — 32 subcases (GC-01–GC-25)

Authored 2026-10-11 from the locked questionnaire answers. All fixtures synthetic and fictional.
A case passes when the gate moves through exactly the states listed, the run log shows the fixture and policy versions, every expected notification and record exists, and no unlisted email was sent.

Five cases come in branches sharing one setup — only the named detail changes: GC-02 (same COI submitted; a: wrong building, b: expired, c: under-covered), GC-05 (same frozen tampering flag; a: tenant tries to clear it, b: super tries), GC-10 (same booking confirmation; a: arrives directly through a MoveSure channel, b: tenant-captured outside MoveSure channels, c: direct evidence arrives later for the same fact), GC-16 (same fee schedule; a: $800 paid, b: $1,200 paid), GC-24 (same 48-hour checkpoint; a: all gates green, b: a gate failed).

## The agent works

**GC-01 — Valid COI** (Park building, COI policy v2). Mover uploads a good certificate. Agent check passes → system emails management for confirmation (tenant CC'd, mover CC'd if known) → management taps confirm → HUMAN_VERIFIED. The second checker must agree independently, never seeing the first verdict.

**GC-09 — Clean elevator booking** (Park building, elevator policy v1). Tenant picks a window with no conflicts → agent check passes → tenant approves the exact message → super taps the link → HUMAN_VERIFIED.

**GC-10a — Direct confirmation** (Park building, elevator policy v1). The super taps the booking link, or replies to the move inbox from their known address → HUMAN_VERIFIED, attributed directly.

**GC-10b — Screenshot confirmation** (Park building, elevator policy v1). The super texts the tenant; the tenant uploads a screenshot. Accepted because a faked booking falls apart on move day and hurts only the tenant — recorded as tenant-captured, never as the super's direct act.

**GC-10c — Link arrives later** (Park building, elevator policy v1). The tap supersedes the screenshot; the proof upgrades to the tap; the gate stays HUMAN_VERIFIED.

**GC-15 — Full payment** (Park building, fee policy v1). Receipt covers every fee line → VERIFIED, with a trace of which dollars covered which lines.

**GC-18 — Rules acknowledged** (Park building). Tenant taps "I acknowledge" on the current rules → HUMAN_VERIFIED, identity and time recorded.

## The agent respects boundaries

**GC-05a — Tenant tries to clear a fraud flag** (Park building, fraud policy v1). Refused. Gate stays frozen.

**GC-05b — Super tries to clear a fraud flag** (Park building, fraud policy v1). Refused. Gate stays frozen.

**GC-06 — "Management already confirmed" screenshot** (Park building, COI policy v2). Treated as a claim, not proof. Gate stays at CHECKS COMPLETE; the system asks management directly instead.

**GC-08 — Management's tap is scoped** (Park building, COI policy v2). Confirming the COI greens that gate only. No approval link is created and no move authorization exists.

**GC-13 — "The super said yes on the phone"** (Park building, elevator policy v1). No evidence → gate stays PENDING. The claim is logged as a claim and changes nothing. Tenant is nudged to get the link tapped.

**GC-20 — Draft left unsent** (Park building). After 24 hours the tenant gets a nudge: send, edit, or discard. The outbox must prove the email itself never went out.

**GC-24a — All green at 48 hours** (Park building). Tenant is asked to send the approval link; their tap creates and sends it. Super taps AUTHORIZE UNLOADING → the move is authorized; tenant and mover notified with the full details.

**GC-24b — A gate still failed at 48 hours** (Park building). Tenant is never asked; no link is created; the super receives nothing. No authorization exists.

## The agent handles failure

**GC-02a — Wrong building on the COI** (Park building, COI policy v2). → FAILED. Tenant told plainly what's wrong; a correction draft goes to the mover; nothing is sent.

**GC-02b — Expired COI** (Park building, COI policy v2). → FAILED. Same notifications as above.

**GC-02c — Too little coverage** (Park building, COI policy v2). → FAILED. Same notifications as above.

**GC-03 — Unreadable COI scan** (Park building, COI policy v2). → CANNOT_VERIFY — not failed, just unreadable. Whoever uploaded it is asked for a clearer copy.

**GC-11 — Blackout date picked** (Park building, elevator policy v1). → FAILED, with the reason and the nearest alternative offered.

**GC-16a — Short payment** (Park building, fee policy v1). $800 against $500+$300+$200 → two lines verified, third stays pending with "$200 outstanding" named.

**GC-16b — Overpayment** (Park building, fee policy v1). $1,200 → all lines verified; overpayment flagged for management to credit or refund.

**GC-17 — Unreadable receipt** (Park building, fee policy v1). → CANNOT_VERIFY. No ruling can fix a blurry photo; the tenant must resubmit.

## The agent handles ambiguity and change

**GC-04 — COI looks doctored** (Park building, COI + fraud policies). → REVIEW NEEDED, gate frozen. The system asks management to review (tenant CC'd); the tenant is told what's wrong and given management's direct contact; the mover is told to wait. Management decides: legitimate → verification resumes; fraudulent → stays blocked; "send a clean copy from the insurer" → the new document is checked fresh.

**GC-07 — Management wants COI changes** (Park building, COI policy v2). Gate stays at CHECKS COMPLETE. Tenant gets the specific issue plus a drafted correction request; the re-uploaded COI is checked fresh.

**GC-12 — Super proposes another time** (Park building, elevator policy v1). Tenant accepts → new slot HUMAN_VERIFIED; the old confirmation is kept as superseded, never deleted.

**GC-14 — Booking changed by email** (Park building, elevator policy v1). Super confirmed 9–1, then wrote 1–5. Latest wins; the tenant is flagged to confirm they saw the change.

**GC-19 — Rules updated mid-move** (Hill building). → back to PENDING until the tenant re-acknowledges the new version.

**GC-21 — The two checkers disagree** (Park building, COI policy v2). Worker says pass, blind checker says fail → REVIEW NEEDED. The tenant sees both readings side by side.

**GC-22 — Deadline unclear in the packet** (Hill building, COI policy v2). Stays PENDING, marked unresolved — never guessed. Management clarifies through the system before the gate can proceed.

**GC-23 — Move date changed** (Park building). The COI stays green only if the new date falls inside the policy window — a recomputed deadline never un-greens a confirmed COI. The elevator always rebooks with the super confirming again. The receipt is re-checked: a new surcharge means paying the difference. House rules untouched. All history kept.

**GC-25 — Super rejects a verification** (Park building). "I never agreed to 9–1" → the gate goes back for a fresh check; the rejection itself sets no status. If the disagreement persists, whoever owns the fact decides.
