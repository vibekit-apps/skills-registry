# VibeKit Skills Registry

Audited registry of domain skills for [VibeKit](https://vibekit.bot).

## What are Skills?

Skills inject domain-specific knowledge into AI code generation. When you build an app with VibeKit and specify a skill, the AI gets context about that domain's best practices, APIs, and patterns.

## Using Skills

```bash
curl -X POST https://vibekit.bot/api/v1/task \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{"prompt": "Build a Solana wallet app", "skills": ["solana"]}'
```

## Available Skills

| ID | Name | Description | Verified |
|----|------|-------------|----------|
| solana | Solana | Wallet connection, dApp development, Anchor programs | ✅ |

## Contributing a Skill

1. **Fork this repo**
2. **Add your skill to `skills.json`:**
   ```json
   {
     "id": "your-skill",
     "name": "Your Skill",
     "description": "What it does",
     "url": "https://your-domain.com/SKILL.md",
     "author": "Your Name",
     "verified": false,
     "tags": ["relevant", "tags"],
     "addedAt": "YYYY-MM-DD"
   }
   ```
3. **Submit a PR**

### SKILL.md Format

Your SKILL.md should contain:
- API patterns and correct usage
- Code examples (working, tested)
- Common pitfalls to avoid
- Package versions that work together

See [Solana's SKILL.md](https://github.com/anza-xyz/solana-pay/blob/master/SKILL.md) for reference.

## Verification

Skills start as `"verified": false`. After review, we'll verify skills that:
- Have accurate, working code examples
- Follow best practices for the domain
- Are actively maintained

## License

MIT
