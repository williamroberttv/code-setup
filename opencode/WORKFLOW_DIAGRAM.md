# Workflow Diagram

## Primary Orchestration Flow

```mermaid
graph TD
    A[User Request] --> B{Auto Agent}
    B -->|Simple| C[Direct Response]
    B -->|Non-Trivial| D[workflow-route]
    D --> E{Route Decision}
    
    E -->|self| C
    E -->|explore| F[Explore]
    F --> E
    E -->|qwen-coder| G[Qwen Coder]
    E -->|gpt-planner| H[GPT Planner]
    E -->|glm-analyzer| I[GLM Analyzer]
    E -->|glm-reviewer| J[GLM Reviewer]
    E -->|gpt-critic| K[GPT Critic]
    E -->|kimi-context| L[Kimi Context]
    E -->|qwen-operator| M[Qwen Operator]
    E -->|minimax-writer| N[MiniMax Writer]
    E -->|general| O[General Agent]
    
    G -->|code changes| J
    H -->|plan| G
    I --> G
    L -->|synthesis| H
    L -->|synthesis| I
    L -->|synthesis| J
    
    G -->|ops| M
    M -->|review| J
    J -->|escalation| K
    J -->|ok| P[Done]
    
    style A fill:#e1e8ed
    style P fill:#90EE90
    style K fill:#FFB6C1
```

## Decision Tree

```mermaid
graph TD
    A[Start] --> B{Scope Known?}
    B -->|No| C[Explore First]
    C --> D{File Count?}
    D -->|>= 4| E[GPT Planner]
    D -->|< 4| F[Qwen Coder]
    B -->|Yes| G{Implementation?}
    
    G -->|Large/Multi| E
    G -->|Contained| F
    G -->|Tests/Git| M[Qwen Operator]
    G -->|Review| J[GLM Reviewer]
    
    E -->|plan| F
    F --> M
    M --> J
    
    J -->|High Risk| K[GPT Critic]
    J -->|OK| P[Done]
    K --> P
```

## Agent Roles

| Agent | Model | Purpose | Tools |
|-------|-------|--------|-------|
| `auto` | Kimi 2.6 | Orchestrator | read, search, context, route |
| `qwen-coder` | MiMo v2.5 Pro | Code implementation | write, edit, bash |
| `qwen-operator` | DeepSeek v4 Flash | Ops (tests, git, PR) | bash, git |
| `glm-analyzer` | GLM-5 | RCA, tradeoffs | read, grep |
| `glm-reviewer` | GLM-5 | Default review | read, grep |
| `gpt-planner` | GPT-5.4 | Large planning | read, grep |
| `gpt-critic` | GPT-5.4 | Escalation review | read, grep |
| `kimi-context` | Kimi 2.6 | Context compression | read |
| `minimax-writer` | MiniMax M2.7 | Creative writing | read |

## Routing Patterns

### Search / Discovery
→ `explore`

### Implementation (scope unknown)
→ `explore` → measure → rerun with scopeKnown=true

### Implementation (large, multi-file)
→ `gpt-planner` → `qwen-coder`

### Implementation (contained, 1-3 files)
→ `qwen-coder`

### RCA / Debugging / Tradeoffs
→ `glm-analyzer`

### Tests / Git / PR
→ `qwen-operator`

### Review Only
→ `glm-reviewer` → escalate to `gpt-critic` if high-stakes

### Large Context
→ `kimi-context` → route to appropriate agent

### Naming / Writing / Brainstorming
→ `minimax-writer`

### Independent Parallel Work
→ `general` → split and integrate

## Common Flows

### Code Change + Repo Ops
```
qwen-coder → qwen-operator → glm-reviewer
```

### Large Implementation
```
explore → gpt-planner → qwen-coder → qwen-operator → glm-reviewer
```

### Large Context Bug
```
kimi-context → glm-analyzer → qwen-coder → qwen-operator → glm-reviewer
```

### High-Stakes Review
```
glm-reviewer → gpt-critic (escalation)
```

## Command Aliases

| Command | Agent | Model |
|---------|-------|-------|
| `/ship` | auto | Kimi 2.6 |
| `/code` | qwen-coder | MiMo Pro |
| `/ops` | qwen-operator | DeepSeek Flash |
| `/rca` | glm-analyzer | GLM-5 |
| `/review` | glm-reviewer | GLM-5 |
| `/ctx` | kimi-context | Kimi 2.6 |
| `/plan` | gpt-planner | GPT-5.4 |
| `/draft` | minimax-writer | MiniMax M2.7 |
| `/judge` | gpt-critic | GPT-5.4 |