# Code Setup

Personal OpenCode configuration with [oh-my-openagent](https://github.com/code-yeongyu/oh-my-openagent) orchestration, SWE-bench scored model routing, and deep fallback chains.

## Architecture

**OpenCode Go models are primary. Copilot models are fallback.**

This setup uses oh-my-openagent (OmO) discipline agents with tiered model routing based on SWE-bench Pro benchmarks. Go models handle 85% of work at ~10–20x lower cost. Copilot activates only when Go hits rate limits or errors.

## Model Tiers

### Tier 1 — OpenCode Go Elite (Primary)

| Model | SWE-Pro | req/5hr | Role |
|-------|---------|---------|------|
| `opencode-go/qwen3.7-plus` | **57.6%** | 4,300 | Sisyphus, Metis, Momus, writing |
| `opencode-go/qwen3.7-max` | **60.6%** | 950 | Oracle, Code-reviewer |
| `opencode-go/minimax-m3` | **59.0%** | 3,200 | Hephaestus, unspecified-high |
| `opencode-go/glm-5.1` | **58.4%** | 880 | Prometheus, long-horizon |
| `opencode-go/kimi-k2.6` | **58.6%** | 1,150 | Complex tasks fallback |

### Tier 2 — OpenCode Go Standard

| Model | SWE-Pro | req/5hr | Role |
|-------|---------|---------|------|
| `opencode-go/deepseek-v4-pro` | **55.4%** | 3,300 | Atlas, standard engineering |
| `opencode-go/qwen3.6-plus` | — | 3,450 | Volume fallback |

### Tier 3 — OpenCode Go Volume (Never Rate-Limited)

| Model | SWE-Verified | req/5hr | Role |
|-------|--------------|---------|------|
| `opencode-go/deepseek-v4-flash` | **79%** | 31,650 | Librarian, Explore, Sisyphus-Junior |
| `opencode-go/mimo-v2.5` | — | 30,100 | Volume fallback |

### Tier 4 — Copilot Frontier (Fallback Only)

| Model | SWE-Pro | SWE-Verified | Role |
|-------|---------|--------------|------|
| `copilot/claude-opus-4` | **64.3%** | 87.6% | Fallback for Sisyphus, Oracle |
| `copilot/gpt-5.4` | **57.7%** | — | Fallback for Hephaestus, Atlas |
| `copilot/claude-sonnet-4` | — | — | Fallback for Prometheus, Metis, Momus |
| `copilot/gpt-5.3-codex` | — | — | Fallback for Atlas, coding |

## Agents

| Agent | Role | Primary | Fallback Chain |
|-------|------|---------|----------------|
| **Sisyphus** | Main orchestrator | qwen3.7-plus | minimax-m3 → qwen3.7-max → claude-opus-4 → kimi-k2.6 |
| **Hephaestus** | Deep worker | minimax-m3 | gpt-5.4 → gpt-5.3-codex → deepseek-v4-pro → deepseek-v4-flash |
| **Oracle** | Architecture / RCA | qwen3.7-max | claude-opus-4 → claude-sonnet-4 → glm-5.1 → kimi-k2.6 |
| **Prometheus** | Planner | glm-5.1 | claude-sonnet-4 → gpt-5.4 → qwen3.7-max → deepseek-v4-pro |
| **Librarian** | Search | deepseek-v4-flash | mimo-v2.5 → claude-sonnet-4 |
| **Explore** | Fast grep | deepseek-v4-flash | mimo-v2.5 |
| **Metis** | Plan consultant | qwen3.7-plus | claude-sonnet-4 → qwen3.6-plus → deepseek-v4-pro |
| **Momus** | Review | qwen3.7-plus | claude-sonnet-4 → qwen3.6-plus → kimi-k2.6 |
| **Atlas** | Ops / tests | deepseek-v4-pro | gpt-5.3-codex → minimax-m3 → deepseek-v4-flash |
| **Code-reviewer** | Quality | qwen3.7-max | claude-opus-4 → claude-sonnet-4 → kimi-k2.6 → deepseek-v4-pro |
| **Sisyphus-Junior** | Quick tasks | deepseek-v4-flash | mimo-v2.5 |

## Installation

### Prerequisites

- [OpenCode](https://opencode.ai/) 1.0.133+
- [OpenCode Go](https://opencode.ai/go) subscription ($10/month, 19 models)
- [GitHub Copilot](https://github.com/features/copilot) subscription (for fallback only)
- Git, Node.js or Bun
- macOS, Linux, or WSL

### Step 1: Install OpenCode

```bash
curl -fsSL https://opencode.ai/install | bash
opencode --version  # Should be >= 1.0.133
```

### Step 2: Install Oh My OpenAgent

**Recommended (Bun — faster, OmO-optimized):**

```bash
# Interactive install
bunx oh-my-openagent install

# Non-interactive (for LLM agents, CI)
bunx oh-my-openagent install --no-tui \
  --opencode-go=yes \
  --opencode-zen=yes \
  --claude=no \
  --openai=no \
  --gemini=no \
  --copilot=yes
```

**Alternative (npx — universal, works with any Node setup):**

```bash
# Interactive install
npx oh-my-openagent install

# Non-interactive
npx oh-my-openagent install --no-tui \
  --opencode-go=yes \
  --opencode-zen=yes \
  --claude=no \
  --openai=no \
  --gemini=no \
  --copilot=yes
```

> **Why Bun?** Oh-my-openagent is Bun-first (`bun.lock`, `bunfig.toml` in repo). `bunx` is ~3x faster than `npx` for package execution. Use `npx` if you don't have Bun installed.

### Step 3: Clone This Config

```bash
# Clone to your OpenCode config directory
git clone https://github.com/williamroberttv/code-setup.git ~/.config/opencode

# Or copy manually
mkdir -p ~/.config/opencode
rsync -a opencode/ ~/.config/opencode/
```

### Step 4: Verify Setup

```bash
# Check OpenCode version
opencode --version

# Run OmO diagnostics
bunx oh-my-openagent doctor --verbose

# List available models
opencode models

# Authenticate providers
opencode auth login
```

## Token Economy (RTK)

This setup includes [RTK (Rust Token Killer)](https://github.com/rtk-ai/rtk) — a CLI proxy that reduces LLM token consumption by **60-90%** on common dev commands.

### What RTK Does

| Command | Without RTK | With RTK | Savings |
|---------|-------------|----------|---------|
| `git status` | ~2,000 tokens | ~400 tokens | **80%** |
| `cat file` | ~40,000 tokens | ~12,000 tokens | **70%** |
| `ls` | ~800 tokens | ~150 tokens | **80%** |
| `cargo test` | ~25,000 tokens | ~2,500 tokens | **90%** |

### Install RTK

**With Homebrew (recommended):**
```bash
brew install rtk
```

**With curl (Linux/macOS):**
```bash
curl -fsSL https://raw.githubusercontent.com/rtk-ai/rtk/refs/heads/master/install.sh | sh
# Add to PATH if needed:
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc  # or ~/.zshrc
```

**With Cargo:**
```bash
cargo install --git https://github.com/rtk-ai/rtk
```

### Verify RTK

```bash
rtk --version        # Should show "rtk 0.x.x"
rtk gain             # Show token savings stats
rtk rewrite "git status"  # Test command rewrite
```

### RTK + OpenCode Plugin

The `opencode-rtk` plugin is already configured in `opencode.json`. It automatically rewrites bash commands to use RTK equivalents.

### Troubleshooting

**If RTK causes loops or errors:**
1. Check RTK is in PATH: `which rtk`
2. Disable plugin temporarily in `opencode.json`: remove `"opencode-rtk"` from plugins array
3. Use RTK manually instead of auto-rewrite: `rtk git status`
4. Check RTK version: `rtk --version` (need >= 0.27.0)

**If commands fail with RTK:**
- The plugin falls back to original command automatically
- Or use absolute path: `/usr/bin/git status` instead of `git status`

## Commands

| Command | Agent | Description | Primary Model |
|---------|-------|-------------|---------------|
| `/ship` | sisyphus | End-to-end implementation | qwen3.7-plus |
| `/ultrawork` | sisyphus | Full team activation | qwen3.7-plus |
| `/plan` | prometheus | Strategic planning | glm-5.1 |
| `/code` | hephaestus | Fast coding pass | minimax-m3 |
| `/ops` | atlas | Tests, git, PR | deepseek-v4-pro |
| `/rca` | oracle | Root cause analysis | qwen3.7-max |
| `/review` | code-reviewer | Final review | qwen3.7-max |
| `/judge` | momus | Second opinion | qwen3.7-plus |
| `/jira` | jira | Jira MCP workflow | qwen3.7-plus |
| `/caveman` | — | Toggle caveman mode | — |

## Fallback Behavior

```
Error / Rate Limit → Blacklist provider 60s → Next fallback model
Go primary fails → Copilot fallback → Go volume fallback
Max 3 attempts per request → Notify on every switch
```

## Rate Limit Budgeting

| Tier | Model | req/5hr | Use For |
|------|-------|---------|---------|
| Volume | deepseek-v4-flash | 31,650 | Search, quick fixes, review |
| Standard | qwen3.7-plus | 4,300 | Orchestration, standard engineering |
| Elite | minimax-m3 | 3,200 | Complex agentic work |
| Elite | kimi-k2.6 | 1,150 | Hard architecture |
| Frontier | qwen3.7-max | 950 | Oracle, Code-reviewer |
| Copilot | claude-opus-4 | subscription | **Fallback only** |

> **Rule:** If a task will take >100 requests, route through v4-flash first. Escalate to elite for complex work. Reserve qwen3.7-max for the hardest tasks only. Copilot is fallback, not primary.

## Jira MCP

```bash
# Authenticate
opencode mcp auth jira

# Use Jira command
opencode /jira Create issue for login bug
```

## Project Structure

```
opencode/
├── opencode.json              # Main OpenCode config
├── oh-my-openagent.json       # OmO agent/model mapping + fallbacks
├── AGENTS.md                  # Rules injection for all agents
├── README.md                  # This file
├── WORKFLOW_DIAGRAM.md        # Mermaid workflow diagrams
└── skills/
    ├── caveman/               # Ultra-compressed communication
    └── grill-me/              # Stress-test planning skill
```

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

# List agents
opencode agent list
```

## References

- [Oh My OpenAgent GitHub](https://github.com/code-yeongyu/oh-my-openagent)
- [Oh My OpenAgent Docs](https://ohmyopenagent.com/en/docs)
- [OpenCode Go](https://opencode.ai/go)
- [SWE-bench Leaderboard](https://www.swebench.com/)
- [Config Schema](https://raw.githubusercontent.com/code-yeongyu/oh-my-openagent/dev/assets/oh-my-opencode.schema.json)
