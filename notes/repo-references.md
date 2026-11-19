# Referensi Repo (dipelajari 19 Nov 2026, permintaan bre)

## Koleksi setyamickala (community agent skills)
### 1. fastmint-fcfs-agentic — https://github.com/setyamickala/fastmint-fcfs-agentic
FCFS mint di BSC via Binance Agentic Wallet (`baw` CLI), masuk blok ±2 detik setelah jam buka. Skill AI-agent, tanpa private key di mesin (signing di backend Binance via Developer Mode).
- Alur: recon (HTML/JS/llms.txt → address, chainId, mint fn, ABI, unix time) → `contract-call preview` (requestId, valid 300s, bisa di-pre-generate SEBELUM jam buka) → `contract-call execute` di momen tepat (trigger via WS newHeads).
- Timing terukur: preview 0.45–0.8s, execute→broadcast 0.68–0.71s, block time BSC ±0.5s, RPC tercepat QuikNode/bloXroute/48.club (31/32/45ms), offset NTP dikoreksi otomatis.
- Prasyarat: Node ≥20, `npm i -g @binance/agentic-wallet@1.12.0`, Developer Mode ON, saldo di bawah cap, skill `binance-agentic-wallet` dari binance-skills-hub.
- .env: FASTMINT_RPCS_EXTRA / FASTMINT_WS_EXTRA (premium RPC), FASTMINT_TZ default Asia/Jakarta.
- Error codes: 351802/351804 = Developer Mode off/cap; 351803 = risk blocked (STOP, jangan bypass); 351805 = simulation failed (normal sebelum buka); UNCONNECTED = sesi expired 2 hari (user login manual). DNS hijack ISP (internetpositif) → encrypted DNS/VPN, jangan ubah system DNS dari agent.
- Hanya BSC. File: fastmint.sh, fire.js, lib.js, recon.js, bench.js, SKILL.md.

### 2. bsc-contract-audit-skill — https://github.com/setyamickala/bsc-contract-audit-skill (MIT)
Dua skill due-diligence BSC (chain 56), output `DECISION: GO | CAUTION | BLOCK` yang bisa gate wallet agent:
- `bsc-contract-audit` — audit ASET & ESCROW (buyer-side): BEP-20, NFT mint (721/1155), lock refund escrow. Skor 0–100: <55 BLOCK, 55–84 CAUTION, ≥85 GO.
- `launch-curve-audit` — audit MEKANISME JUAL (pre-launch): bonding curve/tranche IEO, escrow, graduation & LP custody, coupled emissions. Verdict HOLD→CAUTION, SHIP→GO, BLOCK→BLOCK.
- Runtime: Python + curl/jq read-only, tanpa dependensi; `export RPC_URL=https://bsc-rpc.publicnode.com`, optional ETHERSCAN_KEY. GoPlus optional; stack murah = premium RPC + BscScan key + Sourcify + honeypot.is + audit API publik Binance. Holder concentration dari Transfer logs, integritas escrow dari `rpc.py nbal`.
- Hasil cepat vs audit dalam bertabrakan → yang lebih ketat menang.

## Daftar nomor bre
### 1. cloudflare/security-audit-skill — https://github.com/cloudflare/security-audit-skill (MIT)
Skill audit keamanan multi-fase untuk coding agent; seed dari vulnerability harness Cloudflare (blog "Build your own vulnerability harness").
- 6 fase: (1) Recon → architecture.md + coverage-ledger.json; (2) Coverage-led hunting (hunter terisolasi + coverage critics); (3) Candidate validation (verifier baru tiap kandidat, coba BANTAH); (4) Structured output findings.json (confirmed/needs_validation/rejected) vs report-schema.json; (5) Independent record verification (verifier baru lagi); (6) Reporting REPORT.md / FINDINGS-DETAIL.md / NEEDS-VALIDATION.md.
- Prinsip: hanya konfirmasi boundary failure yang sudah mapan; adversarial validation (pemverifikasi ≠ penemu); severity = likelihood × impact; defense-in-depth gap = hardening note, bukan vuln; run berulang bersifat aditif (1 run ≈ setengah dari total vuln).
- File penting: SKILL.md, ATTACK-CLASSES.md, CLOUD-AND-DEPLOYMENT.md, PROTOCOLS-RPC-AND-MESSAGING.md, DATA-ISOLATION-AND-LIFECYCLE.md, DESKTOP-MOBILE-AND-LOCAL-IPC.md, VALIDATION-AND-REPORTING.md + validator .cjs zero-dependency.
- Butuh sandbox OS (no network eksternal, allowlist env, resource limit); tanpa itu kandidat stays `needs_validation`. Install: `npx skills add ... --skill security-audit`.

