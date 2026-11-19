---
name: security-audit
description: Use when auditing codebases for security vulnerabilities.
metadata:
  author: cloudflare (installed from github.com/cloudflare/security-audit-skill, MIT)
  source: /root/skills-repos/security-audit-skill/skills/security-audit (full SKILL.md, phase docs, report-schema.json, validators — cloned on machine)
---

# Security Audit (Cloudflare)

Trigger: user asks for a security audit, to find vulnerabilities in a codebase, pen-test code, or when a security question needs structured hunting. Direct codebase audit/pen-test request = full audit mode (default output `~/security-audit-skill/<repo>/run-<N>`); security questions / focused vuln work = guidance mode unless report artifacts requested.

Orchestrate isolated agents through 6 phases:
1. **Reconnaissance** — map architecture, trust boundaries, input surfaces, prior evidence, deterministic coverage into `architecture.md` and `coverage-ledger.json`. Then run `node validate-coverage-ledger.cjs` (zero-dependency, in the source dir).
2. **Coverage-led hunting** — assign isolated hunter agents from ledger units, record their checks, use coverage critics to find gaps. Hunt-class prompts live in the source dir: ATTACK-CLASSES.md (core/wildcard/obvious), CLOUD-AND-DEPLOYMENT.md (IAM, IaC, containers, serverless), PROTOCOLS-RPC-AND-MESSAGING.md, RESOURCE-EXHAUSTION-AND-AVAILABILITY.md, DATA-ISOLATION-AND-LIFECYCLE.md (tenant isolation, cache, backup), DESKTOP-MOBILE-AND-LOCAL-IPC.md.
3. **Candidate validation** — every unique candidate goes to a FRESH verifier that tries to DISPROVE it (adversarial validation: the checking agent is never the finding agent).
4. **Structured output** — write confirmed / needs_validation / rejected records to `findings.json`, validate with `node validate-findings.cjs`.
5. **Independent record verification** — fresh agents verify final source claims; material replacements get another independent verifier, then re-validate.
6. **Target-neutral reporting** — derive REPORT.md, FINDINGS-DETAIL.md, NEEDS-VALIDATION.md from verified records + coverage ledger.

## Verdict semantics (distinct, never blur)
- `confirmed`: complete source trace + bounded observed result.
- `needs_validation`: an exact unresolved fact, NO severity assigned.
- `rejected`: records a disproved candidate.
Validate against `report-schema.json`.

## Design principles
- Only confirm established boundary failures; a source-grounded blocked lead stays needs_validation with its exact unresolved fact.
- Severity requires impact = likelihood x impact, not checklist deviation.
- Defense-in-depth gaps are NOT vulnerabilities — if layer A prevents the attack, missing layer B is a hardening note.
- Multiple runs are additive: prior ledgers/findings target gaps, revalidate changed source, carry forward current-source evidence without counting stale work as covered. One run finds roughly half of what repeated runs find.

## Requirements
Needs tool-use model with parallel subagents + Node for validators. Full audit mode needs an OS-enforced sandbox (no external network, sanitized allowlisted env, resource limits, writes only to scratch paths); without it, keep leads as needs_validation instead of executing target code. Write inside the target repo only in a vcs-ignored dir. Read the full SKILL.md + VALIDATION-AND-REPORTING.md at the source path for complete prompts.