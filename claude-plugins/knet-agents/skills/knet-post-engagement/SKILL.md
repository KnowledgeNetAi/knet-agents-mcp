---
name: knet-post-engagement
description: Find out who liked, reacted to, or commented on LinkedIn posts, and turn that audience into a qualified prospect list, using the KNET Agents MCP tools. Use this whenever someone wants to know who is engaging with a person's or company's LinkedIn content, mine a competitor's or influencer's post audience, find warm leads from post engagement, see who commented on a specific post, or follow up with people who interacted with their own posts, even if they never say "reactions" or "engagement."
---

# LinkedIn post engagement → prospect list

People who react to or comment on a post have raised their hand on that topic. This workflow finds them, keeps the ones who fit, and enriches only those, so credits go to people worth contacting.

Prices below are in credits (1 credit = $0.01). Call `knet_license` for current prices and `knet_quota` for the balance. Both are free.

## 1. Get the posts

- **If the user gave a post URL**, skip to step 2. Any form works: `/posts/...`, `/feed/update/urn:li:activity:<id>`, or a bare `urn:li:activity:<id>`.
- **If they gave a person**, call `linkedin_posts_get` with the profile URL (about 0.46 per page, newest posts first). Each post has `url`, `posted_at`, `text` and `engagement` counts.
- **Choose which posts to mine.** The engagement counts tell you what each post will cost before you spend anything. Prefer recent posts with real engagement, and ask the user if the choice isn't obvious (for example, "the last 5 posts" vs. "the one about the product launch").

## 2. Pull reactions and comments

For each chosen post, call `linkedin_post_reactions` and/or `linkedin_post_comments` with the post URL.

- **Page size and cost:** up to 100 people per page, about 0.46 per page.
- **Paging:** keep going while `meta.has_more` is true, and stop there. A page past the end still bills.
- **Comments vs. reactions:** comments are stronger signals than likes, and the text shows what the person cares about. If the user wants the most engaged people, comments alone are often enough.
- **Replies** come nested inside `replies`.
- **The author:** `is_post_author: true` marks the post's own author. Leave them out of the audience.

**Estimate first when the job is large.** Use this formula and quote one total:
- (posts × pages per post) × 0.35
- \+ about 0.74 per shortlisted reactor profile
- \+ 8 per person who gets an email (4 to enrich, 4 to validate)

Tell the user the estimate before running anything over roughly 50 credits.

## 3. Merge and dedupe

- **Combine** everyone across posts and dedupe by `actor.id`.
- **For each person, record** which posts they engaged with, whether they reacted or commented, and a short excerpt of any comment. Someone who engaged with several posts is a stronger lead than a one-time liker.
- **Companies:** `actor.kind: "company"` entries are company pages, not people. Drop them unless the user wants companies.

## 4. Filter before you enrich

`actor.headline` already carries title and company for almost everyone, and it's free.
- Apply the user's criteria (titles, seniority, industry, company type) to the headline first.
- Enrich only the people who pass.
- If the user didn't give criteria, ask who they want before enriching hundreds of profiles.

## 5. Enrich the shortlist

Reactor URLs almost always look like `https://www.linkedin.com/in/ACoAAA...`, a member ID rather than a vanity slug. That changes the order of steps:

- **Profile first, for reactors.** Call `linkedin_profile_get` with `provider: "lens"` (about 0.74), or `linkedin_profile_get_batch` for up to 10 at a time, also with `provider: "lens"`, since the batch tool's default provider is a different one. The Lens engine resolves member-ID URLs, and the returned profile's `url` is the person's real vanity URL. You get experience, location and education along the way.
- **Email only after that.** `people_enrich` (4 each) cannot match a member-ID URL: the enrichment engine can't match one, so the tool rejects `/in/ACoAAA...` with a 400 rather than bill you for nothing. Pass the vanity `url` from the profile step.
- **Commenters** usually come with a normal vanity URL, so they can go straight to `people_enrich` if you don't need the full profile.
- **Email is for the final list only.** Every enrichment costs, so run it just for the people the user will contact.
- **Before anyone sends mail:** run `email_validate_batch` on the emails you found (3 each, up to 200 per call). Keep addresses with `deliverable: true`. Treat `risky: true` (catch-all domains, transient failures) as a judgment call, and drop anything with `toxic` or `disposable` set, or `status: "invalid"`.

## 6. Deliver

Give the user a table with one row per person:

| Name | Headline / title | Company | Engagement | What they said | LinkedIn |
|---|---|---|---|---|---|
| … | … | … | Commented on 2 posts, liked 1 | "…short excerpt…" | URL |

Sort by engagement strength: comments before likes, several posts before one. End with a one-line cost summary, for example: "Pulled 3 posts, 412 engagers, enriched 18: about 29 credits."

## Things that go wrong

- **Paging past the end** wastes credits. Always stop on `meta.has_more: false`.
- **Enriching everyone** is the expensive mistake. The headline filter usually cuts the list by 80% or more.
- **A post with 0 engagers** usually means the post is very new or the URL points at a repost. Check `post.created_at` in the response, and use the original post's URL.
- **Fresh data:** results are cached (about 1 hour for posts under 2 days old, up to 7 days for older ones). A cached page costs list price the first time and is free on your repeat reads. Pass `fresh: true` only when the user needs the latest count.
