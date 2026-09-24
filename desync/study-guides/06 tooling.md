# 06 — Tooling

## Concept

Burp's Repeater normalizes requests by default — it will silently "fix" malformed length headers, duplicate headers, and odd whitespace before sending, which is exactly the malformation you need for desync work. Using default Repeater for this class of bug is the single most common way hunters conclude a target "isn't vulnerable" when it actually is. Tooling fluency here is not optional polish, it's a prerequisite.

## Required tools

**Burp Suite + HTTP Request Smuggler extension** — Purpose-built for this class: disables normalization for the requests it sends, includes automated scanning for several classic primitives, and lets you hand-craft raw byte sequences. Start here for modules 1 and partially module 3.

**Turbo Intruder** — Essential for the timing-based confirmation in module 4 and for the single-packet-attack style precision needed for race-condition-adjacent desync confirmation (sending near-simultaneous requests with controlled timing that ordinary Repeater/Intruder can't achieve reliably). Learn its Python-based request-scripting model, not just the UI.

**Raw socket scripting (Python `socket` module, or `netcat`/`openssl s_client` for manual work)** — For anything Burp's extension doesn't cleanly support, especially HTTP/2 downgrade-seam work (module 3) and CSD lab construction (module 2), where you often need precise control over frame-level behavior that HTTP-library abstractions hide from you.

**Browser DevTools Protocol tab + `curl --http2`** — For fingerprinting whether a target speaks HTTP/2 at the edge (module 3's first step) before you invest lab time assuming a downgrade seam exists.

## Lab task

Rebuild at least one primitive from module 1 three different ways: via Burp + HTTP Request Smuggler, via a raw Python socket script, and via manual `openssl s_client`/`nc`. The goal isn't redundancy — it's understanding exactly what each tool does and doesn't normalize, so you know which tool to reach for when a target's specific quirk doesn't fit the default extension's assumptions.

## Log fields

- Tool used per attempt
- What (if anything) the tool normalized that you had to work around
- Which tool you'd default to for this primitive on a live target, and why