### 2. NVIDIA/auto-ontology — https://github.com/NVIDIA/auto-ontology (Apache-2.0)
Framework Python untuk lapisan reasoning semantik/graph di atas data enterprise: ontologi auto-generate dari metadata, rekonsiliasi definisi antar sistem, expose ke agent via MCP/CLI/API.
- 5 fungsi: Discover (ontology dari metadata + knowledge graph), Ground (makna bisnis terpercaya untuk metrik/entitas), Validate (uji draft dgn pertanyaan nyata, SQL + langkah di-attach tiap jawaban), Expose (MCP/CLI/API), Learn (koreksi tersertifikasi masuk balik ke ontologi).
- 3 skill agent: nvidia-ontology-setup, nvidia-ontology-management, nvidia-ontology-query (mulai dari AGENTS.md). Ada frontend UI, alembic, helm chart, mcp/ dir.

### 3. vercel-labs/agent-desktop — https://github.com/vercel-labs/agent-desktop (Apache-2.0)
Computer-use CLI untuk agent: operasikan app desktop macOS/Windows/Linux via OS ACCESSIBILITY TREE — tanpa screenshot/pixel matching. Baca tree → JSON dengan `ref` per elemen → agent click/type by ref → baca ulang → loop. Rust (crates/) + C lib + npm `agent-desktop`.
- Command: snapshot/find/get/is/screenshot; click/type/set-value/focus/select/toggle/check/scroll; press/hover/drag/mouse-*/key-*; launch/close-app/list-apps/focus-window/resize/move/minimize; status/permissions/batch/skills.
- `agent-desktop skills get desktop` + `get platform` = aturan loop/refs/verifikasi. Install skill: `npx skills add ... --skill agent-desktop`. Privacy: snapshot/clipboard bisa berisi konten privat — jangan upload ke log publik.

### 4. meta/astryx — https://github.com/meta/astryx (MIT, Beta)
Design system open source Meta (8 tahun internal, 13.000+ apps, design system terbesar di Meta). React 19+ + StyleX. 150+ komponen accessible, brand theming (CSS custom property overrides — customisasi tanpa fork/wrap), dark mode, templates, CLI `@astryxdesign/cli` (docs, scaffolding, themes, codemods), 7 tema siap pakai (neutral/butter/chocolate/matcha/stone/gothic/y2k).
- Overridable via className (Tailwind/CSS modules/plain CSS) — no styling lock-in; komponen open internals, "swizzle" eject source. Docs: astryx.atmeta.com. Charts/vega masih @canary.

### 5. anthropics/skills — https://github.com/anthropics/skills
Repo resmi Agent Skills Anthropic: contoh skill (creative, dev/technical, enterprise/comms, document skills), spec Agent Skills (agentskills.io) di ./spec, template di ./template. Install via Claude Code `/plugin install document-skills@anthropic-agent-skills` atau example-skills.

## Toolkit bug hunting
### 6. 0xKayala/NucleiFuzzer — https://github.com/0xKayala/NucleiFuzzer
Otomasi web vuln scanning = Nuclei(fuzzing-templates) + ParamSpider + waybackurls + gauplus + hakrawler + katana, difilter uro (dedupe) + httpx. Deteksi XSS, SQLi, SSRF, Open Redirect.
- Prasyarat: semua tool di atas terinstall. Usage: `nf -d domain` atau `nf -f file.txt -o outdir`. Install via install.sh. Fitur baru: recon+crawl otomatis, JSON+HTML report, AI vuln analysis, grouping severity.

### 7. awarexone/Agentic-Bug-Hunter — https://github.com/awarexone/Agentic-Bug-Hunter (MIT, ~6k stars)
CLI AI-powered bug bounty end-to-end (recon → find → validate → report). Install: `uv tool install agentic-bug-hunter` → `bughunter setup` (AI provider gratis) → `bughunter hunt target.com` atau `/hunt` di Claude Code. Output `~/.bughunter/`.
- Full recon pakai subfinder, httpx, nuclei, katana, ffuf, nmap (install_tools.sh). Ada AXguard, plugin Claude Code, dir skills/, commands/, web3/.
- Khusus authorized testing dalam scope program bug bounty.

## Catatan pemakaian (untuk agent Hermes sendiri)
- BSC audit skill cocok dijalankan SEBELUM fastmint (gate DECISION: GO/CAUTION/BLOCK) → pipeline: launch-curve-audit → bsc-contract-audit → fastmint-fcfs.
- cloudflare security-audit-skill cocok untuk audit kode repo sendiri sebelum publikasi (pola bagus: verifier terpisah dari finder).
- agent-desktop relevan kalau perlu automasi desktop; di workenv cloud SSH tidak ada desktop GUI asli — gunakan browser tool.
