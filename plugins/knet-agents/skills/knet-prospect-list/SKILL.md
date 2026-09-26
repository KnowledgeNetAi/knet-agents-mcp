---
name: knet-prospect-list
description: Build a targeted B2B prospect or lead list (companies matching an ideal customer profile, then the right people at them with verified emails) using the KNET Agents MCP tools. Use this whenever someone wants a lead list, target account list, prospect list, list of companies or decision-makers matching criteria (industry, size, location, technology, title, seniority), or wants to "find me 50 CFOs at..." or "companies like X".
---

# Prospect list building

This turns an ideal-customer description into a list of companies and verified contacts. The main risk is a search that *looks* filtered but isn't, so the workflow checks for that before spending on enrichment.

Prices are in credits (1 credit = $0.01): company and people search 1.2 per page, `people_enrich` 3, email validation 3. `knet_license` has current prices, and `knet_quota` shows the balance.

## 1. Turn the request into filters

Pin down what the user means:
- **Company:** industry, headcount range, revenue, location, technologies.
- **Person:** titles, seniority, department.
- **How many** they want.

Ask about anything that changes the search. Don't ask about details you can default.

## 2. Search, and check that the filters actually applied

Use `company_search` for accounts, or `people_search` for people. Pass filters in `search_params`.

**The search never rejects a bad filter.**
- A wrong enum value (for example, a seniority spelled differently from the documented list) returns zero results that look like a real no-match.
- A wrong format (for example, `"1-10"` instead of `"1,10"` for employee ranges) is dropped silently, so you get the unfiltered set and think it was filtered.
- Both still bill.

To guard against this:
- **Use only the values and formats documented** in the tool's `search_params` description.
- **Run page 1 with `per_page` 25 first** and check `meta.total_entries`. A million-plus total for a narrow ideal customer profile means a filter was ignored. Zero for a broad one means an enum is wrong. Fix it and re-run before paging further.
- **Check both arrays.** Results come back in `results` (the Atlas database) and `saved` (records already in the workspace). A match is in exactly one of them, so look at `meta.saved_count` before concluding a search found nothing.

Then page (`page` 2, 3, … up to `meta.total_pages`, at 1.2 per page) until you have the number the user asked for.

## 3. People at the target companies

If you searched companies first, find people in one of two ways:
- **People search:** `people_search` filtered to those organizations plus titles or seniorities, or
- **LinkedIn leadership:** `company_people_search` with each company's LinkedIn URL or `linkedin_uid` (from `organization_enrich`) and a short list of `titles`. It costs 0.75 per title per company, so keep titles few. Company names are rejected. Read `meta.actionable_hits`; anonymized hits can't be contacted.

## 4. Enrich and verify only the final list

1. `people_enrich` for each person you'll keep (3 each). Pass `linkedin_url` when you have it (most precise), else email, else `first_name` + `last_name` + `organization_domain`.
2. `email_validate_batch` on every email (up to 200 per call, 3 each; duplicates are charged once). Keep `deliverable: true`. Mark `risky: true`, and drop `toxic`, `disposable` and `status: "invalid"`. If the balance runs out partway, the remaining addresses come back with status 402 and are not billed. Tell the user and resend after they top up.

**Estimate before enriching.** People × 6 credits (enrich plus validate) adds up quickly. Confirm with the user before enriching more than about 25 people.

## 5. Deliver

Give a table (or CSV if asked) with these columns: Company · Domain · Industry · Employees · Name · Title · LinkedIn · Email · Email status.

Add three short notes:
- **Match count:** total matches (`meta.total_entries`) vs. how many you returned, so the user knows how much more is available.
- **Filters used:** exactly as applied, so the search can be re-run or adjusted.
- **Cost:** about N credits.

## Pitfalls

- **Zero results** are usually a filter problem or a coverage gap, not proof the market doesn't exist. Loosen one filter at a time.
- **Paging past `meta.total_pages`** still bills.
- **LinkedIn search tools:** `linkedin_search_*` and `linkedin_sales_search_*` need an `identity_id` provisioned for your account. Without one, use `company_search` and `people_search` above.
