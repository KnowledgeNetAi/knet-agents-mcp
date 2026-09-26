# KNET Agents for Claude

Skills that teach Claude how to get the most out of the KNET Agents MCP server:
LinkedIn profile, company, post and engagement data, company and people prospecting and
enrichment, email verification, and KnowledgeNet warm-introduction paths. They
cover the multi-step workflows, what each step costs, and the traps that waste
credits.

| Skill | Use it to |
|---|---|
| `knet-post-engagement` | Find who liked or commented on LinkedIn posts and turn them into a qualified list |
| `knet-account-research` | Build a brief on a company: firmographics, leadership, verified contacts |
| `knet-prospect-list` | Build a lead list from an ideal customer profile with verified emails |
| `knet-warm-intros` | Find who in your network, or your team's, can introduce you (KnowledgeNet accounts) |

## 1. Connect the server

Add a custom connector pointing at `https://agents.knet.ai/mcp` and sign in
when prompted.

- **Claude.ai:** Settings → Connectors → Add custom connector.
- **Claude Code:** installing this plugin (below) adds the server for you.

## 2. Add the skills

**Claude.ai.** Download the skill `.zip` files from the [latest release](https://github.com/KnowledgeNetAi/claude-plugins/releases/latest) and upload each one under Settings →
Capabilities → Skills. An organization admin can add them once for everyone.

**Claude Code:**

```
/plugin marketplace add KnowledgeNetAi/claude-plugins
/plugin install knet-agents@knowledgenet
```

## Pricing

Every tool is priced in credits (1 credit = $0.01). Ask Claude to run
`knet_license` for the current price list, or `knet_quota` for your balance.
Both are free. A cached result costs list price the first time you're served
it and is free when you read it again.
