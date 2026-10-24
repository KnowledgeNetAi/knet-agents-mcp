---
name: knet-review-signals
description: Find local businesses whose own customers describe a problem you can fix, such as a dated or worn building, leaks and mold, or broken facilities, by reading Google Maps reviews with the KNET Agents MCP tools, then find who runs each business. Use this whenever someone wants leads from Google reviews or Google Maps, wants to find hotels, apartments, restaurants, offices or other businesses that need renovation, repairs, cleaning or another service, wants to know what customers complain about at places in an area, or wants to prospect local businesses by location, even if they never say "reviews" or "signals."
---

# Google Maps reviews → businesses with a problem you solve

Customers say in reviews what owners won't say to a salesperson: "outdated rooms", "carpet is worn", "smelled musty", "the elevator was broken again". This workflow finds businesses in an area, reads their recent reviews for the problem the user fixes, and hands back a short list of businesses with the evidence and a person to contact.

The lead is always the business, never the reviewer. Reviews come back with no reviewer names or profiles, and you should not try to identify a reviewer.

Prices are in credits (1 credit = $0.01). Call `knet_license` for current prices and `knet_quota` for the balance. Both are free.

## 1. Plan the sweep

Agree on three things with the user before spending anything.

- **Business type and area:** what to search for ("hotel", "apartment complex", "dental office", "restaurant") and where, one location per search ("Frederick, MD"). For several towns, plan one search per town.
- **Problem words:** the words a customer would use about the problem, not the words a seller would use. For a renovation firm: "outdated", "dated", "worn", "musty", "needs updating", "remodel". For a plumber: "leak", "water pressure", "no hot water". For a cleaning company: "dirty", "stained", "smelled".
- **Recency:** a start date for reviews, usually the last 12 to 24 months. Older complaints may already be fixed.

**Estimate before running.** Places cost about 0.4 credits each, plus about 0.15 per place for each optional filter. Reviews cost about 0.08 credits each, and with `keywords` you pay only for the reviews that match. A typical sweep of 20 places, then 2 problem words across those places, comes to roughly 8 credits for places and a few credits for reviews. Quote the total before running more than about 50 credits.

## 2. Find the places

Call `google_maps_places_search` with `query` and `location`.

- **Size:** `max_places` defaults to 20 (up to 50). Start with 20 and look at the results before going wider.
- **Filter in the results, not in the call, when you can.** `min_stars`, `has_website`, `skip_closed` and `categories` each add a surcharge per place. Every place already comes back with `rating`, `review_count`, `website`, `permanently_closed` and `temporarily_closed`, so you can usually drop places yourself for free.
- **Pick who to read.** Keep places with enough reviews to say something (roughly 30 or more) and drop ones with `permanently_closed: true`. A middling rating (about 3 to 4 stars) with many reviews is often where real, fixable complaints sit: the business is busy enough to afford the work and has customers who notice the problem.
- **Empty results** cost nothing and aren't cached. Retry once before telling the user there are no matches.

## 3. Read the reviews for the problem

Call `google_maps_reviews` with up to 20 `place_ids` from step 2.

- **Use `keywords`, one word or short phrase per call.** Only reviews containing it are fetched and billed, so "outdated" across 20 places might return a few dozen reviews instead of a thousand. Run a separate call for each problem word.
- **Always set `since` (`YYYY-MM-DD`) with `keywords`.** `max_reviews_per_place` caps the matches, not how far back the search goes, so without `since` a keyword search can return complaints from ten years ago.
- **Keep the default `sort` (newest) when you use `since`.** `since` works only with newest, and the tool refuses other combinations.
- **Without keywords,** `sort: "lowest"` shows the worst reviews first, which is a quick way to see what a single place's customers complain about. You pay for every review returned, so keep `max_reviews_per_place` small.
- **`origin`:** the default `all` includes reviews the place collected from other sites (about a third in our tests, often from the business's own website). Their dates are approximate. Pass `origin: "google"` if you only want Google reviews.
- **A place with no matching reviews** comes back with an empty list and costs nothing.

## 4. Read the signals, then the reviews

Each review carries `signals`, building-condition labels matched in its text:

- `dated`: outdated, dated decor or rooms, needs updating, run-down, looks tired.
- `wear`: worn, peeling, stained, cracked, chipped, torn, falling apart.
- `water`: leaks, musty, mold, mildew, water damage, damp.
- `remodel`: the reviewer mentions a remodel or renovation (in progress, needed, or done).
- `facilities`: broken showers, toilets, air conditioning, heating or elevators.

Signals are pointers, not verdicts. One dirty bathroom is housekeeping, not a renovation project, and "remodel" also matches "they just renovated, it looks great." The signals only cover building condition, so for any other offer the keywords do the filtering and you judge the text yourself. Read every flagged review and decide:

- **Strong lead:** several recent reviews, from different dates, describe the same lasting problem, and the owner's reply (`owner_response`) doesn't say it's been fixed.
- **Weak lead:** one complaint, a one-off incident, or an owner reply saying the work is done or underway.

Group by place. A business with three separate "rooms are dated" reviews in the past year is a better lead than one with a single angry review.

## 5. Find who to contact

For each strong lead, find the owner, general manager or the person who decides on the work:

- **Website:** if the place has a `website`, call `organization_enrich` with its domain. That returns the company and, when known, its LinkedIn id (`linkedin_uid`).
- **People:** call `company_people_search` with that LinkedIn company id or URL and titles such as "owner", "general manager", "property manager", "director of operations". It rejects company names, so you need the id or URL.
- **No website or no LinkedIn match:** many local businesses have neither. Give the user the place's `phone` and `maps_url`, and say a person wasn't found rather than guessing one.
- **Chains and franchises:** a branded hotel or franchise location is often run by a management company, not the brand. Look for the local property's manager rather than the brand's head office.
- Enrich only the businesses you will actually present. `people_enrich` costs about 4 credits a person, so don't run it on the whole list.

## 6. Deliver

Give the user a table with one row per business, strongest first:

| Business | Address | Rating (reviews) | What customers say | Signals | Evidence | Contact |
|---|---|---|---|---|---|---|
| … | … | 3.6 (412) | "rooms outdated", "worn carpet" (3 reviews since Mar 2025) | dated, wear | review links | name, title or phone |

Quote the customers' words briefly and link the reviews (`review_url`) so the user can check them. Never include reviewer names or attempt to identify reviewers. End with a one-line cost summary, for example: "Searched 20 hotels, read 2 keywords across 14 of them, kept 5: about 13 credits."

If the user wants a file, produce a CSV with the same columns.

## Things that go wrong

- **No `since` on a keyword search** returns old complaints that may have been fixed years ago. Always set it.
- **Reading reviews for every place found** wastes credits. Drop places with few reviews or the wrong category first.
- **Treating one review as a project.** Look for the same problem repeated over time.
- **Low balance:** if the balance can't cover the request, the tools return fewer places or reviews per place, and `meta.max_places_requested` or `meta.max_reviews_per_place_requested` says so. Tell the user the sweep was cut short.
- **Fresh data:** places are cached for 7 days and reviews for 3 days per place. A cached result costs list price the first time you're served it and is free on your repeat reads, and re-running a list pays only for places not already cached. Pass `fresh: true` only when the user needs the latest reviews.
