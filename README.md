# ML-Hack

> **The STNL workhorse.** Cook, eval, and ship Hack-fleet specialists for the Atlas brokerage. Forked from [huggingface/ml-intern](https://github.com/huggingface/ml-intern) — same agentic loop, different desk.

ML-Hack is the CLI + web agent that takes a Hack from "no model" to "deployed dialer on a vertical." One specialist per asset class — DG, Walgreens, CVS, QSR, auto-retail, multifamily, industrial. Same script every time. The compute changes; the discipline doesn't.

```
   M.A.G.I.C.   →   meetings · appraisals · grind · ink · close
   the desk     →   trained on a vertical, deployed on a phone line
```

Built by **Swarm & Bee LLC** (D-U-N-S 138652395) for the **Atlas Hack-fleet**. Public site: [swarmandbee.ai](https://swarmandbee.ai). The pain page: [pain.defendable.eth](https://pain.defendable.eth.limo).

## What ML-Hack does

A Hack-fleet specialist is a 4-9B model trained on **one CRE asset class** with a **dial-friendly persona** that runs the MAGIC lifecycle (Meetings → Appraisals → Grind → Ink → Close) at machine speed. ML-Hack is the agent that makes those Hacks ship.

The seven steps from idea to live dialer:

1. **Pick the vertical** — DG, Walgreens, CVS, QSR, auto-retail, MF Sun Belt, etc.
2. **Pick the base model** — Qwen3.5-4B-Instruct (locked), Qwen3.5-9B for the next-tier
3. **Assemble the corpus** — query the vertical's WIKI-GRAPH (DG-WIKI-GRAPH precedent), structure pairs by lifecycle stage
4. **Generate training pairs** — supervised by Atlas-SMD (70B teacher) or Atlas-UW (27B teacher) per Recipe #4
5. **Cook the Hack** — queue on swarmrails (PRO 6000 96GB) or RTX 5090, monitor via cook auditor
6. **Eval + tribunal** — 9B+9B different-family judges, Royal Jelly grade required for Class A
7. **Deploy + anchor** — push to vLLM endpoint, wire voice stack (Whisper + F5-TTS + Twilio), Hedera-anchor the deed

## Architecture

Inherited from `ml-intern`. Replace the HF ecosystem with the Swarm & Bee ecosystem.

| Component | ml-intern (upstream) | ML-Hack (this fork) |
|---|---|---|
| **Agentic loop** | `agent/core/agent_loop.py` | unchanged · battle-tested |
| **Doom-loop detector** | repeated tool patterns | unchanged |
| **Tool router** | HF papers · datasets · jobs | CRE comps · Atlas dispatch · Honey ledger |
| **Approval gates** | jobs · sandbox · destructive | jobs · Hedera anchor · email blast |
| **Trace upload** | HF dataset (Claude Code JSONL) | swarm-db Honey ledger (same JSONL format) |
| **Notifications** | Slack webhook | Discord webhook (already wired) |
| **Frontend** | React/Vite | unchanged · re-skinned |
| **Backend** | FastAPI | unchanged · routes added for CRE tools |

## Tools (current + planned)

Inherited from upstream — kept:
- `research` — sub-agent crawls the web
- `web_search` — generic web lookup
- `github_*` — read CRE repos (atlas, swarmandbee-site, DG-WIKI-GRAPH)
- `plan` — multi-step task planning
- `notify` — Discord pings
- `sandbox` — local execution

Replacing / new (CRE-flavored):
- `hf_jobs_tool` → **`atlas_jobs_tool`** — dispatch cooks to swarmrails / vast.ai
- `hf_inspect_dataset` → **`honey_inspect`** — read pair JSONL from Honey ledger
- `hf_papers` → **`market_research`** — Boulder Group · Trepp · MBA · CRED iQ feeds
- new: **`dgwiki_query`** — query DG-WIKI-GRAPH for owners / comps / leases / contacts
- new: **`underwrite_tool`** — auto-BOV from rent roll + comps
- new: **`comp_lookup`** — recent trades by tenant / market / cap-rate band
- new: **`loi_gen`** — LOI / PSA template generator
- new: **`blast_tool`** — email blast to The Book (audited recipient list)
- new: **`hedera_anchor`** — publish deed Merkle root to HCS topic 0.0.10291838

## Quick Start

### Installation

```bash
git clone git@github.com:SudoSuOps/ml-hack.git
cd ml-hack
uv sync
uv tool install -e .
```

That's it. Now `ml-hack` works from any directory:

```bash
ml-hack
```

### `.env` setup

```bash
ANTHROPIC_API_KEY=<your-anthropic-api-key>      # if using anthropic models
OPENAI_API_KEY=<your-openai-api-key>            # if using openai models
HF_TOKEN=<hf-token>                              # for base model downloads
GITHUB_TOKEN=<github-pat>                        # for repo reads

# Swarm & Bee specific
SWARMRAILS_HOST=192.168.0.100                   # cook dispatch target
HONEY_LEDGER_DB=/data2/swarmdev/honey_ledger.db # provenance storage
HEDERA_OPERATOR=0.0.10291827                    # Hedera mainnet operator
HCS_TOPIC=0.0.10291838                          # deed anchor topic
DISCORD_WEBHOOK_URL=<your-discord-webhook>      # ops notifications
```

### Usage

**Interactive mode** — chat session for cooking a new Hack:

```bash
ml-hack
> cook hack-stnl-dg from dg-wiki-graph corpus on swarmrails
```

**Headless mode** — single-shot:

```bash
ml-hack "underwrite 391-unit MF deal at /tmp/om.pdf, return IC memo"
```

**Options:**

```bash
ml-hack --model anthropic/claude-opus-4-7 "cook hack-stnl-walgreens"
ml-hack --max-iterations 100 "eval hack-stnl-dg-v1 on the 180-prompt suite"
```

## The MAGIC sprint workflow

ML-Hack runs the firm's **6-week MAGIC sprint** as a state machine:

```
M — meetings        atlas queries graph → owner-by-zip dial list
A — appraisals      auto-bov → 1-sheet om
G — grind           personalized blast to the book
I — ink             loi/psa generated, sponsor signs
C — close           closing statement + hedera anchor
```

Each phase is gated on the prior phase's deed being anchored. The cycle is the receipt.

## Trace persistence

Every ML-Hack session is auto-uploaded to **swarm-db** in [Claude Code JSONL format](https://huggingface.co/changelog/agent-trace-viewer) — same format upstream uses for HF datasets, but the destination is the firm's Honey ledger. Each session becomes a `pair_id` in `honey_ledger.signals` with full provenance to the Hedera anchor. Glass-wall doctrine.

To opt out:

```json
{ "share_traces": false }
```

## Discord notification gateway

Same notification primitive as upstream's Slack gateway, wired to Discord. Drop the same webhook URL the `swarmandbee.ai` "Hand us a deal" form uses:

```bash
DISCORD_WEBHOOK_URL=https://discord.com/api/webhooks/...
ML_HACK_DISCORD_NOTIFICATIONS=true
ML_HACK_DISCORD_AUTO_EVENTS=approval_required,error,turn_complete
```

## Development

```bash
# Backend (FastAPI)
cd backend && uv run uvicorn main:app --host ::1 --port 7860

# Frontend (Vite)
cd frontend && npm ci && npm run dev   # http://localhost:5173

# Lint + format (matches upstream)
uv run ruff check .
uv run ruff format --check .
```

## Provenance · upstream attribution

Forked from `huggingface/ml-intern@main` on May 4, 2026. We keep the agentic loop, doom-loop detector, context manager, approval policy, session uploader, and tool-router pattern verbatim — they're battle-tested by the smolagents team. We rebrand the surface (CLI banner, system prompt, tool descriptions, notification destinations) and replace HF-ecosystem tools with CRE-ecosystem tools.

Original is **Apache 2.0** — license preserved. Every commit on this fork is signed-off; upstream attribution stays in `git log`.

## License

Inherits Apache 2.0 from upstream. See `LICENSE` (when added; otherwise see upstream).

## Brand glossary

- **Hack** — junior broker (insider CRE term · doubles as tech-shortcut pun). Each Hack-fleet specialist is one 4-9B model trained on one vertical.
- **The Book** — relationship + counterparty + deal graph. Firm-owned moat.
- **MAGIC** — 5-letter lifecycle vocabulary (Meetings · Appraisals · Grind · Ink · Close)
- **The Lane** — STNL net-lease at $1M-$5M (where AI dominates)
- **Atlas-as-a-Closer** — vendor service line: REITs/operators rent the Hack-fleet
- **V/V/V** — Verified · Vetted · Virtu (the bar Atlas climbs to)
- **Royal Jelly** — top tier of training-pair grade (Class A under Defendable v0.1.0)
