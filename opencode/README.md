# Oh My OpenAgent Config

OpenCode setup with oh-my-openagent (OmO) orchestration. **OpenCode Go models are primary. Copilot models are fallback.**

## Files

| File | Purpose | Description |
|------|---------|-------------|
| `opencode.json` | Main config | Plugin registration, MCP, custom commands |
| `oh-my-openagent.json` | OmO config | Agent-to-model mapping, fallback chains, categories, concurrency |
| `AGENTS.md` | Rules injection | Project rules, agent reference, caveman mode guidelines |
| `skills/caveman/` | Caveman skill | Ultra-compressed communication mode |

## Model Architecture

### Tier 1 — OpenCode Go Elite (Primary)

| Model | SWE-Pro | req/5hr | Role |
|-------|---------|---------|------|
| `opencode-go/qwen3.7-max` | 60.6% | 950 | Oracle, Code-reviewer |
| `opencode-go/qwen3.7-plus` | 57.6% | 4,300 | Sisyphus, Metis, Momus, writing |
| `opencode-go/minimax-m3` | 59.0% | 3,200 | Hephaestus, unspecified-high |
| `opencode-go/glm-5.1` | 58.4% | 880 | Prometheus, long-horizon |
| `opencode-go/kimi-k2.6` | 58.6% | 1,150 | Complex tasks fallback |

### Tier 2 — OpenCode Go Standard

| Model | SWE-Pro | req/5hr | Role |
|-------|---------|---------|------|
| `opencode-go/deepseek-v4-pro` | 55.4% | 3,300 | Atlas, standard engineering |
| `opencode-go/qwen3.6-plus` | — | 3,450 | Volume fallback |

### Tier 3 — OpenCode Go Volume (Never Rate-Limited)

| Model | SWE-Verified | req/5hr | Role |
|-------|--------------|---------|------|
| `opencode-go/deepseek-v4-flash` | 79% | 31,650 | Librarian, Explore, Sisyphus-Junior |
| `opencode-go/mimo-v2.5` | — | 30,100 | Volume fallback |

### Tier 4 — Copilot Frontier (Fallback)

| Model | SWE-Pro | SWE-Verified | Role |
|-------|---------|--------------|------|
| `copilot/claude-opus-4` | 64.3% | 87.6% | Fallback for Sisyphus, Oracle |
| `copilot/gpt-5.4` | 57.7% | — | Fallback for Hephaestus, Atlas |
| `copilot/claude-sonnet-4` | — | — | Fallback for Prometheus, Metis, Momus |
| `copilot/gpt-5.3-codex` | — | — | Fallback for Atlas, coding |

## Agents

| Agent | Role | Primary | Fallback Chain |
|-------|------|---------|----------------|
| Sisyphus | Main orchestrator | qwen3.7-plus | qwen3.7-max → claude-opus-4 → gpt-5.4 → minimax-m3 |
| Hephaestus | Deep worker | minimax-m3 | gpt-5.4 → gpt-5.3-codex → deepseek-v4-pro → deepseek-v4-flash |
| Oracle | Architecture / RCA | qwen3.7-max | claude-opus-4 → claude-sonnet-4 → glm-5.1 → kimi-k2.6 |
| Prometheus | Planner | glm-5.1 | claude-sonnet-4 → gpt-5.4 → qwen3.7-max → deepseek-v4-pro |
| Librarian | Search | deepseek-v4-flash | mimo-v2.5 → claude-sonnet-4 |
| Explore | Fast grep | deepseek-v4-flash | mimo-v2.5 |
| Metis | Plan consultant | qwen3.7-plus | claude-sonnet-4 → qwen3.6-plus → deepseek-v4-pro |
| Momus | Review | qwen3.7-plus | claude-sonnet-4 → qwen3.6-plus → kimi-k2.6 |
| Atlas | Ops / tests | deepseek-v4-pro | gpt-5.3-codex → minimax-m3 → deepseek-v4-flash |
| Code-reviewer | Quality | qwen3.7-max | claude-opus-4 → claude-sonnet-4 → kimi-k2.6 → deepseek-v4-pro |
| Sisyphus-Junior | Quick tasks | deepseek-v4-flash | mimo-v2.5 |

## Commands

| Command | Agent | Description |
|---------|-------|-------------|
| `/ship` | sisyphus | End-to-end implementation |
| `/ultrawork` | sisyphus | Full team activation |
| `/plan` | prometheus | Strategic planning |
| `/code` | hephaestus | Fast coding pass |
| `/ops` | atlas | Tests, git, PR |
| `/rca` | oracle | Root cause analysis |
| `/review` | code-reviewer | Final review |
| `/judge` | momus | Second opinion |
| `/jira` | jira | Jira MCP workflow |
| `/caveman` | — | Toggle caveman mode |

## Fallback Behavior

```
Rate limit / error → cooldown provider 60s → try next fallback model
Max 3 fallback attempts per request
Notify on every fallback switch
```

Go models fail → fallback to Copilot. Copilot fails → fallback to Go volume models.

## Installation

### Prerequisites
- OpenCode 1.0.133+
- OpenCode Go subscription (for Tier 1–3 primary models)
- GitHub Copilot subscription (for Tier 4 fallbacks)

### Install

```bash
# Install oh-my-openagent plugin
bunx oh-my-openagent install

# Or clone this config
git clone https://github.com/williamroberttv/code-setup.git ~/.config/opencode
```

### Verify

```bash
opencode --version
bunx oh-my-openagent doctor --verbose
opencode models
```

## Jira MCP

```bash
opencode mcp auth jira
opencode /jira Create issue for login bug
```

## Rate Limit Budgeting

| Tier | Model | req/5hr | Use For |
|------|-------|---------|---------|
| Volume | deepseek-v4-flash | 31,650 | Search, quick fixes, review |
| Standard | qwen3.7-plus | 4,300 | Standard engineering |
| Elite | minimax-m3 | 3,200 | Complex agentic work |
| Elite | kimi-k2.6 | 1,150 | Hard architecture |
| Frontier | qwen3.7-max | 950 | Orchestration, Oracle |
| Copilot | claude-opus-4 | subscription | Fallback only |

> Rule: If a task will take >100 requests, route through v4-flash first. Escalate to elite for complex work. Reserve qwen3.7-max for the hardest tasks only. Copilot is fallback, not primary.

## Updating Models

If model names change, update in `oh-my-openagent.json` under `agents` and `categories` blocks. Restart OpenCode to apply.

## Troubleshooting

```bash
# Diagnostics
bunx oh-my-openagent doctor --verbose

# Refresh model capabilities cache
bunx oh-my-openagent refresh-model-capabilities

# Check config
opencode debug config
```
