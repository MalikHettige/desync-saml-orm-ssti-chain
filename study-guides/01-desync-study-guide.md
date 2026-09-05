# HTTP Request Desynchronization: Complete Study Guide (v2)

## Core Concept

**Desync** = making a front-end (proxy/CDN/load balancer) and a back-end server disagree about where one HTTP request ends and the next begins.

It isn’t one bug. It’s a *disagreement* between two HTTP parsers that both think they’re doing the right thing. The front-end forwards what it thinks is a clean, single request. The back-end reads more (or less) than the front-end intended, and the leftover bytes get interpreted as the start of the next request — which might belong to a completely different user.

---

## Reality Checks

- Classic CL.TE and TE.CL — the 2019-era techniques most tutorials teach — are now blocked by default on most major CDNs and reverse proxies (Cloudflare, AWS ALB, modern nginx/HAProxy configs). Finding one on a well-known target in 2026 is rare.
- The techniques that are still landing valid, paid findings today are the newer, uglier ones: 0.CL, CL.0, TE.0, H2-downgrade desyncs, chunk-extension abuse, and client-side desync (CSD). These require more setup and more patience to confirm.
- Confirmation is the hard part, not the concept. Most people who “find” a desync actually found a flaky server and a coincidence. Programs will bounce unconfirmed smuggling reports as N/A fast — this is a technique where rigor in my PoC matters more than almost anywhere else in the focus areas.
- This is a good skill to *understand* deeply. It is not a fast or reliable path for my first bounty. 

---

## The Real Timeline of Desync Research

| Year | Technique | What changed |
| --- | --- | --- |
| 2019 | CL.TE, TE.CL | Kettle’s original “Request Smuggling Reborn” research — header parser discrepancies |
| 2021 | H2.CL, H2.TE | Techniques targeting HTTP/2-to-HTTP/1 downgrading at the front-end |
| 2022 | CL.0, H2.0, CSD | Attacks against endpoints that ignore Content-Length entirely; birth of client-side (browser-triggerable) desync |
| 2024 | TE.0 | Exploiting how back-ends “dechunk” transfer-encoded bodies |
| 2025 | Chunk-extension abuse | Using chunk extensions (a rarely-implemented, rarely-tested part of the chunked encoding spec) to create discrepancies |
| 2025 | 0.CL desync | Multi-stage attack chaining a 0.CL desync into a CL.0 weaponization — this is currently one of the most productive classes for finding real targets |
| 2026 | AI-assisted discovery (“HTTP Terminator”) | Kettle built an autonomous research system that tested ~30,000 authorized targets, found roughly 700 vulnerable, and produced a “dangling-byte” refinement to Response Queue Poisoning along with a zero-day in Apache Traffic Server |

This is an actively evolving research area, not a solved/closed topic from 2019. Best if i follow PortSwigger Research directly rather than a static guide — new primitives get published most years.

---

## Attack Type Reference

### 1. CL.TE

Front-end trusts `Content-Length`, back-end trusts `Transfer-Encoding: chunked`. Front-end forwards a fixed byte count; back-end keeps reading past it looking for the chunk terminator, absorbing the start of the next request.

### 2. TE.CL

Reversed: front-end honors `Transfer-Encoding`, back-end honors `Content-Length`. Back-end reads more bytes than the front-end sent, pulling in bytes from the next request on the wire.

### 3. TE.TE

Both sides nominally honor `Transfer-Encoding`, but one of them can be tricked into ignoring it — e.g. via a malformed or duplicated header (`Transfer-Encoding: xchunked`, obfuscated whitespace, duplicate headers). Whichever side gets tricked falls back to Content-Length, recreating a CL.TE or TE.CL condition.

### 4. H2.CL / H2.TE

Specific to front-ends that accept HTTP/2 from the client and downgrade to HTTP/1.1 before forwarding to the back-end. The downgrade step can introduce a CL/TE-style discrepancy even though the original client request had no ambiguity (HTTP/2 doesn’t use Content-Length/Transfer-Encoding for framing the same way).

### 5. CL.0 / H2.0

Targets back-end endpoints that simply ignore `Content-Length` on requests they don’t expect a body for (common on static file handlers, redirects, or certain framework defaults). The front-end thinks it sent a full request; the back-end treats the body as the start of a new request.

### 6. TE.0

Targets discrepancies in how a back-end “dechunks” a `Transfer-Encoding: chunked` body — if its dechunking logic under- or over-reads relative to what the front-end expects, I get desync without touching Content-Length at all.

### 7. 0.CL

The current highest-value class for bounty hunting. A request with an obfuscated/broken `Content-Length` header (e.g. extra whitespace: `Content-Length : 20`) causes the back-end to ignore CL and treat the request as having no body, while the front-end still thinks a body exists. This typically causes a connection deadlock unless I find an **early-response gadget (ERG)** — an endpoint that responds without consuming the full expected body (e.g. nginx serving a static file). The ERG breaks the deadlock and confirms the desync is real and exploitable, not just a timeout.

### 8. Chunk-extension abuse

