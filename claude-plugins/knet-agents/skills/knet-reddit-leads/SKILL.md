---
name: knet-reddit-leads
description: Find people on Reddit who are asking for help you can sell, such as someone looking for a developer, naming a budget, or wanting an app, tool or website built, using the KNET Agents MCP tools. Use this whenever someone wants to source leads or buying signals from Reddit, monitor a subreddit for requests, find posts where people describe a problem or need in public, or test whether a niche has demand on Reddit, even if they never say "leads" or "buying signals."
---

# Reddit posts → a short list of real requests

People describe what they need on Reddit more openly than they would to a cold message. This workflow finds those posts, keeps the ones with real intent, and hands the user a list they can review and act on.

Prices are in credits (1 credit = $0.01). Call `knet_license` for current prices and `knet_quota` for the balance. Both are free.

## 1. Plan the sweep

Turn the user's offer into search phrases and places to look before spending anything.

- **Phrases:** write them the way a buyer would, not the way a seller would. For a development agency: "looking for a developer", "need someone to build", "need an app built", "hire a freelancer", "no technical cofounder".
- **Subreddits:** ask the user which communities their buyers use, or start with the obvious ones for the niche. Hiring subreddits (for example r/forhire, r/Entrepreneur, r/SaaS, r/startups) mix buyers with people offering services, so expect noise there.
- **Window:** `time_range` of `week` or `month` for a first sweep. Older requests are usually already filled.
- **Estimate:** each post returned costs about 0.26 credits, so 25 posts is about 6.5 and 100 posts about 26. Several phrases across several subreddits add up quickly, so quote the total before running more than about 50 credits.

## 2. Search

Call `reddit_search` once per phrase or subreddit. It takes one query and one subreddit per call.

- **A phrase across all of Reddit:** pass `query` with `strict: true`. Without `strict`, Reddit's own search returns many unrelated posts. Results are ranked by relevance.
- **Everything new in one subreddit:** pass `subreddit` alone. Results are newest first.
- **A phrase inside one subreddit:** pass both.
- **Size:** `max_posts` defaults to 25 (up to 100). Start small, look at the quality, then go wider. If your balance can't cover the request, the tool returns fewer posts and `meta.max_posts_requested` says so.
- **Exact dates:** `posted_after` / `posted_before` take `YYYY-MM-DD`.

## 3. Read the signals, then the posts

Every post carries `signals`, labels matched in its text:

- `budget`: an amount next to paying or budget wording, "willing to pay", or a "[Paid]" tag.
- `hiring`: looking for a developer, freelancer, agency or contractor.
- `build_request`: asking someone to build an app, site, tool or MVP.
- `no_tech_team`: the author says they are non-technical or have no developers.

Signals are pointers, not verdicts. "Hiring" also matches someone advertising their own services ("[For Hire] web developer"), and "budget" can match a band or a personal trainer posting rates. Read each flagged post and keep only genuine requests that fit the user's offer. A post with two or more signals, real engagement (`score`, `num_comments`) and a recent `created_at` is usually the strongest lead.

## 4. Deliver

Give the user a table with one row per kept post:

| Posted | Subreddit | Title | What they need | Signals | Post | Author |
|---|---|---|---|---|---|---|
| … | r/… | … | one-line summary in your words | budget, hiring | url | author_url |

Sort the strongest first. End with a one-line cost summary, for example: "Ran 4 searches, 85 posts, kept 9: about 22 credits."

If the user wants a file, produce a CSV with the same columns.

## Things that go wrong

- **An empty result** costs nothing, but it can mean Reddit briefly refused the request rather than that no posts exist. Retry once before telling the user there is nothing there.
- **Reddit-wide searches without `strict`** are mostly noise. Use `strict: true`, or search inside a subreddit.
- **Reddit authors are pseudonymous.** There is no reliable link from a Reddit username to a LinkedIn profile or an email, so don't guess one or enrich a lookalike. The way to reach the author is to reply on Reddit or message them there.
- **Respect each community.** Many subreddits ban promotional replies and cold messages. Tell the user to read the subreddit's rules before contacting anyone, and to reply with something useful rather than a pitch.
- **Comments are not included**, only posts. Open a promising post's `url` to read the discussion.
- **Fresh data:** results are cached for a day. A cached result costs list price the first time you're served it and is free on your repeat reads. Pass `fresh: true` only when the user needs the latest posts.
