# [PARKED] CRLF Desync → Golden SAML → ORM-Leak → Blind SSTI-RCE

Status: shelved, not started. Read this whole file before resuming — don't just skip to a task list.

## The idea
A four-stage chain: CRLF-powered HTTP request smuggling → Golden SAML forgery → ORM-type-confusion data exfiltration → blind SSTI escalating to RCE, with server-side prototype pollution held as an RCE fallback.

## Why it was shelved (read this part first)
- **These four steps don't functionally chain.** A real chain means each step produces the access the next step needs. None of these do — a desync bug gives you nothing that helps forge a SAML assertion; forging a SAML assertion gives you nothing that helps an ORM confusion bug. Four separate skill domains, not a pipeline. If you're picking this back up, treat it as four independent things to learn, not one curriculum.
- **Golden SAML doesn't fit bug bounty.** It requires already holding the identity provider's signing-key private key — it's a post-exploitation technique used in internal AD red-team engagements after access is already won, not something external web testing finds. The bug-bounty-relevant near-cousin is SAML signature bypass / XML Signature Wrapping, which forges assertions without ever touching the key. If SSO is genuinely the interest, that's the real target.
- **Prototype pollution as "RCE fallback" only works on Node.js targets.** It's not a universal fallback across stacks — worth knowing that going in.
- **ORM-Leak (blind ORM injection) is real but narrow**, dependent on the specific ORM/framework a target runs.
- **Two of the four are already legitimate and already available**: request smuggling and SSTI are both existing PortSwigger Academy modules. No separate curriculum needed for those two specifically, whenever they're picked up.

## What the fuller analysis actually pointed to
- **Highest expected value given real progress already on hand:** IDOR/Broken Access Control, Authentication, Business Logic. No target-architecture prerequisite — every multi-user app has this surface. Logic-based, so scanners can't find it, which keeps the field smaller. OWASP's #1 category, trending up, not down.
- **Highest theoretical ceiling, if payout size alone is the metric:** smart contract / Web3 security auditing (Solidity, EVM internals). A completely separate discipline from everything above, starting from zero.

## Before reopening this file
This was the fourth different "final answer" landed on in a single sitting — this chain, then a revised web/cloud path built on real existing progress, then smart contract security, then back to this. There are also already a few other bug-bounty project spaces sitting around. None of that means this is the wrong choice — it means: before starting, check this is still the actual answer once some time has passed, not just the most recent thing that sounded impressive.