Chunked encoding technically allows “chunk extensions” (metadata after the chunk size, before the newline) that almost nothing correctly validates. Malformed or oversized extensions can desync parsers that don’t strip them per spec.

### 9. Client-side desync (CSD) / pause-based desync

Unlike the above, these use fully browser-compatible requests — no malformed headers required. A slow request from a normal browser can leave a connection in a state where a front-end forwards a partial request, and back-end timeout behavior lets an attacker complete it later with an injected prefix. This matters because it means a *victim’s own browser* can be weaponized without them sending anything unusual — it defeats the assumption that desync requires “abnormal” traffic my target would immediately flag.

---

## Detection Methodology That Actually Works

### Step 1 — Use the single-packet attack, not naive timing

Sending two separate TCP writes and eyeballing response timing is unreliable — network jitter creates false positives and false negatives. The standard technique (via Burp Turbo Intruder’s request smuggling templates) sends the entire ambiguous request as a single TCP packet, so any delay I observe is attributable to the target’s parsing behavior, not the network. This is the actual basis for Kettle’s research and it’s what separates a real finding from a hopeful guess.

### Step 2 — For 0.CL/CL.0 conditions, find an early-response gadget

If my obfuscated-CL request produces a timeout instead of a clean response, I likely have a deadlock, not a dead end. Look for an endpoint the back-end will answer without reading the full body — static assets served directly by something like nginx are the classic example. Confirmation looks like: send request A (obfuscated CL, ERG target) → back-end responds early despite an incomplete body → send request B down the same connection → if B’s response shows the back-end interpreted leftover bytes from A as part of B, I’ve confirmed a real desync, not a timeout artifact.

### Step 3 — Escalate to Response Queue Poisoning (RQP) for real impact

The end goal on a confirmed desync usually isn’t a one-off cache poison — it’s convincing the back-end to send **two** responses to what the front-end thinks was **one** request. Once the front-end’s response queue is off by one, every subsequent user on that connection gets a response meant for someone else. This is the highest-severity outcome in this whole class of bug, because it’s persistent and affects every user sharing the connection, not just the attacker’s own traffic.

### Step 4 — Watch for CDN-edge desync specifically

If my captured “stolen” responses come back looking like they belong to a completely different, unrelated site, it means I may have triggered the desync inside the CDN’s shared edge infrastructure rather than my target’s own backend — meaning I can potentially route to arbitrary domains hosted on that CDN. This is high-severity and also the kind of finding that triage teams initially doubt — be ready to document it thoroughly (which domain each response belongs to, and confirmation that domain is hosted on the same CDN).

---

## Proof (for the report)

1. **Confirmed desync (no impact yet)** — proves the parsing discrepancy exists via ERG-based confirmation. Weak on its own; programs often want to see impact before paying.
2. **Single-request smuggling** — when I get one malicious request processed as if it came from the backend’s trusted context (e.g., bypassing a front-end access control check that only inspects the first request in a batch).
3. **Cache poisoning** — I poison a shared cache entry so other users receive attacker-controlled content.
4. **Response Queue Poisoning** — persistently intercept other users' responses. This is target outcome for a real submission; it’s unambiguous, high-severity, and hard for triage to dispute once demonstrated.

---

## Honest Note on “Chaining Into SAML/ORM/SSTI”

Be skeptical of framing that treats desync as an automatic stepping stone into SAML forgery or ORM injection. The realistic connection is narrower than “desync gets me into a trusted context, therefore SAML”:

- Desync can let me smuggle a request that bypasses a front-end auth check the back-end assumes was already enforced — this is real and specific, not generic “trusted context” access.
- Desync-based cache/response poisoning can expose internal error pages, stack traces, or metadata *if* the target’s back-end leaks that information on some path — this isn’t guaranteed and depends entirely on target-specific behavior I’d need to discover during recon, not something desync provides by default.
- There is no general mechanism where “desync” hands me a working path into SAML canonicalization bugs. If a chain like that exists on a specific target, it’s because of that target’s specific architecture, not because desync inherently unlocks it. Treat any study material that asserts this as a general pattern with caution — it’s the kind of overstated generalization worth stress-testing before I build a study plan around it.

---

## Updated Tool Stack

- **Burp Suite — HTTP Request Smuggler extension, v3.0+**: Now includes parser-discrepancy detection specifically built to bypass widely-deployed desync mitigations. Older versions will miss most of what’s exploitable in 2026.
- **Burp Turbo Intruder**: Required for the single-packet attack — the de facto standard for eliminating network-timing false positives during detection.
- **smuggler** (community Python tool): Useful for scripted, repeatable differential testing against a target across CL.TE/TE.CL/TE.TE variants.
- Manual raw-socket crafting (Python `socket` module) is still worth practicing by hand at least once — it forces me to actually understand byte-level framing instead of trusting a tool’s output.

---

## Detection & Testing Checklist

### Manual testing sequence

