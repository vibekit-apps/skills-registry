---
name: antfarm
description: Multi-agent workflows for OpenClaw. Use when building complex features that benefit from plan → implement → verify → test → review cycles.
---

# Antfarm + VibeKit

Use [Antfarm](https://github.com/snarktank/antfarm) for sophisticated multi-agent development, VibeKit for deployment.

## When to Use

- Complex features needing multiple implementation steps
- Security audits with automated fixes
- Bug fixes requiring investigation and regression tests
- Any task benefiting from verification gates

## Setup

```bash
# Install antfarm (requires OpenClaw locally)
npx antfarm install

# Run a feature workflow
antfarm workflow run feature-dev "Add user authentication with OAuth"

# Check status
antfarm workflow status "OAuth"
```

## Workflows

| Workflow | Agents | Use Case |
|----------|--------|----------|
| `feature-dev` | 7 | Feature requests → tested PR |
| `security-audit` | 7 | Vulnerability scan → fix PR |
| `bug-fix` | 6 | Bug report → fix with regression test |

## With VibeKit

**Option 1: Antfarm develops, VibeKit deploys**
```bash
# Antfarm creates the PR
antfarm workflow run feature-dev "Add Stripe checkout"

# VibeKit deploys when PR merges
curl -X POST https://vibekit.bot/api/v1/task \
  -H "Authorization: Bearer $VIBEKIT_KEY" \
  -d '{"prompt": "Deploy latest from main", "repo": "user/app"}'
```

**Option 2: VibeKit for quick iterations, Antfarm for complex features**
- Simple UI changes → VibeKit (instant)
- New auth system → Antfarm (thorough)

## How It Works

```
plan → setup → implement → verify → test → PR → review
  │       │         │         │        │      │       │
  └───────┴─────────┴─────────┴────────┴──────┴───────┘
              Each agent = fresh context
              Failed steps retry automatically
              Nothing ships without review
```

## Requirements

- Node.js >= 22
- OpenClaw running locally
- `gh` CLI for PR creation

## Resources

- [Antfarm Repo](https://github.com/snarktank/antfarm)
- [Creating Custom Workflows](https://github.com/snarktank/antfarm/blob/main/docs/creating-workflows.md)
- [OpenClaw Docs](https://docs.openclaw.ai)
