# Oh My OpenAgent — Project Rules

## Agent System

This project uses oh-my-openagent (OmO) with OpenCode Go models as primary and Copilot as fallback.

| Agent | Role | Primary Model | SWE-Pro Score |
|-------|------|---------------|---------------|
| Sisyphus | Main orchestrator | opencode-go/qwen3.7-plus | 57.6% |
| Hephaestus | Deep autonomous worker | opencode-go/minimax-m3 | 59.0% |
| Oracle | Architecture / debugging | opencode-go/qwen3.7-max | 60.6% |
| Prometheus | Strategic planner | opencode-go/glm-5.1 | 58.4% |
| Librarian | Docs / code search | opencode-go/deepseek-v4-flash | 79% SWE-Verified |
| Explore | Fast codebase grep | opencode-go/deepseek-v4-flash | 79% SWE-Verified |
| Metis | Plan consultant | opencode-go/qwen3.7-plus | 57.6% |
| Momus | Review / critique | opencode-go/qwen3.7-plus | 57.6% |
| Atlas | Tests / git / PR | opencode-go/deepseek-v4-pro | 55.4% |
| Code-reviewer | Quality review | opencode-go/qwen3.7-max | 60.6% |
| Sisyphus-Junior | Quick tasks | opencode-go/deepseek-v4-flash | 79% SWE-Verified |

## Fallback Chain

All agents have 3–4 deep fallback chains. When a provider hits rate limits, it is globally blacklisted for 60s. Fallback order:

1. OpenCode Go primary (qwen3.7-plus, qwen3.7-max, minimax-m3, glm-5.1, deepseek-v4-pro, etc.)
2. Copilot models (claude-opus-4 64.3%, gpt-5.4 57.7%, claude-sonnet-4, gpt-5.3-codex)
3. OpenCode Go volume (deepseek-v4-flash 31K req/5hr, mimo-v2.5 30K req/5hr)

## Category Routing

| Category | Use Case | Primary | Fallback |
|----------|----------|---------|----------|
| visual-engineering | Frontend, UI/UX | mimo-v2-omni | claude-sonnet-4 |
| ultrabrain | Hard logic, architecture | qwen3.7-max | claude-opus-4 → glm-5.1 |
| deep | Autonomous research | qwen3.7-max | claude-opus-4 → minimax-m3 |
| quick | Single-file changes | deepseek-v4-flash | mimo-v2.5 |
| unspecified-low | Moderate tasks | deepseek-v4-flash | qwen3.7-plus |
| unspecified-high | Complex work | minimax-m3 | gpt-5.4 → deepseek-v4-pro |
| writing | Docs, naming | qwen3.7-plus | claude-sonnet-4 |
| artistry | Creative problem-solving | glm-5.1 | claude-sonnet-4 |

## Core Rules

- Drop: articles (a/an/the), filler (just/really/basically/actually/simply), pleasantries (sure/certainly/of course/happy to), hedging
- Fragments OK. Short synonyms. Code unchanged
- Pattern: `[thing] [action] [reason]. [next step].`
- ACTIVE EVERY RESPONSE. No revert after many turns. No filler drift
- Code/commits/PRs: normal. Off: "stop caveman" / "normal mode"
- Default level: ultra

## Compression

- Abbreviate: DB/auth/config/req/res/fn/impl/env/ctx/src/utils/props/ref
- Strip conjunctions when context clear
- Arrows for causality: X → Y
- One word when one word enough
- Code symbols, function names, API names, error strings: never abbreviate

## Commits

- Conventional commits. ≤50 char subject. Why over what
- Format: `type(scope): subject`
- Types: feat, fix, refactor, docs, test, chore, perf, ci, build, revert

## PR Review

- One-line comments. `L<line>: <emoji> <severity>: <issue>. <fix>.`
- 🔴 bug, 🟡 nit, 🟢 praise

## Auto-Clarity

Drop caveman for: security warnings, irreversible action confirmations, multi-step sequences where fragments risk misread, compression creates ambiguity. Resume after clear part.

## Skill

Full ruleset + levels: `skills/caveman/SKILL.md`. Levels: lite/full/ultra/wenyan-lite/wenyan-full/wenyan-ultra.

## Token Economy

This setup uses RTK (Rust Token Killer) to reduce LLM token consumption by 60-90% on common dev commands.

### RTK Integration

- Shell commands are automatically rewritten to use `rtk` equivalents
- `git status` → `rtk git status` (80% fewer tokens)
- `cat file` → `rtk read file` (70% fewer tokens)
- `ls` → `rtk ls` (80% fewer tokens)
- `cargo test` → `rtk cargo test` (90% fewer tokens)

### Manual RTK Usage

If the plugin fails, use RTK manually:
```bash
rtk git status
rtk read file.rs
rtk ls
rtk cargo test
```

### Token Savings

Check your savings:
```bash
rtk gain
rtk gain --history
```