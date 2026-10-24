# web scraping without getting blocked: Why Most Bans Are Behavior Problems, the Free Fixes to Try First, and When a Paid Unblocker Like ScraperAPI Is Worth It

Getting blocked while scraping almost always comes down to four things: your IP's reputation, your request headers, your TLS fingerprint, and your behavior. The good news is that most of these problems have free fixes, and most websites you'll ever scrape don't run serious anti-bot protection anyway. The expensive stuff only becomes necessary when you hit sites guarded by Cloudflare, DataDome, or Akamai — and that's the point where a managed unblocking API like ScraperAPI starts making sense.

This guide walks through the escalation ladder in order, because paying unblocker prices for a page that only wanted a proper User-Agent is the most common way people overspend on scraping. It also covers the part most reviews skip: how the credit math actually works once you do start paying.

> **Quick take:** If you're scraping ordinary sites, a real User-Agent plus polite delays will get you through for free. If you're up against sites with real bot protection, ScraperAPI handles proxies, CAPTCHAs, JS rendering, and retries through a single API endpoint. Plans start at **$49/month** (Hobby, 100,000 credits), there's a 7-day trial with 5,000 credits and no card required, and annual billing knocks 10% off every plan. 👉 [check current ScraperAPI plan prices](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## Why websites block scrapers in the first place

Sites don't block you because they detected "a bot" in the abstract. They block you because one or more specific signals failed a check. There are four layers, and they're evaluated roughly in this order:

1. **IP reputation.** Datacenter IPs from AWS, GCP, and most cheap proxy providers are pre-flagged and trusted less. Residential and mobile IPs score much higher, and cost more in that order.
2. **Headers.** The single most common giveaway is the default User-Agent string from `python-requests/2.x`. And one real UA isn't enough — modern anti-bot systems expect a full, internally consistent header set (`Accept`, `Accept-Language`, `Accept-Encoding`, the `Sec-Fetch-*` family), with your UA matching the rest. A Chrome UA paired with Firefox-style headers is itself a red flag.
3. **TLS fingerprint.** The TLS handshake happens before any headers are sent, and Python's `requests`, Go's default client, and plain cURL all produce JA3/JA4 hashes that no real browser would emit. Anti-bot systems also read HTTP/2 frame ordering. This is why a perfect residential IP can still get a 403: the proxy only fixes the reputation problem, nothing else.
4. **Behavior and JS challenges.** Fast, perfectly even request rhythms are the most obvious bot signature of all. Harder sites also run challenge pages ("Just a moment…") that probe your hardware, canvas, timezone, and browser internals with JavaScript — something a pure HTTP client can't pass at all.

The practical takeaway: you have to clear every layer your target actually checks. You don't have to pay for layers it doesn't.

## The free fixes to try first

Work through these in order and stop the moment you're getting clean 200s.

1. **Read the target's `robots.txt` and slow down.** Add randomized delays (something like 2–6 seconds between requests), respect `Crawl-delay`, and use exponential backoff with jitter when you hit a 429 or 403. Sustained high-frequency traffic is how a soft 429 escalates into a hard IP-level 403.
2. **Send a real, current User-Agent with a full header set.** Rotate a small pool of up-to-date browser UAs — a Chrome 89 string in an era of Chrome 12x is its own detection signal — and keep the UA, platform, and browser version consistent across headers.
3. **Switch to residential proxies if your IP is the problem.** Clean IP still 403s usually means reputation. Residential pools fix this; they're billed per GB rather than per request.
4. **Match your TLS fingerprint with `curl_cffi`** (the Python binding of curl-impersonate). One line — `cffi.get(url, impersonate="chrome124")` — replaces the `requests` call and fixes the JA3 mismatch. Its limit: it executes no JavaScript, so real JS challenges still get through.

