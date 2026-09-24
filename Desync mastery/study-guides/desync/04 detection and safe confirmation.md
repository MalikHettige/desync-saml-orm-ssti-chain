# 04 — Detection & Safe Confirmation

## Concept

The hardest part of desync hunting isn't finding a candidate — it's proving a real primitive exists without (a) generating a false positive that wastes a target-week, or (b) corrupting another live user's request, which your own methodology and every program's rules explicitly prohibit.

## Timing-based differential confirmation

The standard safe approach: send a request designed so that if the primitive is real, the *back end* will pause waiting for bytes that never arrive (because the front end already considered the request complete). Measure response time. A consistent, reproducible delay — not a one-off network blip — is your signal. This never touches another user's traffic because you're only ever measuring your own connection's behavior.

Repeat 3-5 times minimum before treating a delay as signal. Network jitter produces false positives; a single slow response proves nothing.

## Response-queue / differential confirmation

Send a probe request immediately followed by a canary request on the same connection. If the primitive is real, the canary's response will be malformed, delayed, or answer a question you never asked (evidence the server processed a smuggled fragment as part of it). This is more definitive than timing alone but requires careful construction so the canary can't accidentally affect anything beyond your own connection.

## What "safe" specifically means here

- Every probe request targets only your own connection/session — never craft a probe designed to land in another user's request stream during confirmation. That's the difference between *proving the primitive exists* and *exploiting it* — confirmation should stop at proof.
- Use disposable/throwaway data in any probe body — never real-looking credentials or session tokens, even your own.
- Rate-limit yourself. A burst of malformed requests against a production target is itself indistinguishable from an attack from the target's monitoring perspective, and can trip abuse detection before you've confirmed anything.
- If a program's scope or rules explicitly prohibit connection-pool interference or multi-request techniques, that closes this class off for that program — check before, not after.

## Lab task

In your labs from modules 1-3, practice building both confirmation methods against each primitive you've implemented. The goal is a confirmation you'd be comfortable running against a real production target on day one of hunting, not just a lab where you already know the answer.

## Log fields

- Confirmation method used (timing vs. differential)
- Number of repetitions and variance observed
- What would have looked like a false positive, and how you ruled it out
- Any scope/rule constraints that would block this method on a real program
