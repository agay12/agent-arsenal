# Agent Arsenal — bug bounty & BSC due diligence toolkit
Curated from 9 repos studied 19 Nov 2026. Skills live in the Hermes agent profile; this repo mirrors the installed SKILL.md files and environment notes for reference/reinstall.

## Sources
- https://github.com/setyamickala/fastmint-fcfs-agentic (FCFS mint BSC via Binance Agentic Wallet)
- https://github.com/setyamickala/bsc-contract-audit-skill (BSC token/NFT/lock + launch-curve audits)
- https://github.com/cloudflare/security-audit-skill (6-phase adversarial audit)
- https://github.com/awarexone/Agentic-Bug-Hunter (bughunter CLI)
- https://github.com/0xKayala/NucleiFuzzer (nf bulk scanner)

## Pipeline (BSC)
launch-curve-audit → bsc-contract-audit → only on `DECISION: GO` → fastmint-fcfs (never bypass its safety rails).

## Hunt pipeline
/scope (in-scope check) → /recon → /hunt → /validate (7-question gate) → /report

Details: notes/repo-references.md · notes/environment-2026-11-19.md
