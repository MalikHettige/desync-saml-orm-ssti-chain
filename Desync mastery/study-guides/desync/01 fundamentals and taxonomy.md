# 01 — Fundamentals & Primitive Taxonomy

## Concept

HTTP request smuggling exists because HTTP/1.1 has two independent ways to tell a server where a request body ends: `Content-Length` (a byte count) and `Transfer-Encoding: chunked` (a self-terminating stream). When a request passes through two systems that parse it independently — a CDN/proxy/load balancer as the front end, an application server as the back end — and those two systems disagree about which header governs, or how to interpret it, the byte stream desynchronizes. One system thinks a request ended; the other thinks it's still reading. The attacker controls exactly where that disagreement occurs, which lets a smuggled fragment of a second request get processed as part of the next unrelated user's request.

This is not one bug — it's a family of primitives, each defined by which header each system trusts.

## Full primitive taxonomy

Master all of these, not just the first two — most hunters stop early and that's exactly where the competition thins out.

**CL.TE** — Front end uses Content-Length, back end uses Transfer-Encoding. The classic. Front end forwards a fixed byte count that includes a smuggled request; back end reads chunk-by-chunk and stops early, leaving the smuggled bytes to prefix the next request.

**TE.CL** — Reverse of the above. Front end honors Transfer-Encoding, back end honors Content-Length. Requires crafting a chunked body whose declared Content-Length undercounts it.

**TE.TE (obfuscated Transfer-Encoding)** — Both systems nominally support chunked encoding, but one can be tricked into *not* recognizing the header via obfuscation — extra whitespace, header casing, duplicate headers, a bogus parameter appended to the value. One system treats it as TE, the other silently falls back to CL. This is where most real bugs live today, because it survives basic "does this system support both headers" audits.

**CL.0** — Back end ignores Content-Length entirely on requests it doesn't expect a body on (common on GET-only or redirect-handling backend routes) while the front end still forwards based on it. A body the front end thinks belongs to one request gets reinterpreted as the start of the next.

**TE.0** — Same idea, back end ignores Transfer-Encoding under specific conditions (some backends only honor it on POST, or only above a size threshold).

**0.CL / 0.TE** — Front end forwards with no length-governing header interpretation at all (pass-through mode) while downstream systems still apply CL or TE logic — desync introduced purely by the front end's passivity rather than active disagreement.

**Chunk-extension abuse** — The chunked-encoding spec allows arbitrary `;extension` data after a chunk-size number. Some parsers strip it, some don't, some choke on specific characters within it. This becomes a secondary obfuscation layer that can convert what looks like a safe TE.TE configuration into an exploitable one.

## Lab task

In `labs/desync/`, stand up a minimal reverse-proxy pair (e.g. nginx front + a simple backend you control) and deliberately misconfigure it to reproduce each primitive above, one at a time. For each: capture the raw request/response, confirm the desync with a timing or response-splitting differential, then fix the misconfiguration and confirm the primitive no longer works. Building the patched version yourself is what makes the mechanism stick.

## Log fields for each primitive lab

- Initial hypothesis (which header mismatch you're testing)
- Raw request bytes sent (not Burp's normalized view — see module 6)
- Response(s) received, including timing
- Why the front end and back end disagreed
- Conditions required (specific server/version/config)
- Confirmed impact in the lab
- How you'd fingerprint this pairing on an unfamiliar live target
- The fix, and why it closes the gap
