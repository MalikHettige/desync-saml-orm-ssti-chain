# 03 — The HTTP/2 → HTTP/1.1 Downgrade Seam (current frontier)

## Concept

HTTP/2 frames requests with explicit length prefixes — there's no Content-Length/Transfer-Encoding ambiguity within HTTP/2 itself. But almost no backend speaks HTTP/2 all the way through. The typical modern path is: client → HTTP/2 to the edge (CDN, WAF, API gateway, service mesh ingress, auth proxy) → **downgraded to HTTP/1.1** for the backend. The downgrade step is where the ambiguity gets reintroduced — the edge has to reconstruct HTTP/1.1 framing from HTTP/2's clean framing, and different products do this reconstruction differently. A request that's perfectly well-formed in HTTP/2 can be translated into something the backend parses differently than the edge intended.

This is the class the 2025-era disclosures (compromising major CDNs) were built on — not because the old CL.TE/TE.CL bugs came back, but because the entire industry moved its edge to HTTP/2 and reintroduced the same underlying ambiguity at a new seam.

## Why this matters for you specifically

This is the least-documented, most current part of the taxonomy — most public material (including a lot of what's indexed and easy to find) still teaches the 2019 model. Fluency here is a direct, current competitive edge, not a historical curiosity.

## What to look for on a target

- Any target sitting behind a CDN/WAF/API gateway/service mesh where you can infer HTTP/2 is spoken at the edge (browser dev tools protocol indicator, `Alt-Svc` headers, response header ordering quirks) but the backend is plausibly HTTP/1.1 (common in most non-Google-scale backends)
- Header smuggling opportunities introduced specifically by the reconstruction step — e.g. pseudo-headers or HTTP/2-specific framing quirks that don't have a clean HTTP/1.1 equivalent, forcing the edge to make an interpretation choice

## Lab task

This one is harder to fully replicate locally since it requires a real HTTP/2-terminating edge product. Two options: (a) spin up a local HTTP/2-to-HTTP/1.1 reverse proxy (e.g. nginx or a small custom Go/Node proxy configured this way) and probe its own reconstruction logic for inconsistencies; (b) treat this module as primarily conceptual + read-the-research until you have an authorized live target where you can test it directly — record your prediction of which edge products are likely candidates and why, so you have a rehearsed hypothesis ready the first time you meet one live.

## Log fields

- Edge product identified/suspected (or "unknown, inferred from behavior")
- Evidence HTTP/2 is spoken at the edge
- Evidence/reasoning for HTTP/1.1 backend
- Reconstruction quirk hypothesized
- Confirmation approach (lab or conceptual, note which)
- Impact if confirmed
