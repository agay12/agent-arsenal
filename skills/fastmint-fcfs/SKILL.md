---
name: fastmint-fcfs
description: Use for FCFS mint/claim on BSC via Binance Agentic Wallet.
metadata:
  author: setyamickala (installed from github.com/setyamickala/fastmint-fcfs-agentic)
  source: /root/skills-repos (workenv)/fastmint-fcfs-agentic
---

# Fast FCFS mint (Binance Agentic Wallet, BSC 56) — target: broadcast < T+5s

Trigger words: "mint free jam HH:MM", "fcfs", "gas", "gausah riset", "mint cepat", free/paid FCFS mint, claim, or share sale fired the moment it opens from the Binance Agentic Wallet (baw) on BSC, broadcast inside 5 seconds of the open.

`SKILL_DIR` = /root/skills-repos (workenv)/fastmint-fcfs-agentic (cloned; original files: fastmint.sh, fire.js, lib.js, recon.js, bench.js, bundled ethers 6.13, .env with QuikNode BSC endpoint).
Everything runs through `./fastmint.sh` from SKILL_DIR: `bench`, `recon`, `fire`, `logs`, `locks`.
Speed matters, so skip deep risk research unless the user asks. The safety rails below are never skipped.

## Safety rails (never skip)
- Fire only on the user's explicit command for this mint ("mint free jam 23:00", "gas", "ok"). A link alone is not a go-ahead.
- Free mint = `--value 0`. A paid mint needs an amount the user stated. Pass it as `--value` and `--maxValue` (cap). fire.js refuses value > maxValue and refuses value > 0 without a cap.
- One broadcast only. fire.js writes `runs/lock_<hash>.json` before execute and refuses the same call again. Never auto-retry after execute, even on an execute error or a reverted tx. Delete a lock only on the user's explicit new command.
- Never run `baw auth signin` yourself. If status is not CONNECTED, stop and ask the user. Never ask for a private key, seed phrase, or website session token.
- Never work around Binance risk blocks (351803), RISK flags, or allowance/authority changes in the preview. Stop and report.
- Use only the connected wallet. `bench --live` and any real execute cost gas: only on the user's explicit OK.

## Timing (measured 2026-11-11)
block with ts >= start seen: 0.4-0.9s (WS newHeads + HTTP poll on 3 fastest RPCs) | preview 0.55-0.75s (requestId lives 300s; fails 351805 until open) | execute 0.7s | inclusion 0.4-0.5s. Expected: broadcast T+1.3-2.0s, included T+1.8-2.5s (old sequential flow was T+5-6s).

fire.js runs 3 triggers in parallel (spaced --gap 350ms, max --inflight 3): (1) clock at NTP-corrected start, (2) head = block ts >= start, (3) eth_call OK on the 4 fastest RPCs every 100ms. Clock+head only during --burst (3s); after that only eth_call OK launches previews. First preview with simulationCode 000000000 is executed once.

## Flow
0. **Env check (~10s):** `./fastmint.sh bench` — ranks RPCs, WS delay, clock offset, baw status/devMode/session, Agent ID, preview latency. Fix reds before promising a time: UNCONNECTED -> user signs in; 351802/351804 -> user enables Developer Mode in Binance App; ISP DNS hijack (SSL_ERROR internetbaik.telkomsel.com / internetpositif.id) -> user needs encrypted DNS (Cloudflare 1.1.1.1) or VPN — never change system DNS yourself. TLS UNABLE_TO_GET_ISSUER -> `bench --ca`.
1. **Recon (~5-10s):** `./fastmint.sh recon <site-url>` (add `--contract 0x..`). Fetches HTML/JS bundles/agent docs (llms.txt, skill.md...), extracts addresses, chainId, mint fns, ABI sigs, unix times, eth_getCode, Sourcify ABI, view getters, prints MINT CANDIDATES + ready `fire:` command with `--start` from START HINT. Arg guesses: inviter 0x0, agentId `$agentId` (registry 0x60B9262456dE011F7d15a54CdD8c48bAaa8B022F, tokenId = uint256(keccak256(handle))), qty 1, deadline `$now+1800`. Dynamic args re-read right before every preview: `{"getter":"totalMinted()","add":1}`, `{"getter":"saleStart()"}`. SoldOut/FreeGone/AlreadyTaken/AgentIdRequired -> stop and tell user.
2. **Rehearse (always):** same command + `--dry --start 10s-ahead`; stops before execute. Use `--gateSim` when call already mintable. Check the `call = ...` line decodes to intended function/args.
3. **Fire (background):**
   ```bash
   cd /root/skills-repos (workenv)/fastmint-fcfs-agentic
   ./fastmint.sh fire --to 0xSALE --sig "claimFreeShare()" --start 23:00
   ./fastmint.sh fire --to 0xSALE --sig "mintFree(uint256,uint256,uint256,uint256,address)" --args '["$agentId","1","0","$now+1800","0x0000000000000000000000000000000000000000"]' --start 21:00
   ./fastmint.sh fire --to 0xSALE --sig "mint(uint256)" --args '[{"getter":"totalMinted()","add":1}]' --start 1794330000
   ./fastmint.sh fire ... --value 0.01 --maxValue 0.01 --start 23:00   # paid
   ./fastmint.sh fire ... --dry                                       # rehearsal
   ```
   Prechecks parallel (CONNECTED, devMode, code, balance, no lock, ABI); re-check session T-20s, re-rank RPCs T-10s; at T-3s opens WS heads + eth_call spin. NotStarted/351805 retried until --timeout (600s); terminal reverts (sold out/gone/already/agent/eligible/expired) stop at once. Guards: `--expectNft <n>` executes only if exactly n NFTs arrive. Tune: --waitOn/--fatalOn, --once, --clockFire -1, --gasLimit only on request, FASTMINT_RPCS override. Premium RPC in .env: FASTMINT_RPCS_EXTRA / FASTMINT_WS_EXTRA (mode 600, never in logs). Logs: runs/<stamp>_<label>.log (WIB + T+ offsets).
4. **Report:** tx hash, status, BscScan link, TIMELINE line, what was received, log path; if stopped: decoded revert + meaning.

## Troubleshooting
UNCONNECTED/SSL mid-fire: stop, tell user; only on OK run `baw auth signin --json` then `auth verify --qrCodeId`. 351802/351804: user enables Developer Mode. NotStarted/351805 forever: verify start getter vs site. Sold out: terminal — never switch to paid without explicit "ok paid" + amount. AgentIdRequired: pass "$agentId". PaymentLow/WrongValue: value = price x qty, re-confirm with user. Preview RISK/351803: stop, show risk text, do not bypass. PENDING_CONFIRMATION: user approves in app; do not resend. Execute error/no receipt: do NOT retry — `baw wallet tx-history --json` and report. "gave up after N previews": backend node vs public RPC state mismatch — check preview JSON in runs/.