1. Fingerprint the stack: identify front-end (via headers, `Server`, CDN fingerprinting) and guess the back-end (framework error pages, timing behavior).
2. Run baseline CL.TE / TE.CL / TE.TE probes via the single-packet attack — don’t skip straight to advanced classes.
3. If those come back clean, test CL.0/0.CL — obfuscate Content-Length and look for either a clean early response (ERG present) or a hung connection (deadlock, needs an ERG).
4. Test TE.0 by manipulating chunk termination and observing whether the back-end’s dechunking reads differently than the front-end expects.
5. If the target serves HTTP/2 to clients, test H2.CL/H2.TE at the downgrade point specifically — don’t assume HTTP/1.1 findings transfer.
6. Only after a technique is confirmed via ERG or single-packet timing, attempt escalation to RQP.

### What actually indicates desync (vs. a flaky server)

- Reproducible delayed responses tied to a specific malformed request, not intermittent slowness across the whole site.
- A follow-up request on the *same connection* consistently returning content/errors that only make sense if bytes from my prior request leaked into it.
- Cache entries that change content depending on which upstream request populated them, not just cache staleness.

---

## Practice Scenarios

1. **nginx (front-end) vs. uWSGI (back-end):** nginx is strict about CL/TE; uWSGI’s re-parsing behavior can create CL.TE conditions. Good for practicing classic detection.
2. **A CDN edge vs. origin:** modern CDNs mitigate classic CL.TE/TE.CL by normalization, but obfuscated-CL (0.CL) probes can still slip through if the CDN’s own edge servers have inconsistent parsing — this is where CDN-edge desync (see Step 4 above) becomes relevant.
3. **HTTP/2-terminating load balancer vs. HTTP/1.1-only origin:** practice H2.CL/H2.TE specifically at the downgrade boundary; this is a distinct attack surface from anything HTTP/1.1-only.
4. **Static-file-serving backend as an ERG:** deliberately set up a target where a static asset path responds without consuming a full request body, and practice using it to break a 0.CL deadlock.

---

## Red Flags

1. Reproducible timeouts/delays tied to a specific malformed request (via single-packet attack, not raw send-and-wait).
2. A follow-up request on the same connection returning content inconsistent with what I sent — evidence of leaked bytes.
3. Cache entries whose content depends on which request populated them.
4. Responses that appear to belong to an unrelated domain — possible CDN-edge desync.
5. A connection that hangs on an obfuscated-CL request but responds cleanly once I add a known early-response gadget — this pattern specifically confirms 0.CL rather than a dead end.

---

## Study Resources

1. **Primary, in order:** PortSwigger Web Security Academy’s Request Smuggling section — it now includes dedicated labs for CL.0, 0.CL, response queue poisoning, and client-side/pause-based desync, not just the 2019-era labs.
2. **James Kettle’s Black Hat/DEF CON research papers**, year by year (2019 original paper, 2021 HTTP/2 paper, 2022 CL.0/CSD paper, 2024–2025 papers on TE.0, chunk extensions, and the 0.CL “desync endgame” research, 2026 HTTP Terminator talk). Read them in chronological order — each one assumes the last.
3. **RFC 7230, Section 3.3** (Message Body) — still the correct spec reference for why these ambiguities are legal in the first place.
4. **HTTP Request Smuggler extension changelog** — worth reading directly to understand exactly what the v3.0 parser-discrepancy detection checks for, since that tells I what’s now considered “already mitigated” vs. still open.

## Learning Materials (Primary Sources)

- [HTTP Desync Attacks: Request Smuggling Reborn](https://portswigger.net/research/http-desync-attacks-request-smuggling-reborn) — Kettle's original 2019 paper. Start here; everything else builds on this.
- [HTTP/2: The Sequel is Always Worse](https://portswigger.net/research/http2) — H2.CL/H2.TE research, front-end HTTP/2-to-HTTP/1.1 downgrade attacks.
- [Browser-Powered Desync Attacks](https://portswigger.net/research/browser-powered-desync-attacks) — client-side/pause-based desync (CSD); no malformed headers required, victim's own browser does the work.
- [HTTP/1 Must Die](https://portswigger.net/research/http1-must-die) — Kettle's case for why HTTP/1.1's inherent ambiguity is the root cause across this entire bug class.
- [Black Hat 2024](https://portswigger.net/black-hat-2024) — check what this specific year's talk covers before treating it as required reading.
- [RFC 7230, Section 3.3 (Message Body)](https://www.rfc-editor.org/info/rfc7230/) — the spec section that makes CL/TE ambiguity legal in the first place.
- [HTTP Request Smuggler (BApp Store)](https://portswigger.net/bappstore/aaaa60ef945341e8a450217a54a11646) — the Burp extension the whole detection methodology depends on. Install this.
- [portswigger/http-request-smuggler (GitHub)](https://github.com/portswigger/http-request-smuggler) — source code for the extension above. Save for later, once you're past foundational study — reading it tells you exactly what's checked for, and therefore what's already considered "covered."
- [HackerOne Report #1238099 — Node/ATS Request Smuggling via Chunk Extensions](https://hackerone.com/reports/1238099) — the real, primary-source disclosure behind the chunk-extension technique (the X post you found is just a pointer to this; this is the actual report). Note: this is from 2021, not 2025 — corrects the dating error in the current file.

---

