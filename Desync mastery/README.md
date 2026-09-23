# HTTP Request Smuggling / Desync — Mastery Course

Research and operational work toward elite-level HTTP request smuggling (desync) skill for bug bounty hunting.

> **Status:** Active research. Target: live hunting starting January, VDP reps until then.
> **Scope decision:** This repo previously covered a 4-stage chain (Desync → Golden SAML → ORM → SSTI). As of this revision, the primary focus is **desync alone**, mastered to elite depth, on the reasoning that it has the broadest applicability across modern architectures (anything behind a CDN/reverse proxy) with the lowest competition among hunters. SAML/ORM/SSTI remain valuable opportunistic skills but are no longer the primary track — see `resources/scope-decision.md`.

## Why desync, specifically

Most hunters still default to IDOR and classic auth bypass. Very few have operationalized:

- The full primitive taxonomy (not just the 2019 classics)
- Client-side desync (2022 research) — works without a reverse-proxy pair
- The HTTP/2-to-HTTP/1.1 downgrade seam (2024–2025 frontier) — CDN/WAF/gateway edges
- Safe, non-destructive confirmation methodology
- A rehearsed primitive-to-impact playbook, not just detection

This repo is the structured, working implementation of that gap.

## Course map

Read in this order — each module builds on the last:

1. [`study-guides/desync/00-course-map.md`](study-guides/desync/00-course-map.md) — how to use this course
2. [`study-guides/desync/01-fundamentals-and-taxonomy.md`](study-guides/desync/01-fundamentals-and-taxonomy.md) — why desync exists, full primitive taxonomy
3. [`study-guides/desync/02-client-side-desync.md`](study-guides/desync/02-client-side-desync.md) — CSD, browser-as-frontend
4. [`study-guides/desync/03-http2-downgrade-seam.md`](study-guides/desync/03-http2-downgrade-seam.md) — the current frontier
5. [`study-guides/desync/04-detection-and-safe-confirmation.md`](study-guides/desync/04-detection-and-safe-confirmation.md) — proving a primitive without collateral damage
6. [`study-guides/desync/05-primitive-to-impact-playbooks.md`](study-guides/desync/05-primitive-to-impact-playbooks.md) — turning a confirmed primitive into a reportable bug
7. [`study-guides/desync/06-tooling.md`](study-guides/desync/06-tooling.md) — Burp extensions, Turbo Intruder, raw sockets
8. [`recon/target-fingerprinting-and-primitive-matrix.md`](recon/target-fingerprinting-and-primitive-matrix.md) — pre-qualifying a target before spending a target-week on it
9. [`resources/primary-research-reading-list.md`](resources/primary-research-reading-list.md) — the source papers, in order
10. [`progress/mastery-checklist.md`](progress/mastery-checklist.md) — how you'll know you've actually mastered this, not just read about it

## Repository structure

```
desync-course/
│
├── study-guides/desync/   # The 8-module course above
├── labs/desync/           # Your intentionally vulnerable practice labs (build per module 1-4)
├── recon/                 # Target fingerprinting & primitive-matrix methodology
├── progress/              # Weekly notes, lab validation logs, mastery checklist
└── resources/             # Primary research reading list + scope-decision note
```

## Important notes

- All labs are intentionally vulnerable, for educational/authorized use only.
- Never use any material here against systems you do not own or have explicit permission to test.
- Confirmed findings on real programs get written up separately, after responsible disclosure, in `Bug-bounty-reports`.

## Related repositories

- Conceptual notes & paper summaries → [`Research-notes`](https://github.com/MalikHettige/Research-notes)
- Reusable Python tooling → [`python-programming`](https://github.com/MalikHettige/python-programming)
- Educational lab writeups → [`Bug-bounty-writeups`](https://github.com/MalikHettige/Bug-bounty-writeups)
- Live findings → [`Bug-bounty-reports`](https://github.com/MalikHettige/Bug-bounty-reports)

## Disclaimer

This work is for authorized security research and educational purposes only. The author is not responsible for any misuse of the information or code contained in this repository.
