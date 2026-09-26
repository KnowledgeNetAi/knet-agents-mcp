---
name: knet-account-research
description: Research a company for sales or partnership work (firmographics, LinkedIn page, leadership team, decision-makers, and verified contact details) using the KNET Agents MCP tools. Use this whenever someone asks to research, profile, or "look up" a company or account, find the CEO, owner, founder, or other leaders at a company, prepare an account brief before outreach, or get contacts at a specific business, even if they only give a company name or website.
---

# Account research

This builds a sales-ready brief on one company: what it is, who runs it, and how to reach them. Each step feeds the next, so the order matters.

Prices are in credits (1 credit = $0.01). `knet_license` lists current prices, and `knet_quota` shows the balance.

## 1. Pin down the company

You need its **web domain**. If the user gave only a name, work out the domain (ask if it's ambiguous: "Acme" could be many companies).

Call `organization_enrich` with `domain` (3 credits). It returns industry, employee count, founding year, technologies, funding, social links, and `linkedin_uid` (the company's LinkedIn ID). The later steps depend on `linkedin_uid` or the LinkedIn company URL.

- **Domain form:** pass a bare domain like `acme.com`, not a URL and not a company name. A non-domain is rejected.
- **No result:** if `organization_enrich` finds nothing, try the company's other domains. If there's still nothing, get the LinkedIn company URL from the user and continue from step 2.

## 2. LinkedIn company page (optional)

`linkedin_company_get` with the LinkedIn company URL (`https://www.linkedin.com/company/<slug>`, about 0.25–0.75 credits) adds the LinkedIn description, headcount, specialties and HQ.
- Useful when the enrichment data is thin.
- Skip it when the enrichment already answers the user's question.

## 3. Find the leaders

`company_people_search` with:
- `company`: the LinkedIn company URL, or `linkedin_uid` as a string. **A company name is rejected, by design.** The underlying filter matches a company entity, not text, so a name would return zero people while still billing.
- `titles`: the roles you want, for example `["CEO", "founder", "owner", "president"]` for small businesses, or `["VP Sales", "CRO", "Head of Revenue"]` for a sales-leader search. Each title is a separate billed search (0.75 each), so pick deliberately and don't send 15 variants.

**Reading the results:**
- Use `meta.actionable_hits`, not `meta.total_hits`. Hits with `anonymized: true` appear as "LinkedIn Member" with no name or URL. LinkedIn hides people outside the searching account's network this way, and they can't be contacted.
- Zero actionable hits means the index has no match. It does **not** mean the company has no leadership. Say that, and offer to try other titles.

## 4. Contact details for the people who matter

For each person the user actually wants to contact:
1. `people_enrich` with `linkedin_url` (3 credits). This is the most precise match and returns email, phone when available, and current employment. Use the vanity form (`/in/jane-doe`). A member-ID URL (`/in/ACoAAA...`) is rejected because the enrichment engine can't match it; resolve that with `linkedin_profile_get` (provider `"lens"`) first and use the returned `url`.
2. `email_validate` (or `email_validate_batch` for several, 3 each) before anyone sends mail. Keep `deliverable: true`, flag `risky: true`, and drop `toxic`, `disposable` or `status: "invalid"`. `role_based: true` means a shared inbox (info@, sales@), not a person.

If there's no LinkedIn URL, `people_enrich` also accepts `first_name` + `last_name` + `organization_domain`. That's more precise than `organization_name`, and the domain has to be a real domain.

## 5. The brief

Use this structure:

**[Company]** · domain · industry · ~N employees · founded YYYY · HQ
**What they do:** 2–3 sentences.
**Signals:** funding, notable technologies, growth or hiring hints from the data.
**Leadership:**

| Name | Title | LinkedIn | Email (status) |
|---|---|---|---|

**Gaps:** anything you couldn't find, and why (for example, "no CFO in LinkedIn's index; may not be listed").
**Cost:** about N credits.

## Keep it cheap

- Don't run every step when the user asked one question. "Who's the CEO of X?" is steps 1 and 3 only.
- Enrich contacts only for people the user will actually reach out to.
- Repeat lookups of cached data you've already paid for are free, so re-running a brief a week later mostly costs nothing.
