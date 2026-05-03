---
description: Single visible default agent. Use for normal development work and automatically route to specialized subagents when useful.
mode: primary
model: opencode-go/kimi-k2.6
temperature: 0.2
color: primary
permission:
  edit: deny
  bash: deny
  task:
    "*": deny
    explore: allow
    general: allow
    gpt-planner: allow
    glm-analyzer: allow
    glm-reviewer: allow
    gpt-critic: allow
    kimi-context: allow
    qwen-coder: allow
    qwen-operator: allow
    revenuecat-agent: deny
    minimax-writer: allow
