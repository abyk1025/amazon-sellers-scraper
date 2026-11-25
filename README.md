# Amazon Best Sellers Scraper: How ScraperAPI's Amazon Credits Work, Which Tier Fits Your Volume, and Where the Structured Data Comes From

ScraperAPI bills an Amazon request at 5 credits, so the credit figure on any plan is one fifth of the Best Sellers pages you can actually pull with it. The free tier's 1,000 credits cover about 200 Amazon page loads, and the 5,000,000-credit Scaling tier at $475/month works out to roughly $0.48 per 1,000 Amazon pages. Below is the credit arithmetic for realistic crawl volumes, the plan table, the failures that quietly truncate bulk runs, and the parsing work the API leaves on your side.

> **Quick pick:** start on the free tier while you build. 1,000 credits is 200 Amazon pages, enough for several full sweeps of Amazon's 37 top departments, so you can validate your field mapping before paying anything. Move up only after you've measured a month of real page counts. 👉 [Start on the free plan and check the live credit costs](https://www.scraperapi.com/pricing?fp_ref=coupons)

## What one Amazon Best Sellers page actually costs

ScraperAPI prices by domain, and Amazon sits in the e-commerce bucket at **5 credits per request** instead of the standard 1. That multiplier is the number to carry into every estimate you build.

Extra parameters stack on top of the base cost:

| Request type | Credits per request |
| --- | --- |
| Standard page | 1 |
| Amazon, Walmart, eBay | 5 |
| JS rendering (`render=true`) | +10 |
| Premium proxy (`premium=true`) | +10 |
| Bot-protection bypass (Cloudflare, DataDome, PerimeterX, Akamai) | +10 |
| Google or Bing | 25 |
| LinkedIn | 30 |
| Country code, session, device type, output format, autoparse | 0 |

An Amazon page with rendering enabled costs 15 credits, three times the plain request. At the Scaling tier's monthly rate ($475 for 5,000,000 credits, or $0.000095 per credit) that is **$0.48 per 1,000 plain Amazon pages and $1.43 per 1,000 rendered ones**. Billed annually at $427.50/month, the same pages come to about $0.43 and $1.28.

Three levers let you check this before committing a month of spend: the `sa-credit-cost` response header on every request, the `/account/urlcost` endpoint (which accepts the same parameters as a real scrape), and `max_cost`, which caps what a single request is allowed to spend. If a page costs more than you expected, you find out from the header rather than from an empty balance.

👉 [See how per-request credit costs map onto each plan](https://www.scraperapi.com/pricing?fp_ref=coupons)

## ScraperAPI plans at a glance

| Plan | Credits per month | Price | Concurrent threads | Start |
| --- | --- | --- | --- | --- |
| Free | 1,000 | $0 | 5 | [Start free](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Hobby | — | — | — | [Check Hobby on the live pricing page](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Startup | — | — | — | [Check Startup's current allowance](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Business | — | — | — | [Check Business pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Scaling | 5,000,000 | $475/mo, or $427.50/mo billed annually ($5,130/yr) | 200 | [See Scaling's full terms](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Professional | — | — | — | [Check Professional pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | — | — | — | [Check Advanced pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | — | — | — | [Check Enterprise terms](https://www.scraperapi.com/pricing?fp_ref=coupons) |

ScraperAPI lists these eight tiers in ascending order: credits, price and thread count all step up as you move down the list, and the per-credit rate improves with them. The Free and Scaling rows carry figures published on its pricing page. The tiers in between are re-priced often enough that I'd rather send you to the live page than print a number that may have moved since. One structural detail worth knowing regardless of which tier you land on: Advanced is the highest self-serve plan, and Enterprise is the one above it.

## How many pages a real Best Sellers sweep needs

Amazon's Best Sellers tree runs to 37 top-level departments, the number both Apify Best Sellers actors default to walking. One request per department page covers the top 50 items in each, which is 37 requests or **185 credits per full sweep of the US store**.

Run that daily and you are at 1,110 requests and 5,550 credits a month. Turn rendering on and it becomes 16,650. Pull page two of every list to capture the full top 100 and both figures double.

Set those against the tiers. The 1,000 free credits buy about five sweeps. Scaling's 5,000,000 credits buy a million Amazon page loads a month, roughly 33,000 a day, so a daily single-marketplace sweep uses about a tenth of one percent of it. Scaling exists for multi-marketplace price tracking across dozens of categories; watching a best-seller list is not what it is for. That gap between the smallest and largest tiers is where most readers' actual volume sits.

## Where the structured data comes from

ScraperAPI returns the page, not a curated product record. Its `autoparse` parameter is listed at no extra credit cost, and coverage varies by page type, so confirm the field set for the exact page you're hitting before you design a schema around it. Everything else you extract yourself.

The fields worth planning for, based on what competing Best Sellers tools return: rank position, ASIN, title, price, star rating, review count, category name and URL, parent categories, thumbnail, product URL, currency and HTTP status.

Three parsing traps show up in the sample output of every tool in this category:

- **Prices arrive as strings with local formatting.** A US listing comes back as `$31.49`, a German one as `5,95 €`. Discounted listings can carry a second price field next to the sale price. Normalizing to a decimal plus a currency code is your job, and it has to handle comma decimals, thousands separators and local symbols.
- **Review counts are abbreviated on the page itself.** `1.2K` and `1,2 mil` are what Amazon displays. No scraper can recover exact totals from a listing page, so treat review counts as approximations and don't compute growth rates from them.
- **Ratings sometimes arrive as a bare integer** when the page shows whole stars. Casting everything to a float on ingest saves a type error three joins downstream.

Amazon's Conditions of Use prohibit scraping, and that exposure sits with you and your legal team rather than with the API vendor.

## The failures that truncate a batch

Two mechanisms decide whether a 5,000-page run finishes.

**Concurrency.** Exceed your plan's thread count and ScraperAPI returns 429 and refuses the request. It doesn't queue it, and it doesn't bill extra for the attempt. A crawler that treats 429 as fatal ends the batch there; one that backs off and retries finishes it. Size your parallelism below the thread limit instead of discovering the limit mid-run.

**Credit exhaustion.** Scraping pauses until credits renew by default. Auto-upgrade, which moves you to the next tier when credits run out, is available on Hobby, Startup, Business, Scaling and Professional. Pay-as-you-go, which keeps your plan and bills extra credits at a fixed rate against a monthly cap you set, exists only on Scaling, Professional, Advanced and Enterprise. If you expect occasional spikes and don't want to change tiers, that gap should influence which tier you pick.

Compare how ready-made tools handle the same situation. The 2023 Apify Best Sellers actor writes typed error records into the dataset (`invalid_url`, `bestsellers_category_not_found`, `no_results_found`), so a batch completes with per-row diagnostics. The older open-source Apify actor behind several tutorials ends its run immediately when an error occurs, without writing any data to the dataset. Both behaviours are documented; only one of them lets you resume a 37-category sweep without starting over.

👉 [Compare thread limits and overage rules before sizing a crawler](https://www.scraperapi.com/pricing?fp_ref=coupons)

## ScraperAPI vs ready-made Best Sellers actors

| Tool | Marketplaces | Fields you get | When a page fails |
| --- | --- | --- | --- |
| ScraperAPI | Any Amazon domain you send | Page HTML; `autoparse` at no extra credit cost, coverage varies | 429 on thread overflow, refused not queued |
| Apify Fast Amazon Bestsellers Scraper (2025) | 18 | asin, name, price string, URL, category, parents, thumbnail, stars, reviewsCount | Not specified on the actor page |
| Apify Amazon Bestsellers Scraper (2023) | 6 | name, numeric price, currency, URL, thumbnail, numberOfOffers | Typed error items written to the dataset |
| Open-source Apify actor (m-murasovs) | 6 | name, price string, URL, thumbnail | Run stops, nothing written |
| Crawlbase-based Python script (ScraperHub) | One script run | Title, price, rating, reviews, ASIN, Prime status, rank, categories, pagination | Depends on your script |

The marketplace counts drift with version, which is the thing to watch when you compare: the 2025 actor covers 18 locales from the US through Turkey, while the 2023 actor and the 2020-era open-source repository both stop at 6. Same platform, three generations of coverage.

On cost, the open-source actor's README quotes 0.6 compute units per 100 pages, with roughly 160 pages per compute unit at about 25 cents each. Those are the repository's own numbers from a project started in 2020, so treat them as a ballpark rather than a current price: on that basis a thousand pages runs around $1.50, roughly three times ScraperAPI's Amazon rate at the Scaling tier. Different billing models, different years; use it to sanity-check an order of magnitude, not to settle a purchase.

Depth is the other shared constraint. The Apify actors default to 37 top subcategories with crawl depth capped at two subcategory levels, and the documented workaround is to scrape two levels, deduplicate the category URLs, then feed them back in. If your plan assumes full-catalog coverage in one pass, that limit invalidates it before price does.

## Who shouldn't pay for an API

A one-off table for a single category is a free-tier job. At 185 credits per full sweep, ScraperAPI's free allowance covers five of them, and a subscription buys you nothing extra if you never run it again.

If you need JSON tomorrow and have no parser, a ready-made actor that returns `stars` and `reviewsCount` as typed fields gets you to an analysis faster than an HTML pipeline will. ScraperAPI hands you the page and the proxy rotation; the field extraction is a project.

If you scrape one marketplace a few times a year, a monthly commit is the wrong shape for the work. And if your volume stays under 200 Amazon pages a month indefinitely, the free tier is the answer, not the entry-level paid plan.

👉 [Check what the free plan includes before buying anything](https://www.scraperapi.com/pricing?fp_ref=coupons)

## Choosing a tier by page count

Work backwards from pages, not from credits. Multiply your monthly Amazon page count by 5, and triple it for any page type that needs rendering.

- **Under 200 Amazon pages a month:** free tier. That covers a daily sweep of a handful of categories, or a weekly one across all 37.
- **A daily sweep of one marketplace without rendering** (about 5,550 credits): the smallest paid tier that clears that with headroom for retries. Check the live page for which one that is today, because the answer changes with each re-pricing.
- **Several marketplaces, six figures of monthly pages, parallel workers:** Scaling at 5,000,000 credits and 200 threads, $475/month or $427.50/month on annual billing.
- **Predictable baseline with unpredictable spikes:** pay-as-you-go, which starts at Scaling on the self-serve ladder.

Two things to confirm before you enter a card. First, whether your target pages need `render=true`, because that single parameter turns $0.48 per 1,000 pages into $1.43. Test one URL with it and one without, then read the `sa-credit-cost` header on both. Second, how overage is handled on the tier you're choosing: pause, auto-upgrade, or capped pay-as-you-go are three different behaviours and only two of them are available on the cheaper plans.

👉 [Pick the tier that matches your monthly page count](https://www.scraperapi.com/pricing?fp_ref=coupons)
