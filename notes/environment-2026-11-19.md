# Bug Bounty Environment Checklist (verified 19 Nov 2026)
All installed & on PATH (persisted ~/.bashrc: /usr/local/go/bin, /root/go/bin, /root/.foundry/bin, /root/.local/bin):

| Category | Tools |
|---|---|
| AI hunting CLI | bughunter, bughunter-agent (Agentic-Bug-Hunter via uv), Ollama + llama3.2:3b |
| Recon | subfinder, httpx, katana, waybackurls, gau, dnsx, nmap, hakrawler |
| Vuln scanning | nuclei (+101MB templates ~/nuclei-templates), ffuf, dalfox, uro, arjun |
| Param discovery | paramspider (wrapper /usr/local/bin/paramspider -> /root/ParamSpider) |
| Bulk scan | nf = NucleiFuzzer v4 (wrapper -> /root/skills-repos/NucleiFuzzer) |
| Solidity PoC | forge, cast, anvil (Foundry) |
| Runtime | go 1.22.5, node, python3, jq |

Repos cloned: /root/skills-repos/ (Agentic-Bug-Hunter, NucleiFuzzer, bsc-contract-audit-skill, fastmint-fcfs-agentic, security-audit-skill)
Tip: projectdiscovery release zips for old versions return 404 — use `go install -v repo@tag` instead.
