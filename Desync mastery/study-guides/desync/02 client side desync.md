# 02 — Client-Side Desync (CSD)

## Concept

Classic desync (module 1) requires a front-end/back-end pair that disagree — no reverse proxy, no smuggling. CSD reframes the problem: the victim's own **browser** becomes the "front end." A malicious page causes the victim's browser to send a request that the server's single-server parsing logic misinterprets, poisoning the connection for whatever the browser sends next on that same connection (a legitimate request to the real site, e.g. via a redirect or subsequent navigation).

This matters because it works against architectures with **no proxy pair at all** — a single server, correctly matched against itself, can still be vulnerable, because the desync is introduced by the browser's own connection-reuse behavior rather than by two disagreeing systems. This is the reason CSD is where the field moved after the 2019 classics got broadly patched: it reopens targets that were "fixed" against the old model.

## Why this is a competition edge

Most hunters who know desync at all know the 2019 CL.TE/TE.CL model and stop there. CSD requires understanding browser connection-pooling behavior and cross-origin request sequencing, which is a genuinely different mental model — fewer people have internalized it, and it applies to a category of targets (single-server, no visible reverse proxy) that classic-desync hunters skip entirely because they assume there's no pair to desynchronize.

## Lab task

Build a single-server lab (no separate front/back end) and construct a page that demonstrates connection reuse causing a subsequent same-connection request to be misparsed. Confirm the victim-side effect: what does the *next* request on that connection receive that it shouldn't.

## Log fields

- Which browser connection-reuse behavior you're exploiting
- The malicious page's request sequence
- What the victim's subsequent request received
- Why a single-server target was still vulnerable
- Detection approach for CSD specifically on a live target (this differs from module 4's server-side confirmation — note the difference explicitly)
- Remediation at the server config level
