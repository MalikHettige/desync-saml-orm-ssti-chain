# Master Checklist (Reconciled) 
# Implementation adapted from external references and modified for this project.

**What this document is:** the desync/SAML/ORM/SSTI chain material, corrected and folded into your actual live plan — not a competing 4-month track with its own deadline.

**Your real plan is the three-weapon plan (IDOR, Auth, Business Logic) with a hard first-submission deadline of July 14.** That deadline does not move for this document. Everything below is enrichment that happens inside your existing 2:30–4:30 PM study block, not a parallel program.

---

## Why This Version Exists

The previous version of this checklist had three problems serious enough to fix before you follow another line of it:

1. **It misdefined Golden SAML.** Golden SAML specifically means forging tokens with a *stolen IdP private signing key*, obtained via prior compromise of the identity provider server itself (this is how APT29 used it in the SolarWinds campaign). It is not something you find by probing public metadata on an external web target — that's a different, narrower attack surface: SAML signature-wrapping and XML-canonicalization bypasses against a service provider's validation logic. That's real and bounty-findable. "Golden SAML" as a general hunting target mostly isn't. Everything below uses the corrected name.
2. **The bounty math was fantasy.** $30k–$75k for a chain, $5k–$15k for a single stage — not representative numbers. Your three-weapon plan's own math ($300–$1,500 for an IDOR, $500–$5,000 for a business logic finding) is the realistic range. This document now uses that math.
3. **It had no live-submission gate for four months.** You've already named "permanent preparation mode" as your specific, recurring risk and built a hard July 14 deadline into your real plan to prevent it. A study track with zero submissions until week 17 recreates the exact failure you already caught yourself in once. This version doesn't get its own deadline — it lives inside the schedule that already has one.

---

## Where This Fits Into Your Actual Week

- Chain study happens in the existing **2:30–4:30 PM study/CTF block**, on the days that block isn't already committed to Hacker101 CTF or API/JS/recon practice. It does not get separate hours.
- **Weekends are still fully consumed by the Saturday/Sunday diploma class (7am–7pm).** Nothing below assumes weekend hours. If you're reading this on a weekend, you're already off-plan — go do the class.
- Nothing here delays or substitutes for the July 14 live VDP submission on your real plan. If chain study and submission prep ever compete for the same hour, submission prep wins, no exception.

---

## Pivot Triggers (Automatic — Check Weekly)

These replace vague "watch out for X" language with conditions that force a decision, not a vibe check.

1. **If two consecutive weeks pass with zero progress on a chain stage** (not "slow progress" — zero), drop that stage from active study and mark it dormant. Revisit only after your first live bounty.
2. **If any single chain stage exceeds three logged study sessions without reaching the "can explain it to someone else" bar**, stop building toward exploitation and go back to pure concept study for that stage only.
3. **Golden-SAML-style signature/canonicalization work stays gated:** don't start it until desync and ORM stages have a working local PoC each. If July 14 arrives and that gate hasn't opened, the SAML stage is dropped for this cycle, no exception.
4. **If your real plan's July 14 submission deadline is at risk from chain study eating into hunt-block or report-writing time**, chain study is paused immediately, full stop, regardless of how close a PoC feels.
5. **If you get three or more N/A or duplicate rejections on live submissions**, pause all chain study for one week and run a report-quality review instead (see below) — the problem is more likely your reporting than your target selection.
6. **If the Saturday/Sunday diploma schedule changes or the class ends**, re-run this whole checklist's time budget before assuming the freed hours go to the chain — check them against your real plan's hunt and report-writing blocks first.

---

## Phase 1: Foundation Study (folds into 2:30–4:30 PM block, ~2 weeks of that slot)

### Study Objectives — corrected framing

- [ ]  **Desync:** Why front-ends and back-ends disagree on request boundaries; the actual current technique landscape (CL.TE/TE.CL are largely patched on major CDNs now — the productive techniques today are 0.CL, CL.0, TE.0, and client-side desync); what Response Queue Poisoning is and why it's the real target outcome, not cache poisoning alone.
- [ ]  **SAML (corrected):** Why SSO metadata exposes attack surface; XML canonicalization collisions and signature-wrapping bypasses against a service provider's validation. Treat true Golden SAML (stolen IdP key) as a separate, much narrower topic you study for awareness, not as something you're hunting for.
- [ ]  **ORM:** Type confusion in query filters (e.g. how an operator like `$ne` can flip a filter into "match anything"), and how this varies by ORM/database combination.
- [ ]  **SSTI:** How template engines evaluate expressions, blind exploitation via timing or error-based side channels, and what a realistic gadget chain to RCE looks like versus a lab toy example.

### Resources

- Study guides you already have: desync, SAML(-bypass), ORM, SSTI
- Primary research: James Kettle's desync research (read chronologically — 2019 origin paper through the 2025–2026 papers, since each builds on the last), the actual SAML signature-wrapping/canonicalization research (not "Golden SAML" search terms), elttam's ORM injection writeup, blind SSTI research on error/timing-based detection
- PortSwigger Web Security Academy has dedicated labs for the current desync techniques (0.CL, CL.0, response queue poisoning, client-side desync) — use these over generic tutorials, they're kept current

---

## Phase 2: PoC Building (folds into 2:30–4:30 PM block, pace: as long as it takes per stage, capped by Pivot Trigger 2)

**Realistic pacing note:** the original version budgeted one week per stage including a full working exploit. That's compressed for a first pass at any of these. Better default: don't set a week number, set the trigger — you move to the next stage when you can explain the current one to someone else and have a local PoC working at least once, whichever takes longer, capped at three sessions per Pivot Trigger 2.

