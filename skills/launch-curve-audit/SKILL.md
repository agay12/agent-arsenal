---
name: launch-curve-audit
description: Use to audit a BSC bonding-curve/IEO launch before it opens.
metadata:
  author: setyamickala (installed from github.com/setyamickala/bsc-contract-audit-skill, MIT)
  source: /root/skills-repos/bsc-contract-audit-skill/skills/launch-curve-audit (full SKILL.md, references/ economics+invariants+attack-surface, scripts/ rpc.py, curve_math.py, snapshot.sh — cloned on machine)
  version: 1.0.1
---

# Launch Curve Audit

Trigger: auditing a BSC token launch curve pre-launch/pre-investment — bonding curve or tranche IEO, escrowed bids, graduation into LP, coupled post-launch emissions. Returns SHIP/HOLD/BLOCK + DECISION line. Not for post-listing trading risk (use bsc-contract-audit).

Object = the SALE MECHANISM: pricing, admission, escrow, graduation, LP custody, coupling to the emission contract. The money is the buyer's and leaves through the sale contract.

## Intake
Chain/addresses (on a SPA, grep `0x[a-fA-F0-9]{40}` in the JS bundle, not the page), curve parameters (start/end price, supply cap, budget cap, per-identity cap, duration), admission model, graduation trigger + LP destination, emission contract, verified source (Sourcify first, Etherscan v2 chainid=56). An address without params is still auditable — read from constructor args and bytecode. Never accept params from docs alone.

## Evidence rules
1. Bytecode + on-chain state at a pinned block; 2. Verified source exact_match only; 3. Forked simulation of the exact call sequence; 4. Constructor args + deployment tx; 5. Docs/site = claims to be diffed (claim-vs-code gap is a finding of its own). Price every claim in BNB and token units — "cheap" without a number is not a finding. A curve property proved only by a comment is unproven. A snapshot without a block is a rumour.

## Tooling
```bash
cd /root/skills-repos/bsc-contract-audit-skill/skills/launch-curve-audit
export RPC_URL=https://bsc-rpc.publicnode.com
./scripts/snapshot.sh roles.env                 # lifecycle, supply, privilege, balances; prints owner EOA-or-contract
cd .. && python3 scripts/curve_math.py --p <wei> --supply <units> --check 1e18,100e18 [--budget <wei>]   # monotonicity, rounding, step-vs-integral
```
Read the FULL SKILL.md at the source path for the complete workflow + references/ (economics.md, invariants.md, attack-surface.md).

## Gates (hard BLOCKs; custody is the classic HOLD)
G1 exit: buyers can't exit/refund -> BLOCK (funds trapped). G2 curve: undercharge (step-split pays less than integral; compare cost(n) vs cost(n/2)*2 vs closed form), overflow (mul-div without 512-bit mulDiv near 2^96 supply), cap admission silently overshoots -> BLOCK. G3 opening price settable (donate+sync) -> BLOCK. G4 graduation: not triggerable by anyone or LP recoverable (not burned/locked) -> BLOCK. G5 custody: owner EOA / unbounded setters (fees, tax, caps, curve endpoints, pair, fee recipient) -> HOLD. Caps: owner EOA (HOLD), launch owner-gated (HOLD), >half supply insider-allocated (HOLD), promised refund missing (BLOCK).

## Adversary catalogue (attack-surface.md)
Donate-then-sync to move opening price; buy first unit then trigger tax swap in the gap; amountOutMin=0 on the sale's own swaps; refund reentrancy + one-sided refund accounting; front-run the cap (last-block sniping); claim paths whose gas grows with entries (rewards unclaimable).

## Coupling & admission
Post-launch mint: budget-bound (monotonic carryover, not per-period reset)? minter list lockable, owner out? cap clamps or reverts everything? sale owner can swap token/emission contract afterwards? Allowlist: identities, per-identity allocation, publication time, who can add — a 500-wallet allowlist on 1,000 units is insider capture. tx.origin gating breaks smart wallets (record as behaviour, not bug).

## Verdict
SHIP = curve, invariants, graduation, custody all proven (residual Info). HOLD = exits work, curve sound, but open admin path or unproven invariant. BLOCK = exit-insecure / cap blowable / price settable / LP recoverable. Map: SHIP->GO, HOLD->CAUTION (proceed only after user acknowledges caps), BLOCK->BLOCK. End with `DECISION: GO | CAUTION | BLOCK — <reason>`. Hand off the graduated asset to bsc-contract-audit (TOKEN/MINT/LOCK). If this audit and the wallet's own query-token-audit disagree, the stricter result wins.

Report: header (block, source exact_match, owner type, cap, sold), verdict + caps, Economics line (price path, integral-vs-stepped delta), findings table (ID/Sev/Phase/Function/Trigger/Invariant), invariants, unproven, recommendations, evidence log, snapshots. Every finding names phase + function + broken invariant. Close with `Reference only, not investment advice.`

Common misreads: sold out != healthy (check team's own wallets); renounced owner on an unlocked emission contract means nothing; "only 1% tax" — read _transfer (what %, when, whom); donation moves the opening price; absent getter != absent feature; passing tests prove what they assert and nothing else — price the arithmetic yourself; graduation "looks automatic" but if behind onlyOwner the sale can stay open forever.