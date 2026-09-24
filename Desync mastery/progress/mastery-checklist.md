# Mastery Checklist

Mirrors your existing lab methodology: a module isn't complete until re-solved without notes and explained out loud, not just checked off.

## Module 1 — Fundamentals & taxonomy
- [ ] Can explain the Content-Length/Transfer-Encoding ambiguity from memory
- [ ] Built and confirmed CL.TE in lab
- [ ] Built and confirmed TE.CL in lab
- [ ] Built and confirmed TE.TE (obfuscated) in lab
- [ ] Built and confirmed CL.0 in lab
- [ ] Built and confirmed TE.0 in lab
- [ ] Built and confirmed 0.CL / 0.TE in lab
- [ ] Demonstrated chunk-extension abuse as a secondary obfuscation layer
- [ ] Re-solved at least 2 of the above without notes

## Module 2 — Client-side desync
- [ ] Can explain why CSD doesn't require a reverse-proxy pair
- [ ] Built single-server CSD lab demonstrating victim-side effect
- [ ] Can articulate CSD detection differences vs. server-side confirmation

## Module 3 — HTTP/2 downgrade seam
- [ ] Can name and explain at least 3 real edge products performing HTTP/2→1.1 downgrade
- [ ] Attempted local reverse-proxy reconstruction-logic probing
- [ ] Documented fingerprinting signals for "HTTP/2 at edge, HTTP/1.1 backend likely"

## Module 4 — Detection & safe confirmation
- [ ] Can perform timing-based confirmation with proper repetition/variance handling
- [ ] Can perform response-queue/differential confirmation
- [ ] Can articulate exactly what makes a confirmation method "safe" (own-connection-only, disposable data, rate-limited)

## Module 5 — Primitive → impact playbooks
- [ ] Playbook A (session hijacking) — full lab demonstration complete
- [ ] Playbook B (cache poisoning chain) — full lab demonstration complete
- [ ] Playbook C (security control bypass) — full lab demonstration complete
- [ ] Playbook D (queue poisoning / account takeover direction) — full lab demonstration complete
- [ ] Can write a triager-legible impact description for each, unprompted

## Module 6 — Tooling
- [ ] Comfortable with Burp + HTTP Request Smuggler
- [ ] Comfortable with Turbo Intruder for timing-precision work
- [ ] Comfortable crafting raw requests via Python sockets / openssl s_client
- [ ] Can identify what a given tool normalizes and route around it

## Recon
- [ ] Primitive matrix has at least 5 characterized front-end/back-end pairings
- [ ] Pre-qualification checklist run and internalized (fast, not a lookup exercise anymore)

## Overall mastery bar

You're ready to call this "mastered" when you can take a target you've never seen, fingerprint its edge/backend pairing in under an hour, predict which primitive is most plausible before testing, confirm it safely, and articulate the concrete impact path — all without referring back to these files.