This stack clears what one detailed industry guide calls "the long tail of the web" — blogs, documentation, most product pages — at zero cost. Dedicated unblocking services benchmark at roughly 97–99% success on protected targets, which tells you how big the gap is once a site actually fights back. That gap is the maintenance burden you'd otherwise own yourself.

## When a managed unblocker is worth paying for

A managed unblocking API earns its price when your targets run Cloudflare, DataDome, Akamai, or PerimeterX, or when you need JS rendering and CAPTCHA handling at volume and don't want to maintain headless browsers and proxy pools yourself. That's the category ScraperAPI was built for: you send a URL to one endpoint, and it handles proxy rotation, retries, geotargeting, CAPTCHAs, and optional JS rendering, then returns the HTML or parsed JSON.

The current lineup, straight from the official pricing page:

| Plan | Monthly | Annual (per mo) | Credits / mo | Concurrency | Get it |
| --- | --- | --- | --- | --- | --- |
| Free Trial | $0 (7 days) | — | 5,000 (one-time) | 5 | [start free, no card needed](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Hobby | $49 | $44.10 | 100,000 | 20 | [get Hobby](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Startup | $149 | $134.10 | 1,000,000 | 50 | [get Startup](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Business | $299 | $269.10 | 3,000,000 | 100 | [get Business](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Scaling | $475 | $427.50 | 5,000,000 | 200 | [get Scaling](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Professional | $975 | $877.50 | 10,500,000 | 300 | [get Professional](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Advanced | $1,975 | $1,777.50 | 21,500,000 | 500 | [get Advanced](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Enterprise | Custom | Custom | 22,000,000+ | 500+ | [talk to sales](https://www.scraperapi.com/contact-sales/?fp_ref=coupons) |

Every paid plan includes JS rendering, premium residential and mobile IPs, CAPTCHA and anti-bot handling, automatic retries, custom headers and sessions, unlimited bandwidth, and a 99.9% uptime guarantee. Beyond pricing, three plan-level limits matter and rarely show up in comparison tables — they're covered in the next section.

## The credit math most reviews skip

The number that actually determines your cost per page isn't the plan price. It's the credits-per-request table, because **a credit is not a request.** Standard pages cost 1 credit, but per the official pricing page:

| Target | Credits per request |
| --- | --- |
| Standard page | 1 |
| Amazon | 5 |
| Google / Bing (incl. subdomains) | 25 |
| LinkedIn | 30 |
| Anti-bot bypass (Cloudflare, Datadome, PerimeterX) | +10 |
| JS rendering (`render=true`) | +10 |
| `premium=true` / `screenshot=true` | +10 each |
| `ultra_premium=true` | +30 |

Run the numbers before picking a plan. Hobby's 100,000 credits is a lot of standard pages — but an Amazon product page with JS rendering costs 15 credits, so that's about 6,600 fetches. Google SERP requests run 25–35 credits depending on rendering, which cuts the same allowance to roughly 2,800–4,000 searches. ScraperAPI gives you a Domain Cost Estimator in the dashboard to check any URL before you commit, plus a `max_cost` parameter that caps what a single request can burn. Both are worth using from day one; the full domain cost list is documented on their pricing page.

Three billing mechanics catch people off guard:

- **Credits don't roll over.** Unused credits reset when your subscription renews. Size your plan to your average monthly usage, not your worst week.
- **Pay-as-you-go only exists on Scaling and above.** On Hobby, Startup, or Business, running out of credits means scraping pauses until renewal, unless you've opted into auto-upgrade, which moves you to the next plan automatically. There's no PAYG safety net on the cheaper tiers.
- **Exceeding your concurrency limit returns a 429.** The request is turned away, not queued, and there's no overage charge — but if your code doesn't handle 429s, it'll look like the site is blocking you when it's actually your own plan limit.

One more restriction worth checking against your use case: **geotargeting on Hobby and Startup is limited to US and EU regions.** Country-level targeting worldwide starts at Business ($299/month). If your project needs requests to appear from, say, Japan or Brazil, the $49 plan can't do it regardless of credits.

## How well does it actually avoid blocks?

Honestly? Depends entirely on the target. ScraperAPI advertises automatic CAPTCHA handling and IP rotation, and on ordinary and moderately protected sites it performs fine. Independent benchmarks are less flattering on hard targets: measured overall success rates range from about 52% to about 72.57% depending on the test and the site mix — above the roughly 58–59% industry average, but nowhere near the 97–99% that top-tier dedicated unblockers post. Reviewers specifically report 500 errors on some Cloudflare and DataDome pages, and one recurring complaint is that CAPTCHA or error pages occasionally come back billed as successful responses (support does refund credits when this is pointed out).

For calibration: Bright Data's Web Unlocker, the strongest competitor in this category, benchmarks at roughly 95–98% success on the same kinds of protected targets, at a higher price and with more integration friction. So the honest framing is this: ScraperAPI is a solid mid-market choice for easy-to-moderate targets at small-to-mid volume, and the wrong tool if your entire target list sits behind serious anti-bot systems.

You can judge for yourself cheaply. The signup tier includes 1,000 free credits (5 concurrent connections) with no card, and the 7-day trial bumps that to 5,000 — enough to run a few hundred real requests against your actual targets before paying anything. 👉 [start the 7-day free trial and test it on your own target list](https://www.scraperapi.com/signup?fp_ref=coupons)

## Who shouldn't pay for this at all

Plenty of people searching for ways to scrape without getting blocked don't need an unblocker yet. If your targets are blogs, docs sites, e-commerce pages without protection, or public APIs, the free stack — real headers, randomized delays, maybe `curl_cffi` — will carry you indefinitely. Spending $49/month there buys you nothing except convenience.

On the other end, if you're fighting DataDome or Akamai at scale every day, ScraperAPI's benchmark numbers suggest you'd be paying for misses. That's the segment where the premium unblockers justify their premium pricing, and it's worth benchmarking two or three services head-to-head on your specific targets before committing to an annual plan with any of them.

## FAQ

**How many requests do I actually get with 100,000 credits?**
Anywhere from about 2,800 (Google searches with rendering) to 100,000 (standard unprotected pages) per month. Check your specific domains with the Domain Cost Estimator before choosing.

**Does the free trial require a credit card?**
No. The 7-day trial includes 5,000 credits with no card, and the permanent free tier gives 1,000 credits at signup with up to 5 concurrent connections.

**What happens when I run out of credits mid-month?**
Scraping pauses until renewal by default. On Hobby through Professional you can enable auto-upgrade; on Scaling and above you can keep going with pay-as-you-go at a fixed per-credit rate under a monthly cap you set.

**Does switching to annual billing change anything besides price?**
It cuts every plan by 10% — Hobby drops from $49 to $44.10 per month, Advanced from $1,975 to $1,777.50 — and annual plans can renew early for a fresh batch of credits if you burn through them.

## Picking a plan in 30 seconds

- Scraping ordinary or lightly protected sites, under ~6,000 heavy requests a month → **Hobby at $49/month** is the right entry point.
- Need country-level geotargeting outside the US and EU → jump to **Business**; nothing cheaper can do it.
- Expecting usage spikes and don't want mid-month surprises → **Scaling**, the plan ScraperAPI itself flags as most popular, is the cheapest tier with pay-as-you-go.
- Your targets are mostly Cloudflare/DataDome-protected at high volume → benchmark ScraperAPI against a dedicated unblocker first, because its hard-target success rate is mid-pack.
- Just experimenting → the free tier costs nothing and needs no card.

Two things to confirm before you click buy: run your real target URLs through the Domain Cost Estimator so you're sizing plans in requests rather than credits, and start on monthly billing for the first month or two — annual billing's 10% discount is real, but you want a month of actual usage data before locking in a year. 👉 [compare all ScraperAPI plans and start free](https://www.scraperapi.com/pricing/?fp_ref=coupons)
