# 05 — Primitive → Impact Playbooks

## Concept

A confirmed desync primitive is a door, not a bug report. Triagers pay for demonstrated impact, not "I can desynchronize requests." Rehearse each of these paths explicitly in your labs so you're not improvising the escalation live, on the clock, during a real target-week.

## Playbook A: Session hijacking

Once you can reliably smuggle a fragment into the next user's request stream, craft the smuggled fragment to capture whatever the *next* connection sends — session cookies, auth headers, CSRF tokens. Practice: in your lab, have a second "victim" client send a request right after your smuggle and confirm you can recover something from its request in your own response.

## Playbook B: Cache poisoning via smuggling

Combine with your cache-poisoning work (curriculum item 7). A smuggled request can be crafted so its *response* gets cached under a URL that legitimate users will hit — meaning you only need to pull this off once, and every subsequent visitor to that cached URL is affected. This is the highest-severity playbook here because of blast radius: it converts a single successful smuggle into a persistent, self-serving compromise.

## Playbook C: Front-end security control bypass

Many architectures put auth checks, WAF rules, or rate limiting at the front end/edge, trusting the backend to only ever see pre-validated traffic. A smuggled request can reach backend routes that assume they're unreachable without passing the edge's checks first — internal admin routes, debug endpoints, routes the backend developer never expected to be directly exposed. Practice identifying "front-end-only" security controls in your lab setup and confirming a smuggled request bypasses them.

## Playbook D: Request queue poisoning for account takeover

In some architectures, smuggling can cause your crafted request to be answered *to a different, unrelated victim connection* rather than the other way around — meaning a victim receives a response meant for you (e.g., an authenticated response, a password-reset token, an OAuth redirect with a code). Practice constructing this direction of the primitive deliberately, not just the "I capture their data" direction — both are valid depending on target architecture.

## Lab task

For each playbook, using the primitives you built in modules 1-3, produce one complete end-to-end demonstration in `labs/desync/` — primitive confirmed → playbook executed → concrete, articulable impact. This is the artifact you'll be able to point to and say "I've done this before" the first time you find a real primitive live.

## Log fields (per playbook)

- Primitive used as the entry point
- Exact escalation steps
- Concrete impact achieved (not hypothetical — what you actually captured/bypassed/poisoned in the lab)
- How this playbook's impact would be described in a report for a triager unfamiliar with desync
- Severity justification tied to the program's own rubric language
