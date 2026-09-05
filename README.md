# Desync → Golden SAML → ORM → Blind SSTI Chain

Research and operational work on the multi-stage attack chain:

**HTTP Request Desynchronization → Golden SAML (XML Canonicalization) → ORM Type Confusion → Blind SSTI → RCE**

This repository contains:

- Study guides for each stage
- Intentionally vulnerable lab environments
- Full-chain integration notes
- Reconnaissance & targeting methodology tailored to this chain

> **Status:** Active research (4-month mastery plan – target first real-world report by mid-January 2027)

---

## Why This Chain?

Most hunters still focus on IDOR / classic auth bypasses.  
Very few have operationalized the combination of modern desync primitives (0.CL, TE.0, chunk-extension, CSD), Golden SAML signature reuse via canonicalization collisions, ORM type confusion, and blind SSTI side-channel exploitation.

This repo is the practical implementation of that research.

---

## Repository Structure

```text
desync-saml-orm-ssti-chain/
│
├── study-guides/        # Deep conceptual guides for each stage
│
├── labs/                # Standalone vulnerable labs + exploitation
│   ├── desync/
│   ├── saml/
│   ├── orm/
│   └── ssti/
│
├── integration/         # Full-chain design, diagrams, and end-to-end notes
├── recon/               # Targeting methodology specific to this chain
├── progress/            # Weekly notes & lab validation (sanitized)
└── resources/           # Curated links to primary research papers
```

---

## Current Focus (Phase 1–2)

- [x] Master Checklist created
- [ ] Foundation study (Desync, SAML, ORM, SSTI)
- [ ] Individual PoC labs built and verified
- [ ] Full-chain integration
- [ ] Live hunting

---

## Important Notes

- All labs are **intentionally vulnerable** for educational purposes only.
- Never use any material from this repository against systems you do not own or have explicit permission to test.
- Real findings against bug bounty programs will be published separately in the `Bug-bounty-reports` repository after responsible disclosure.

---

## Related Repositories

- Conceptual notes & paper summaries → [`Research-notes`](https://github.com/MalikHettige/Research-notes)
- Reusable Python tooling → [`python-programming`](https://github.com/MalikHettige/python-programming)
- Educational lab writeups → [`Bug-bounty-writeups`](https://github.com/MalikHettige/Bug-bounty-writeups)
- Live findings → [`Bug-bounty-reports`](https://github.com/MalikHettige/Bug-bounty-reports)

---

## Disclaimer

This work is for authorized security research and educational purposes only.

The author is not responsible for any misuse of the information or code contained in this repository.

## Disclaimer

This work is for authorized security research and educational purposes only.  
The author is not responsible for any misuse of the information or code contained in this repository.
