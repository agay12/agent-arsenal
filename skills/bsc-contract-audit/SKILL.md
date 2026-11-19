---
name: bsc-contract-audit
description: Use for BSC token/NFT/lock safety audit before buying.
metadata:
  author: setyamickala (installed from github.com/setyamickala/bsc-contract-audit-skill, MIT)
  source: /root/skills-repos/bsc-contract-audit-skill/skills/bsc-contract-audit (full SKILL.md, references/, scripts/rpc.py, binance_audit.sh — cloned on machine)
  version: 3.2.0
---

# BSC Contract & NFT Mint Audit

Trigger: user supplies a BSC (chain 56) contract address — BEP-20 token, NFT mint, or deposit-refund lock escrow — and wants a safety/scam/rug judgement, scoring, or a pre-buy gate ending in DECISION: GO | CAUTION | BLOCK.

Three tracks share one rubric: **TOKEN** (BEP-20: can owner hurt you, can you sell, LP locked), **MINT** (BEP-721/1155 primary sale: team takes fee & delivers nothing? resale? supply capped? art frozen?), **LOCK** (deposit-refund escrow: owner drains pool / pauses refund? unlock fixed? allocation real?). Classify the track first (`ownerOf(1)`/`tokenURI` succeed -> MINT; deposit/lock + unlockTime + allocation -> LOCK), probes differ, score does not.

## Evidence rules (trust order, highest first)
1. On-chain state (eth_call / eth_getStorageAt / getReserves / ownerOf at a PINNED block — always record the block).
2. Verified source — Sourcify first (no key), Etherscan v2 second (exact_match only).
3. Simulation — honeypot.is (token) or mint eth_call with value (NFT), chain 56.
4. Binance Web3 audit API — one input, never the verdict (read isHit, not title; riskLevel LOW is token-shaped, means little on NFTs).
5. Aggregators — GoPlus optional, DexScreener; leads only. Holder concentration from `scripts/rpc.py holders` (Transfer-log reconstruction), not GoPlus.

Prohibited: judging by name/logo/art/age/followers; writing "safe" for anything never probed (write `unknown`, score partial); treating a selector in bytecode as a live feature (confirm via ABI+modifier+reachability); concluding from proxy bytecode (resolve implementation first); publishing a score without the block number and command.

## Tooling
```bash
cd /root/skills-repos/bsc-contract-audit-skill/skills/bsc-contract-audit
export RPC_URL=https://bsc-rpc.publicnode.com
python3 scripts/rpc.py block|meta|code|call|proxy|holders|scan $ADDR ...
python3 scripts/rpc.py call $ADDR "ownerOf(uint256)(address)" 1   # reverts on BEP-20
./scripts/binance_audit.sh   # DoH-resolves web3.binance.com past Telkomsel DNS hijack
```
Backup RPCs: bsc-dataseed.binance.org, bsc-dataseed1.defibit.io, 1rpc.io/bnb. Public nodes cap eth_getLogs ~1000 blocks (set LOGS_WINDOW or score holder concentration partial). Read the FULL SKILL.md at the source path for complete probe recipes, selector table, and references/ (code-red-flags, data-sources, selectors, probes).

## Gates (fail-fast hard caps, tightest wins)
TOKEN: exit sim fails -> cap 40 (honeypot); LP not locked/burned or owner EOA can pull -> AVOID; no exact_match source -> cap 55 low confidence; then privilege inventory (owner/pause/blacklist/hidden mint/upgrade/tax/maxTx/baseURI).
MINT: no resale -> cap 45; proceeds withdrawable before delivery -> cap 65; art mutable/no deadline -> cap 60-70; maxSupply uncapped -> cap 60.
LOCK: no self-exit after unlock -> cap 40; owner can drain pool -> cap 40; timer mutable/pausable -> cap 55; allocation asset must also pass TOKEN/MINT.

## Scoring & verdict
5 criteria -> /100: Security 30, Investor protection 25, Anti-manipulation 20, Transparency 15, Market viability 10. Apply tightest hard cap. Score <55 -> BLOCK, 55-84 -> CAUTION, >=85 -> GO. Always end with:
```
Score: n/100 — PASS/REVIEW/AVOID (binding caps)
DECISION: GO | CAUTION | BLOCK — <one-line reason>
```
Report: verdict, criterion table with evidence IDs, Critical/Moderate/Unproven/Recommendations, evidence log (command -> trimmed output). Every score cites replayable evidence. If a fast check and a deep audit disagree, the stricter result wins.

## Common misreads
owner() reverting != renounced (only a successful call returning zero is); supportsInterface reverts on pre-ERC165 (use ownerOf/tokenURI); tax 0 in simulation may miss pair-keyed/size-keyed tax (retest on real pair); audit API "Contract Renounced" with isHit:false means NOT renounced; LP "in a locker" must be confirmed on-chain now; revealed != delivered if baseURI mutable; sold out may be team wallets forcing state; SafeMoon-fork optics (reflection+auto-liquify+renounced) still leaves swapAndLiquify/setMaxTx — read the code.

Reference only, not investment advice.