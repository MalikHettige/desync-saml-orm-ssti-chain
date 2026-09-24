# Target Fingerprinting & the Primitive Matrix

## Purpose

Given the 1-week-per-target hunting cadence, you cannot afford to discover mid-week that a target's architecture doesn't support the primitive you're testing for. This is the pre-qualification step that happens *before* you commit a target-week to desync specifically — ideally during the VDP/prep phase, building the matrix below, so live hunting is fast lookup rather than cold analysis.

## Fingerprinting signals (fast, non-invasive)

- **Server/edge headers**: `Server`, `Via`, `X-Cache`, `CF-Ray` (Cloudflare), `X-Amz-Cf-Id` (CloudFront), `X-Akamai-*` — identify the edge product directly when present
- **`Alt-Svc` header presence** — signals HTTP/2 or HTTP/3 support at the edge
- **Response header casing/ordering quirks** — different backend frameworks have distinctive default header ordering (useful for guessing backend stack when edge headers are stripped)
- **Timing baseline** — establish normal response time for a known-clean request before attempting any differential confirmation (module 4 depends on a clean baseline)
- **Behavior on malformed-but-not-malicious requests** — e.g. a request with both Content-Length and Transfer-Encoding present but consistent; does the target 400 immediately (suggests strict, less promising) or process it (suggests lenient parsing, more promising)?

## The primitive matrix

Maintain this as a living reference — every target you fingerprint (in VDP, later in BBP) adds a row. This becomes your fastest recon step once live hunting starts, because you'll often recognize a front-end/back-end pairing you've already characterized.

| Front end (edge) | Backend (inferred) | Primitives most plausible | Notes |
|---|---|---|---|
| _(fill in as researched)_ | | | |

Suggested starting research targets for filling this in (from public research and your own lab work, not live targets): common nginx configurations, common Apache configurations, major CDN default behaviors as described in public disclosures (module in `resources/`), and your own lab pairings from modules 1-3.

## Target pre-qualification checklist (run before committing a target-week)

- [ ] Edge product identified or strongly inferred
- [ ] HTTP/2-at-edge vs HTTP/1.1-only determined
- [ ] Matrix lookup performed — matching or similar pairing already characterized?
- [ ] Program scope/rules confirmed to allow this testing method
- [ ] Clean timing baseline captured
- [ ] If no matrix match: is this target worth spending original research time on, or should it default to your IDOR/business-logic base layer instead this week?

That last question is the actual point of this document — it's what turns "desync is my specialty" from a liability (dead-end risk on the wrong target) into an asset (fast go/no-go decision that protects your target-week budget).
