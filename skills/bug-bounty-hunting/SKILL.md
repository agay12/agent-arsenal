---
name: bug-bounty-hunting
description: Use when hunting bugs or doing bounty recon/report.
version: 1.0.0
author: controller
license: MIT
metadata:
  hermes:
    tags: [bug-bounty, security, recon, vulnerability]
    related_skills: [systematic-debugging, requesting-code-review]
---

# Bug Bounty Hunting — Agentic-Bug-Hunter toolkit

## When to Use
Load this when the user asks to hunt/find bugs, run recon on a domain, validate a vulnerability finding, write a bug bounty report, or work with the `bughunter` CLI / Agentic-Bug-Hunter repo.
User runs bug bounty hunting with **Agentic-Bug-Hunter** (https://github.com/awarexone/Agentic-Bug-Hunter, MIT, standalone CLI `bughunter` + Claude Code plugin + MCP server). Repo studied 2026-11-17.

## Pipeline
`/recon target.com` (subfinder, httpx, katana, nuclei sweep) → `/hunt target.com` (IDOR, auth bypass, SSRF, XSS, SQLi, logic flaws) → `/validate` (7-Question Gate, kills weak findings) → `/report` (H1/Bugcrowd/Intigriti/Immunefi format). `/autopilot target.com` = full autonomous loop.

## Key commands
- recon: `/surface`, `/scope-aggregate <program>`, `/cloud-recon --keyword`, `/param-discover`, `/secrets-hunt --js-bundle`, `/takeover --recon`, `/scan-cves`, `/bypass-403`, `/portscan`, `/screenshot -l urls.txt`
- scanners: `/cors`, `/crlf`, `/nosqli`, `/jwt-scan`, `/oob` (interactsh blind SSRF/XXE), `/sast` (semgrep), `/domxss` (headless Chromium), `/llm-redteam`
- web3: `/web3-audit <contract.sol>`, `/token-scan <contract>` (rug pull)
- session: `/pickup` (resume), `/intel`, `/chain` (chain bug A with B/C), `/scope` (check in-scope first — ALWAYS before testing), `/triage`, `/remember`, `/memory-gc`, `/arsenal`

## Architecture
- 9 agents: recon-agent, report-writer, validator, web3-auditor, chain-builder, autopilot, recon-ranker, token-auditor, credential-hunter (hard-stop before password spray)
- 26 web2 vuln classes + 10 web3 classes; skills/ dir has knowledge bases (bug-bounty, bb-methodology, web2-vuln-classes, security-arsenal payloads, triage-validation, report-writing, web3-audit)
- CLI: `bughunter setup` (providers: Ollama free/offline → Groq free → DeepSeek → Claude/OpenAI, auto-detect order), `hunt/recon/validate/report/chat`, install via `./install.sh --agent standalone`, tools via `install_tools.sh` (subfinder, httpx, nuclei, katana, ffuf, gau, dnsx, nmap, dalfox)
- MCP mode: `./install.sh --agent mcp` then `bughunter mcp serve` — lets this agent drive it
- Output dir `~/.bughunter/`; hunt memory persists across sessions (JSONL, 10MB cap); auth headers set once carry through all tools
- demo local target: `python3 serve.py` in repo

## Choosing between programs
When the user hands over several candidate bounty programs, triage them with `references/program-selection.md` (submission-gate decision table + selection order) before committing to one — the gate determines whether the hunt can ever be submitted.

## Web3 / contract-repo program workflow
1. Pull the program's full docs set first: bounty page, scope/contracts page, admin page, AND the published security review / known-issues page. The security review is the baseline for dedupe — every L-x/I-x/K-x finding listed there is already known and unpayable; scope your hunt to what it does NOT cover.
2. Trust the program's security-review page over the repo's SECURITY.md when they disagree about bounty status — docs pages are kept current per report; repo markdown can go stale mid-program.
3. Audit the DEPLOYED commit, not main: find the deployed commit hash (usually cited in the security review or deployment records) and `git fetch --unshallow` + `git diff --stat <deployed> HEAD -- contracts/` to see what is actually live. A repo's latest code may not be deployed yet. When no deployed commit is cited, find the last-AUDITED baseline instead: scan `git log --oneline` for a commit named after the audit (pattern `audit <date>`), then `git diff --stat <baseline> HEAD -- contracts/` — everything in that diff is the unaudited surface and the highest-yield target.
4. Highest-value targets are the contracts most recently added or being rolled out (deployed-or-pending, least-reviewed by the team's own reviewers) plus anything the published review explicitly left for a later external audit.
5. Parallelize the audit by contract area (router, pools/fees, market/registry, cover/limit) with one subagent per area, each told the out-of-scope list verbatim (admin-key compromise, third-party tokens, user slippage, no-loss DoS) AND explicitly that the attacker must be a NON-privileged external party — most programs pay nothing for findings that need an owner/admin/solver/relayer to act maliciously — so findings come back triaged.
6. Grep the repo's docs and code comments for fix/report IDs (pattern `K-[0-9]`, `L-[0-9]`, `M-HRL`) — commit messages and inline comments often reference the team's internal finding numbering; add those IDs to the dedupe baseline even when the security review page omits them. Also grep NatSpec/`@dev` comments for documented-intent disclaimers (e.g. 'leftover balances can be claimed by any caller') — behavior documented in a code comment is intended and unpayable; collect these BEFORE dispatching subagents so they can kill those leads early.
7. Verify every subagent finding yourself before it enters a report: re-read the cited file:line range and re-derive the exploit. Subagent reports are self-assessments; several will be Low-but-arguable or already-known — your own read decides which ones survive the payout table. Match severity to the program's own severity definitions, not generic CVSS (e.g. "a contract failing to deliver what it promises without loss of funds" is usually an explicit Medium band in web3 bounty tables).
8. Check the SUBMISSION MECHANICS at hunt start, not at the end: many protocols accept private reports only via GitHub security advisories (DMs explicitly refused). GitHub's REST/GraphQL API cannot create an advisory on a repo you lack access to (REST returns 403; the GraphQL mutation does not exist for third-party repos) — only the web UI works, which needs a logged-in browser session. If no saved login exists (`browser_vault action=list`), surface the credential need to the user BEFORE the audit finishes so submission isn't blocked by a login wall afterward.
   **Login is a hard budget: at most ONE alternative attempt.** If the first browser-login route fails (no saved credential, preview-pane bridge dead, cookie not shared across browsers), stop probing variants — device flow/API tokens cannot mint a web session, and each extra retry reads as stalling. Immediately switch to the manual path (step 10) and hand the user a finished, copy-paste-ready submission. This user reacts to repeated dead-end retries with open frustration.
9. On testnet-gated programs (closed-source node/API targets), bootstrap the funded test account FIRST — most faucets check mainnet existence server-side. Discover the faucet's real mechanics by reading the SPA's lazy-loaded page chunk (fetch the main bundle, extract the `assets/<Page>-*.js` name from its chunk map, fetch that chunk, grep the fetch/axios call) — claim flows often reduce to one unsigned info endpoint. Probe eligibility by calling it with a fresh wallet and reading the rejection message; verify the user's own addresses via the chain's public info endpoints (perp state + spot state) before assuming any wallet qualifies. Never assume — a wallet that has touched the token on an EVM side may not exist as a core account at all.
10. When submission needs a user login the agent cannot reach, write the findings to a self-contained report file the user can submit manually — do not leave findings only in chat. One advisory per finding, highest severity first. For durable future access, offer `browser_vault action=save_login` (user types credentials into the masked prompt, never in chat); if the save comes back declined, do NOT retry in the same turn — ask the user to send a fresh message and re-trigger there.
    GitHub's advisory web form splits the report across SEPARATE fields (Title, Details, Proof of concept, Impact, Affected products with Ecosystem/Package/versions, Severity + CVSS vector string, CWE search) — handing the user one mega Description block makes them guess where each piece goes and they end up pasting into the wrong page entirely. Pre-split the paste blocks to match the form fields one-to-one, and drop markdown fencing from the PoC block (plain text) so nested backticks cannot break the paste. Always state the CVSS vector explicitly and the CWE number to search for.
   For email-submission programs (bugs@..., bounty@...), the user prefers sending the report email HIMSELF from his personal address — deliver the finished email (subject + body, one per finding, per the program's recommended layout) as a file in the workspace and notify him; do not send it yourself.
    When the description itself contains code fences (PoC), open the OUTER block with FOUR backticks (````markdown) — nesting a three-backtick fence inside a three-backtick fence inserts an invisible zero-width character into the outer fence and the pasted report carries a stray glyph the user must hunt down and delete.
11. Validate the top finding with a runnable Foundry PoC before submission (one unit test against the deployed source, no fork needed for admin/state-transition bugs). A passing PoC turns a 'sketch' into a confirmed report and often sharpens the claim — a static sketch undersold one such finding before the test showed the whole bond escaped instead of just the pending share.
12. In PoCs for delayed-admin contracts (timelock, unbond delays), model schedule and execution as SEPARATE time points — execute the admin call at `queueTime + delay`, never at queue time. Calling it early inflates what the admin can reach and produces a false PoC failure that looks like a non-bug.

## Closed-source SaaS / consumer product programs (agent products, black-box web apps)
1. Derive the attack plan from the program's payout-guidelines page FIRST: list its categories and mark which are testable from a SELF-OWNED account (sandbox escape, credential extraction, egress bypass) versus which need a second victim account — self-only categories are the realistic starting tier when you have one account. Read the vendor's own security-architecture blog next: it names the boundary components (approval engine, credential store, isolation layer) that the guidelines pay for crossing.
2. When testing needs a platform account, attempt automated signup with a disposable email at most ONCE. Large consumer platforms (Meta/Google-scale) detect datacenter IPs plus disposable domains and hard-block creation; looping retries only burns the session. On block, immediately surface the credential need to the user (browser_vault save_login with the masked prompt) — an account the user owns is valid for testing under standard program terms.
3. Disposable test email: mail.tm API works (rate-limited — retry with backoff on 429; create account, POST /token, then poll /messages for the OTP code). Verification codes usually arrive within seconds and resending often returns the SAME code, so re-try the last fetched code before requesting a new one.
4. Automating React-controlled signup forms: plain property assignment does not trigger the framework's change detection — set values via the native prototype setter then dispatch `input`/`change` events (or focus + document.execCommand("insertText") for text fields), or the Continue/Confirm button silently does nothing.
5. After EVERY action in a multi-step auth flow, re-read the page state (inputs/buttons/visible text) before the next step — login flows mutate state between calls, and a button that matched on the last call may be gone. When the browser tab shows about:blank mid-flow, call ensure_real_tab() and re-navigate before concluding an element is missing.
6. Before attempting any signup/login automation, write a HUNT-PLAN.md in the workspace with: program URL, payout categories, what is blocked and why, and the exact credential ask. If the account is blocked, the plan file lets the next session resume without redoing the research.

## Toolkit upgrade (19 Nov 2026) — repos cloned at /root/skills-repos/
- **Agentic-Bug-Hunter** cloned at /root/skills-repos/Agentic-Bug-Hunter (was fetched ad-hoc before). Also **NucleiFuzzer** at /root/skills-repos/NucleiFuzzer (`nf -d domain` = nuclei fuzzing-templates + paramspider/waybackurls/katana/gauplus/hakrawler, dedupe uro + httpx; XSS/SQLi/SSRF/open-redirect) — good bulk-scan complement for wide-scope programs; needs nuclei+katana+httpx+uro installed (`install.sh`).
- **cloudflare/security-audit-skill** at /root/skills-repos/security-audit-skill — use its 6-phase adversarial-validation workflow (see the installed `security-audit` skill) for auditing OUR OWN repos/agents before publishing, and as a coverage-ledger pattern for big hunts; web3 programs still use the manual contract-area workflow above.
- **BSC due-diligence pipeline**: before any BSC buy/mint, run the installed `launch-curve-audit` (sale mechanism) then `bsc-contract-audit` (asset/escrow); only proceed on `DECISION: GO`. For FCFS mints via Binance Agentic Wallet use the installed `fastmint-fcfs` skill (never bypass its safety rails).
- Repositories studied & saved: NVIDIA/auto-ontology (ontologi semantik MCP), vercel-labs/agent-desktop (computer-use via accessibility tree), meta/astryx (design system React+StyleX), anthropics/skills (Agent Skills spec) — details in /root/notes/repo-references.md.

## Environment checklist (VERIFIED 19 Nov 2026 — ALL COMPLETE)
Installed & on PATH (persisted ~/.bashrc): bughunter (ACTIVE PROVIDER: Ollama @ localhost:11434, model llama3.2:3b — verified via API; `ollama serve` is a systemd service, auto-starts), subfinder, httpx, nuclei (templates 101MB ~/nuclei-templates), katana, waybackurls, gau, hakrawler, ffuf, nmap, dnsx, dalfox, uro, arjun, paramspider (wrapper /usr/local/bin/paramspider -> /root/ParamSpider), nf = NucleiFuzzer v4 (wrapper -> /root/skills-repos/NucleiFuzzer), forge/cast/anvil (Foundry), go1.22.5, node, python3, jq. Ollama needs zstd (installed) for its installer; no GPU — CPU-only inference, 3B model fits the 5.8GB workenv RAM cap.
Public mirror: https://github.com/agay12/agent-arsenal (installed SKILL.md files + notes/repo-references.md + notes/environment-2026-11-19.md; local clone /root/agent-arsenal — update there and push with GH_TOKEN from /root/.makefun/github-token.json).
Tip: projectdiscovery release zips for old versions return 404 — use `go install -v repo@tag` instead. bughunter `chat` is interactive-only; for one-shot AI use the engine's hunt/recon commands or Ollama HTTP API directly.
Remaining optional (not installed): gauplus (gau installed instead), amass, Jason, interlace. Optional paid upgrade: GROQ_API_KEY/DEEPSEEK_API_KEY would make bughunter's AI much faster than local 3B — ask user if hunts feel slow.

## Rules when hunting
1. Always verify target is IN SCOPE (`/scope` or program policy) before any testing.
2. Run validate gate before writing any report; kill weak/duplicate findings.
3. Reports: impact-first, submission-ready per platform format.
4. Authorized testing only — no out-of-scope, no credential spraying without explicit stop-check.
5. Missing external tools are skipped not errored; check `/arsenal` first and install what's needed.

## Environment notes
See `references/solidity-audit-pitfalls.md` for Solidity semantics that mislead line-by-line audits (revert-in-try, silent narrowing casts, balance-based "backing" checks, eviction/DoS compounds).
- Foundry in the work environment: `curl -sL https://foundry.paradigm.xyz | bash` then `export PATH="$PATH:/root/.foundry/bin" && foundryup` (plain `pip` installs do not provide forge).
- Bounty-repo JS dependencies: `yarn install --frozen-lockfile` fails when a dep pins a newer node engine — retry with `--ignore-engines`; on a corrupted yarn cache error (ENOENT .yarn-tarball.tgz) run `yarn cache clean` first. A large contracts repo's `forge build` takes many minutes and saturates CPU when several run concurrently — start it as a background terminal process and build once, shared, before subagents run forge test.
- GitHub in the work environment: a token without the `read:org` scope fails `gh auth login` scope validation — export `GH_TOKEN=<token>` instead (gh uses it for all API calls without logging in). For git push, `git config --global credential.helper store` plus `https://<user>:<token>@github.com` in `~/.git-credentials` (chmod 600); persist the GH_TOKEN export in `~/.bashrc`. Verify with `gh api user --jq .login`.
- Running this on Hermes: install in work environment via terminal (clone repo, install_tools.sh, `pip install agentic-bug-hunter` or `install.sh --agent standalone`). Resource budget applies (check /etc/hermes-workenv-resources.json if OOM); nmap etc. not guaranteed preinstalled.
- Next step when user orders a hunt: confirm target + program scope, check arsenal, then recon → hunt → validate → report.