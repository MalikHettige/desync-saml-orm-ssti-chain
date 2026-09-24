# Scope Decision: Desync-Only

This repo originally scoped a 4-stage chain: Desync → Golden SAML (XML canonicalization) → ORM type confusion → Blind SSTI → RCE.

## Why the scope narrowed to desync alone

Comparing the four links on applicability, competition, and dead-end risk given a hard time ceiling before live hunting starts:

- **Desync** — prerequisite is just "sits behind a CDN/reverse proxy," which is nearly universal in modern architecture. Broadest applicability of the four, genuinely complex (not obvious-payload territory), low competition among hunters, lowest dead-end risk.
- **SSTI** — narrower: needs a user-facing template-rendering feature specifically. Still common, kept as an opportunistic skill.
- **ORM type confusion** — narrower still: needs a loosely-typed backend (Node/PHP/Mongo-style stacks). Common but not universal; strictly-typed enterprise backends (Java/Go/Rust) are a real chunk of the target pool where this doesn't apply.
- **Golden SAML** — narrowest: requires the target to specifically run SAML-based SSO, a shrinking and heavily enterprise-concentrated slice of modern auth, often on programs already picked over by SAML specialists. Highest dead-end risk of the four.

Given a hard 3-month paid hunting ceiling with no room to spend weeks on a technique that turns out inapplicable to most targets, desync is the single highest-leverage specialty to master to elite depth. SAML/ORM/SSTI remain valuable as opportunistic bonus skills when a target happens to fit — the existing lab work for them isn't wasted — but they are no longer the primary training track this repo is structured around.

## What this changes practically

- Primary weekly training time → desync (this course)
- SAML/ORM/SSTI → deployed only when a specific live target's recon shows fit (see `recon/target-fingerprinting-and-primitive-matrix.md` for the same pre-qualification logic applied more broadly)
- IDOR/Auth/business logic remains the separate, parallel foundation track (tracked elsewhere, not in this repo) — the universal base layer that guarantees a submittable finding on any target regardless of whether desync applies
