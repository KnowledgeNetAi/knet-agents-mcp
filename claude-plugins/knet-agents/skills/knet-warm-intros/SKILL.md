---
name: knet-warm-intros
description: Find warm introduction paths (who you or your colleagues know who can introduce you) to a target company or person, using KnowledgeNet relationship intelligence through the KNET Agents MCP tools. Use this whenever someone asks "who do we know at...", "how can I get an intro to...", "find a warm path to...", "who on my team knows...", or wants to reach a prospect through a mutual connection rather than a cold email, even if they don't mention KnowledgeNet.
---

# Warm introductions

KnowledgeNet maps who knows whom across your organization's relationship data. These tools find the best-connected route from you, or a teammate, to a target. They need a KnowledgeNet account signed in through OAuth. If the tools answer that no identity or organization is available, the caller doesn't have one, so say so plainly rather than retrying.

Every `knet_*` tool is free, including `knet_ri_paths`, `knet_ri_stats`, `knet_interactions_nodes`, `knet_org_users` and the organization tools. Only the relationship tools (`knet_ri_paths`, `knet_ri_stats`, `knet_interactions_nodes`) are rate limited, at 30 per minute and 300 per hour, so don't loop over hundreds of targets at once. The organization and roster tools have no rate limit.

**Try the free search before paying for anything.** A `target_domain` search finds paths to everyone you're connected with at the company, often including the exact person the user named. Only spend credits identifying a specific person (`people_search`, `company_people_search`) when the domain search doesn't surface them.

## 1. Pick the target form

Call `knet_ri_paths` with **exactly one** of:
- `target_domain`: a company (`acme.com`). Finds paths to anyone you're connected with there.
- `target_email`: a specific person.
- `target_linkedin_url`: a specific person, by LinkedIn profile URL or just the slug. Company URLs are rejected; use `target_domain` for a company.

If the user gave only a company name, work out the domain first (`organization_enrich` can confirm it).

When the user wants a specific role ("their CFO"), start with `target_domain` and look through the paths for that role. Identify the person by name only if the domain paths don't include them, then re-run with `target_linkedin_url` or `target_email`.

## 2. Read the paths

The response ranks `scoredRoutes`, each with a human-readable explanation of the chain (you → colleague → target). Defaults: up to 10 paths, at most 3 hops, shortest preferred. You can raise `max_paths` or `max_length` if nothing short turns up.

When you present a path, name the connector and say *why* they're a good route (recent meetings, frequent email), because that's what makes the intro ask credible.

## 3. When you have no path yourself, check your team

Paths run from *your* network by default. To see who on your team can open the door:
1. `knet_org_users` lists the active members of your KnowledgeNet organization.
2. `knet_ri_paths` with `from_email` (or `from_user_id`) set to a teammate, for the teammates most likely to know the target (same region, industry, or function).

For a quick "do we have anything at all?", `knet_ri_stats` gives relationship strength counts for a domain or email without full paths.

## 4. Organization context

Every call runs in your currently active KnowledgeNet organization. If you belong to several and the results look wrong, `knet_orgs_list` shows them and `knet_orgs_set_active` switches. Say which organization you searched in.

## 5. Deliver

For each target:

**[Target]**. Best path: You → **Jane Smith** (met 4 times this quarter) → **Target**.
Other options: … (up to 3).
**Suggested ask:** a one-line intro request the user could send the connector.

If there's no path anywhere, say so clearly and suggest the next best step: a colleague search with `from_email`, or cold outreach with `knet-account-research` for contact details.
