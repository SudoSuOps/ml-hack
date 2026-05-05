# Agent Notes

## Local Dev Servers

- Frontend: from `frontend/`, run `npm ci` if dependencies are missing, then `npm run dev`.
- Backend: from `backend/`, run `uv run uvicorn main:app --host ::1 --port 7860`.
- Frontend URL: http://localhost:5173/
- Backend health check: `curl -g http://[::1]:7860/api`
- Frontend proxy health check: `curl http://localhost:5173/api`

Notes:

- Vite proxies `/api` and `/auth` to `http://localhost:7860`.
- If `127.0.0.1:7860` is already owned by another local process, binding the backend to `::1` lets the Vite proxy resolve `localhost` cleanly.
- Prefer `npm ci` over `npm install` for setup, since `npm install` may rewrite `frontend/package-lock.json` metadata depending on npm version.
- Production defaults to the Bedrock Claude model. For local development with a personal Anthropic key, set `ANTHROPIC_API_KEY` and `ML_HACK_CLAUDE_MODEL_ID=anthropic/claude-opus-4-7` before starting the backend. Other models are selected through the app's model switcher.

## Swarm & Bee infrastructure (operating environment)

ML-Hack assumes you have access to the firm's compute and data infrastructure:

- **swarmrails** @ 192.168.0.100 — primary cook box (2× RTX PRO 6000 96GB · Xeon w9-3475X · 256GB DDR5 · 3.6TB NVMe)
- **smash** @ 192.168.0.164 — RTX 5090 32GB · Hack-tier cook + dial production
- **whale** — RTX 3090 Ti 24GB · currently rented via vast.ai
- **Honey ledger** — `/data2/swarmdev/honey_ledger.db` on swarmrails (SQLite WAL · 7-table provenance chain)
- **Hedera mainnet** — operator `0.0.10291827` · deed anchor topic `0.0.10291838`
- **Tribunal scales** — gemma3:12b + qwen2.5:32b (different families) on swarmrails GPUs 0+1
- **DG-WIKI-GRAPH** — first vertical corpus · `git@github.com:SudoSuOps/DG-WIKI-GRAPH.git`

When ML-Hack queues a cook, it dispatches to swarmrails. When it audits a corpus, it reads from the Honey ledger. When it publishes a deed, it goes to Hedera. There's no mocking; the infrastructure is the product.

## Locked recipes

ML-Hack always reads the locked training recipe before starting a cook:

- **Recipe #1 · FSDP-QLoRA bf16** — Atlas-tier (70B). Source: `atlas/recipes/recipe_1_70b_atlas.md`. Proven on the May 3 Atlas-70B v1 cook.
- **Recipe #2 · LoRA r=64 alpha=32** — 27B specialist. Source: `atlas/recipes/recipe_2_27b_specialist.md`.
- **Recipe #3 · QLoRA r=32 alpha=16** — 9B Hack-fleet next-tier. Source: `atlas/recipes/recipe_3_9b_hack.md`.
- **Recipe #4 · QLoRA r=64 alpha=32** — 4B Hack-fleet primary. Source: `atlas/recipes/recipe_4_4b_hack.md`. **Locked base: Qwen3.5-4B-Instruct.**

Deviation from a locked recipe requires written justification in the cook log.

## Development Checks

- Before every commit, run `uv run ruff check .` and `uv run ruff format --check .`.
- If formatting fails, run `uv run ruff format .`, then re-run the Ruff checks before committing.

## GitHub CLI

- For multiline PR descriptions, prefer `gh pr edit <number> --body-file <file>` over inline `--body` so shell quoting, `$` env-var names, backticks, and newlines are preserved correctly.

## Hugging Face Space deploys (optional surface)

If we mirror ML-Hack to an HF Space for public demo (parallel to upstream's `smolagents/ml-intern` Space), the deploy flow follows the same pattern:

```bash
# space remote points to https://huggingface.co/spaces/swarmandbee/ml-hack (when created)
git pull --ff-only origin main
git switch space-main
git config merge.ours.driver true
git merge --no-ff origin/main -m "Deploy $(date +%Y-%m-%d)"
git push space space-main:main
git switch main
```

Keep the Space-only README frontmatter on `space-main`. `.gitattributes` should contain `README.md merge=ours` and the local repo config should include `merge.ours.driver=true`.

## Cook auditor protocol

Every cook on swarmrails is monitored by the cook auditor (3-hour cron). The kill switch fires at >1% think-tag contamination. ML-Hack must NOT bypass or disable the auditor — it's the firm's defense against Atlas-v1-class corpus disasters.

## Memory (firm-wide context)

The user's cross-session memory lives at `~/.claude/projects/-home-swarm-Desktop/memory/`. Key files relevant to ML-Hack:

- `atlas_firm_os.md` — overall firm architecture
- `the_lane_stnl_thesis.md` — STNL is the lane (positioning)
- `the_hacks_lineage.md` — named Hack lineage (Elon, Alec, AJ, Harvey)
- `hack_stnl_dg_first_cook.md` — first Hack cook plan
- `unit_economics_thesis.md` — 1% × 100 deals math
- `gold_standard_70b_meta_cookbook.md` — proven 70B recipe
- `cook_auditor_protocol.md` — kill-switch protocol
- `data_quality_protocol.md` — mandatory pre-cook checks
- `the_hedera_ecosystem.md` — anchor topology

Read these when context is missing.
