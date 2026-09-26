# KnowledgeNet.ai Claude plugins

Plugins and skills that help Claude work with KnowledgeNet.ai services.

## knet-agents

Teaches Claude to use the [KNET Agents](https://agents.knet.ai) MCP server well:
LinkedIn profile, company, post and engagement data, company and people
prospecting and enrichment, email verification, and KnowledgeNet
warm-introduction paths.

| Skill | What it does |
|---|---|
| `knet-post-engagement` | Finds who liked or commented on LinkedIn posts and turns them into a qualified list |
| `knet-account-research` | Builds a brief on a company: firmographics, leadership, verified contacts |
| `knet-prospect-list` | Builds a lead list from an ideal customer profile, with verified emails |
| `knet-warm-intros` | Finds who in your network, or your team's, can introduce you (KnowledgeNet accounts) |

### Claude Code

```
/plugin marketplace add KnowledgeNetAi/claude-plugins
/plugin install knet-agents@knowledgenet
```

This installs the skills and connects the KNET Agents MCP server. Sign in when
Claude first uses it.

### Claude.ai

1. **Connect the server.** Settings → Connectors → Add custom connector →
   `https://agents.knet.ai/mcp`, then sign in.
2. **Add the skills.** Download the `.zip` files from the
   [latest release](https://github.com/KnowledgeNetAi/claude-plugins/releases/latest)
   and upload each under Settings → Capabilities → Skills. An organization
   admin can add them once for everyone.

### Pricing

Tools are priced in credits (1 credit = $0.01). Ask Claude to run
`knet_license` for the current price list or `knet_quota` for your balance.
Both are free.
