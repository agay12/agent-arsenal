# Program selection — triaging multiple bounty programs

Use when the user gives several bounty-program links and expects a recommendation. Check submission mechanics BEFORE starting any hunt (see SKILL.md step 8); a program you cannot submit to is worth zero no matter how good the bug.

## Submission-gate decision table

| Gate type | What it needs | Agent workable? |
|---|---|---|
| Self-hosted email submit (security.txt / docs page) | Nothing — plain email | YES — ideal; verify the address exists via the site's `/.well-known/security.txt` |
| Vendor platform form (bugbounty.<vendor>.com/report) | Vendor account login (often same account as the product) | Only with a saved login (browser_vault) or the user submitting manually |
| Immunefi / Intigriti / H1 | Platform account; PoC mandatory; KYC + tax docs at payout time | Only with a user-supplied account; flag KYC at selection time, not after the bug |
| GitHub security advisory | Logged-in GitHub web session on a repo you have access to; API cannot create advisories | Only via browser + saved GitHub login; check token's repo access FIRST |

## Selection order (apply in sequence, stop at first differentiator)
1. **Gate** — prefer programs the agent can actually submit to unaided (email, or a platform already logged in). Record every program's gate in the triage output.
2. **KYC/payment requirements** — note programs that pay only after KYC/KYB; the user may not want identity verification even for a valid bug.
3. **Unaudited surface** — fresh code beats old flagship code: find the last-audited baseline commit and diff it against HEAD (SKILL.md step 3). A program whose contracts were last audited years ago by a crowd of researchers has a thin surface; one with post-audit commits is the highest-yield target.
4. **Scope breadth** — programs that include off-chain code (bots, relayers, front end) alongside contracts have surfaces researchers rarely cover; programs with narrow 'impacts in scope' lists kill generic findings before triage.
5. **Payout structure** — floors and caps matter less than eligibility breadth, but a min-reward clause for critical findings and a clear severity matrix make triage disputes less likely.

## Output shape
One block per program: payout, scope breadth, submission channel + its gate, KYC needed, unaudited surface size, and a ranked recommendation with the reason. Do not start hunting until the user picks (or the standing instruction says go autonomous).