# Primary Research Reading List

Read the source research itself, in this order — not summaries. This alone is a differentiator, since most hunters only know this material secondhand.

1. **"HTTP Desync Attacks: Request Smuggling Reborn"** (James Kettle / PortSwigger, 2019) — the foundational paper. Establishes CL.TE/TE.CL/TE.TE and the original methodology. Read this first regardless of how much secondary material you've absorbed elsewhere — it's where the timing-based confirmation technique in module 4 originates.

2. **PortSwigger Web Security Academy — HTTP Request Smuggling labs** — pair with paper #1. Work every lab, including the ones that feel repetitive; the repetition is what builds the pattern recognition for spotting real-world variants that don't look exactly like the lab.

3. **"Browser-Powered Desync Attacks"** (James Kettle / PortSwigger, 2022) — the CSD paper covered in module 2. This is the point where the field moved past "needs a reverse-proxy pair."

4. **PortSwigger's more recent HTTP/1.1-parsing-inconsistency research (2024–2025)** and the associated CDN-compromise disclosures — this is module 3's source material, covering the HTTP/2-to-HTTP/1.1 downgrade seam. Search PortSwigger's research blog directly for the current, exact titles and dates rather than relying on this list — this area is actively being published on, and exact naming may have moved since this file was written.

5. **HTTP Request Smuggler (Burp extension) documentation** — read the docs, not just the tool. It explains *why* it constructs requests the way it does, which teaches the underlying parsing logic better than using it blind.

6. **Any public bug bounty disclosure reports tagged "request smuggling" or "desync"** on HackerOne Hacktivity / disclosed reports on your 5 target platforms — real-world variance from the textbook cases, and a source for module 5's playbook practice (see also your own daily-training-schedule's disclosed-report-reading block — prioritize desync-tagged reports specifically during that time).

## Note on staying current

This is one of the more actively-researched corners of web security right now. Re-check PortSwigger's research output roughly monthly during your prep phase — new primitive variants and new edge-product-specific findings are still being published as of this writing, which is exactly why fluency here remains a competitive edge rather than settled, commoditized knowledge.
