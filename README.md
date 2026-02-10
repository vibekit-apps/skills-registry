# VibeKit Skills Registry

Domain skills for [VibeKit](https://vibekit.bot) AI code generation.

## What are Skills?

Skills inject domain-specific knowledge into AI code generation. When Claude builds your app, it fetches relevant skills on-demand via MCP (Model Context Protocol) — no prompt bloat.

## Available Skills (29)

| Category | Skills |
|----------|--------|
| **Frontend** | nextjs, shadcn, react-expert, react-perf, responsive-design, animations, loading-states |
| **Backend** | backend, trpc, error-handling, rate-limiting, caching, logging, websocket |
| **Database** | drizzle |
| **Security** | auth, security |
| **Quality** | typescript, testing, clean-code, accessibility, conventional-commits |
| **DevOps** | docker, api-design |
| **Domain** | stripe, solana |
| **Foundation** | design, workspace, regex |

## Using Skills

### Via VibeKit API
```bash
curl -X POST https://vibekit.bot/api/v1/task \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{"prompt": "Build a dashboard with auth", "skills": ["nextjs", "auth"]}'
```

Skills are auto-detected from your prompt, but you can specify them explicitly.

### Via MCP (for AI agents)
```typescript
// Connect to vibekit-mcp
const skills = await mcp.call("list_skills");     // Get manifest
const nextjs = await mcp.call("get_skill", { id: "nextjs" });  // Fetch content
```

## Skill Format

Each skill is a `SKILL.md` file containing:
- API patterns and correct usage
- Working code examples
- Common pitfalls to avoid
- Package versions that work together

```
skills/
├── nextjs/SKILL.md
├── shadcn/SKILL.md
├── trpc/SKILL.md
└── ...
```

## Contributing

1. Fork this repo
2. Add `skills/your-skill/SKILL.md`
3. Add entry to `skills.json`
4. Submit PR

See existing skills for format reference.

## BYOK = Free

Bring your own Anthropic API key and VibeKit is completely free. Pay Anthropic directly for the AI — we handle deployment, databases, and domains.

## License

MIT