- [ ]  **Desync:** Set up nginx + a back-end with different parsing behavior. Get a confirmed 0.CL or CL.TE condition via the single-packet attack (Turbo Intruder), not naive send-and-wait timing.
- [ ]  **SAML signature/canonicalization:** Set up an IdP + SP pair, extract metadata, and attempt a signature-wrapping or canonicalization bypass. This is the stage gated behind Pivot Trigger 3 — don't start until desync and ORM each have a local PoC.
- [ ]  **ORM:** Set up a Node.js + Mongoose (or equivalent) app, seed test data, and demonstrate an operator injection that bypasses a filter or leaks unauthorized data.
- [ ]  **SSTI:** Set up a template-engine-backed app, demonstrate blind detection via a timing or error side channel, then escalate to a controlled RCE PoC.

### Success Criteria (corrected — "reliable" means "understood," not "100%")

- [ ]  You can explain, without notes, why each vulnerability exists at the protocol/parsing/type level — not just which payload works.
- [ ]  Each stage has at least one working local exploitation, once, that you fully understand rather than copy-pasted.

---

## Phase 3: Integration (only after Phase 2, and only if hunt-block/submission work isn't competing for the hour)

- [ ]  Diagram how the stages *could* connect on a real target — but hold this loosely. The honest version: desync can smuggle a request past a front-end check the back-end assumes was already enforced; it does not generically hand you access to "the trusted context" or a path into SAML/ORM/SSTI. Any real chain on a live target will be specific to that target's architecture, discovered during recon — not something you pre-build and go looking for.
- [ ]  Treat a genuine end-to-end chain on a real target as a bonus outcome, not a milestone you're required to hit. Most real bounty income in this class of bug comes from single-stage findings reported independently, not four-stage chains — chains are rare even for experienced hunters.

---

## Phase 4: Hunting (only runs inside your real plan's existing hunt block — 4:30–6:30 PM — never as separate hours)

### Per-target checklist (unchanged in spirit, corrected labels)

- [ ]  SAML-based SSO in scope (for signature/canonicalization work — not "Golden SAML," which needs prior IdP compromise you won't have)
- [ ]  Reverse proxy or CDN in front of the app (desync surface)
- [ ]  Public API with JSON endpoints (ORM injection surface)
- [ ]  User-generated content rendered through templates (SSTI surface)
- [ ]  Passes your existing 5-signal program screen (response efficiency, time to first response, resolved report count, real bounty table, meaningful scope)

### Realistic success criteria

- [ ]  You've identified a handful of targets matching one or more of these surfaces — not a required count of 10+ before you're "ready."
- [ ]  One confirmed single-stage finding (desync, SAML bypass, ORM injection, or SSTI alone) is a complete, submittable report. Don't hold a valid single-stage finding hostage waiting for a full chain.

---

## Phase 5: Reporting — corrected bounty expectations

Use these ranges, not the original document's numbers:

1. **Single confirmed finding** (any one stage, reported alone): realistic range roughly **$300–$3,000** depending on severity and program tier — in line with your real plan's IDOR/business-logic math, not a separate scale.
2. **Two or more stages genuinely chained on a real target:** severity and payout scale with demonstrated impact (e.g., confirmed RCE chains typically land at the higher end of a program's table) — but don't plan your finances around a specific number here. Treat any chain payout as upside, not a target.
3. If you only get one stage working: report it. A real, well-documented single-stage finding is worth more than an unfinished chain.

### Reporting checklist (unchanged — this part was fine)

- [ ]  Clear, step-by-step reproduction
- [ ]  Evidence (screenshots/short clip) of the actual impact, not just the technique
- [ ]  Severity justification
- [ ]  Remediation suggestion
- [ ]  Professional, factual tone — no need to oversell a single-stage finding as part of a chain if the chain isn't actually demonstrated

---

## Report-Quality Review (new — this was missing entirely)

Triggered automatically by Pivot Trigger 5 (three or more N/A/duplicate rejections), but worth doing once proactively before your first submission regardless:

- [ ]  Have someone else (or a fresh read the next day) check whether your report explains impact clearly to someone unfamiliar with the target
- [ ]  Confirm your PoC steps work from a clean state, not just in your own browser session with existing cookies/context
- [ ]  Check whether the report over-claims severity relative to demonstrated impact — this is a common, avoidable cause of downgrades

---

## Tools (unchanged, still accurate)

- Burp Suite (HTTP Request Smuggler extension — use a current version; older versions miss the desync techniques that actually work in 2026) + Turbo Intruder
- Python 3 + requests/socket for manual crafting
- Docker for lab environments
- tplmap, commix, Nikto for SSTI/injection assistance — treat as confirmation aids, not primary discovery method

---

## The Honest Version of "Why This Chain Is Worth Studying"

- It's genuinely lower-competition than IDOR — that part holds up.
- It is **not** a faster or more reliable path to your first bounty than your three-weapon plan. Treat it as depth you build in parallel with your existing schedule, cashed in opportunistically when a target's surface happens to match, not as the plan itself.
- The single biggest risk with this material isn't the technical content — it's letting "this is more advanced and impressive" pull hours away from the plan that actually has a submission deadline. Trigger 4 exists specifically to catch that.

---

## Before You Start

- [ ]  You understand this document is subordinate to your three-weapon plan, not a replacement for it
- [ ]  You've corrected any notes you'd already made using the old "Golden SAML" framing
- [ ]  You've mentally reset the bounty numbers to the realistic range
- [ ]  You know all six pivot triggers without re-reading them
