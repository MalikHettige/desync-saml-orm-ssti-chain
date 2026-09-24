# 00 — Course Map

## How to use this course

Each module has three parts:
- **Concept** — what the primitive/technique is and why it works
- **Lab task** — what to build in `labs/desync/` to prove you understand it, not just read it
- **Log fields** — matching your existing mastery-bar method: initial hypothesis, requests/responses that mattered, why the server accepted the malformed request, required conditions, impact, detection clues, how to test the same class on a new target, remediation

A module isn't complete until you can re-solve its lab without notes and explain the mechanism out loud.

## Sequencing

| Order | Module | Depth needed before moving on |
|---|---|---|
| 1 | Fundamentals & taxonomy | Can explain why HTTP/1.1's dual length-specification (Content-Length vs Transfer-Encoding) creates the ambiguity, from memory |
| 2 | Client-side desync | Can explain how CSD removes the reverse-proxy-pair requirement |
| 3 | HTTP/2 downgrade seam | Can name at least 3 real-world edge products that perform this downgrade and why each is a candidate |
| 4 | Detection & safe confirmation | Can confirm a primitive with zero risk of a false positive or collateral request corruption |
| 5 | Primitive → impact playbooks | Can go from "confirmed primitive" to a specific, reportable impact without hand-waving |
| 6 | Tooling | Can craft and send a malformed request without Burp normalizing it away |

Modules 1–3 are "what exists." Modules 4–6 are "what you can actually do with it." Elite = fluent in both halves, not just the first.

## What "mastered" means for this course specifically

Not: "I read about CL.TE once."
Yes: "I can look at an unfamiliar target's response headers and infer which front-end/back-end pairing is plausible, pick the correct primitive to test for that pairing, confirm it safely, and describe the concrete impact path — in under a day, without notes."
