# user agent list for scraping: Copy-Paste Chrome, Firefox & Safari Strings, Plus the Rotation Rules That Actually Keep You Unblocked

The short answer: a current desktop Chrome string on Windows is the safest default for scraping, because Chrome holds roughly two-thirds of global browser traffic (StatCounter put it near 69% in June 2026) and blending in is the whole game. But most "user agent list" articles skip the part that actually decides whether you get blocked: your UA string has to match your Client Hints headers, your Accept headers, and increasingly, your TLS fingerprint. Below you'll get a copy-paste list, the rules that make rotation work, and the point at which maintaining your own list stops making sense.

**Quick picks**

> - Just testing? Python `requests` default (`python-requests/2.x`) is the fastest way to get a 403. Swap in one Chrome string from the table below before anything else.
> - Need scale or protected targets (Amazon, Google, LinkedIn)? Hand-maintained lists hit a wall fast. ScraperAPI's free plan gives you 1,000 credits with no credit card, and its 7-day trial adds 5,000 more — 👉 [see current plans and credit pricing](https://www.scraperapi.com/pricing?fp_ref=coupons)
> - Annual billing saves 10% on every paid tier if you stick around.

## The user agent list for scraping that works right now

Copy these as a starting pool. Versions age fast — Chrome was on 130 in January 2026 and 151 by July, so treat the version number as a slot to refresh monthly, not a permanent value.

| Scenario | User-Agent string |
| --- | --- |
| Chrome / Windows 10–11 (default choice) | `Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36` |
| Chrome / macOS | `Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36` |
| Firefox / Windows | `Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:133.0) Gecko/20100101 Firefox/133.0` |
| Safari / macOS | `Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/17.6 Safari/605.1.15` |
| Edge / Windows | `Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/130.0.0.0 Safari/537.36 Edg/130.0.0.0` |
| Chrome / Android | `Mozilla/5.0 (Linux; Android 14; Pixel 7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/130.0.0.0 Mobile Safari/537.36` |
| Safari / iPhone | `Mozilla/5.0 (iPhone; CPU iPhone OS 17_6 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/17.6 Mobile/15E148 Safari/604.1` |

Two structure notes worth knowing before you modify anything. Every segment in a Chrome string has a job: `Mozilla/5.0` is a legacy token every modern browser keeps, `Windows NT 10.0; Win64; x64` declares OS and architecture, `AppleWebKit/537.36` is the engine marker, `Chrome/151.0.0.0` is the browser and version, and the trailing `Safari/537.36` is a compatibility token Chrome still sends even though it runs on Blink. On Apple Silicon Macs, `Macintosh; Intel` should become `Macintosh; arm64` or the string describes a machine that doesn't exist.

Where each string fits: Chrome/Windows for general scraping, SERPs and e-commerce; Firefox to break up an all-Chromium rotation pool (it runs Gecko, a genuinely different engine); Safari for Apple-hostile-to-others sites and publisher stacks; Edge for Microsoft properties and enterprise B2B portals; mobile strings when a target serves different HTML to phones, like AMP pages or m-only layouts.

## Why the list alone gets you blocked

A realistic UA with everything else wrong is worse than no effort at all, because it looks like something pretending to be a browser. Modern bot detection correlates the UA with several other signals:

- **Header consistency.** Your `Accept`, `Accept-Language`, `Accept-Encoding` and Client Hints must match the claimed browser. A Firefox UA sending Chrome's Client Hints is an instant tell — Firefox doesn't send `Sec-CH-UA` at all, so don't add it.
- **Version mismatch.** A current version in the UA but an old or missing version in the Client Hints is one of the strongest bot signals a scraper can leak. Chrome now sends `Sec-CH-UA`, `Sec-CH-UA-Mobile` and `Sec-CH-UA-Platform` alongside its UA, and all four have to agree. For Chrome 151 on Windows that means:

python
headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36",
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8",
    "Accept-Language": "en-US,en;q=0.9",
    "Accept-Encoding": "gzip, deflate, br, zstd",
    "Sec-CH-UA": '"Chromium";v="151", "Google Chrome";v="151", "Not)A;Brand";v="8"',
    "Sec-CH-UA-Mobile": "?0",
    "Sec-CH-UA-Platform": '"Windows"',
}


The `Not)A;Brand` entry is a GREASE value Chrome deliberately changes every release to break hard-coded lists, so copy yours from `navigator.userAgentData` in your own browser's DevTools rather than from any article.

- **TLS and HTTP/2 fingerprints.** Cipher suites, ALPN and flow control can betray a `requests` or `curl` client no matter what UA it claims.
- **Behavior and IP reputation.** Requesting faster than a human, never touching `robots.txt`, or hitting a site from a datacenter IP all override a perfect UA.

A debugging pattern that works in practice when you get 200 OK with blank HTML or a CAPTCHA page: log the full request and response, switch UA families (Chrome to Firefox, or desktop to mobile), re-check header pairing, rotate IPs, and if needed render JavaScript to see what a real browser would receive.

## Rotation rules that most lists skip

Refreshing strings monthly beats rotating through stale ones. Aged-out strings create a fingerprint that matches nothing in the wild — several 2026 guides converge on this, suggesting a monthly review against sources like WhatIsMyBrowser or auto-updating lists like the microlinkhq/top-user-agents GitHub project, which rebuilds its top 100 weekly from over 300 million monthly real-traffic requests.

Keep the pool small and weighted toward reality. Chrome should dominate your rotation in proportion to its real market share, with Firefox and a mobile string as diversity. A pool of twenty equally-weighted exotic UAs is itself a pattern.

## When to stop maintaining your own list

There's a point where UA management stops being the bottleneck and everything else — proxies, CAPTCHA solving, JavaScript rendering, header consistency at scale — becomes the job. That's the territory of scraping APIs, and it's worth being honest about the trade-off before spending money.

ScraperAPI handles UA rotation automatically: it assigns realistic user agents per request and pairs them with matching headers, and its pricing page lists **desktop and mobile user agents** among the features included on every plan. Its docs are unusually candid about the alternative, though. If you send your own custom headers with `keep_headers=true`, you override their internal header management system, which the docs say "oftentimes lowers your success rates." Their explicit advice: don't use custom headers unless a site genuinely requires them, and for mobile results use the `device_type` parameter instead of hand-setting a UA — `device_type=desktop` is the default behavior anyway.

That's a direct counterpoint to the usual "maintain a 50-string list" advice, and it's worth testing before you invest weekends in list upkeep. The free tier makes that cheap: register and you get 1,000 credits with up to 5 concurrent connections, or 👉 [start the 7-day trial with 5,000 credits](https://www.scraperapi.com/signup?fp_ref=coupons) if you want to test against real targets first — no credit card required for either.

## What ScraperAPI costs (all current plans)

Prices are per month, with the annual-billing price (10% off) alongside:

| Plan | Price / month (annual) | API credits | Concurrent threads | Link |
| --- | --- | --- | --- | --- |
| Free | $0 | 1,000 | 5 | [Create a free account](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Hobby | $49 ($44.10) | 100,000 | 20 | [View Hobby pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Startup | $149 ($134.10) | 1,000,000 | 50 | [View Startup pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Business | $299 ($269.10) | 3,000,000 | 100 | [View Business pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Scaling (most popular) | $475 ($427.50) | 5,000,000 | 200 | [View Scaling pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Professional | $975 ($877.50) | 10,500,000 | 300 | [View Professional pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | $1,975 ($1,777.50) | 21,500,000 | 500 | [View Advanced pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | Custom quote | 22,000,000+ | 500+ | [Contact sales for custom volume](https://www.scraperapi.com/pricing?fp_ref=coupons) |

Every paid plan includes JavaScript rendering, premium proxies, JSON auto-parsing, rotating proxy pools, CAPTCHA and anti-bot bypass, custom session support, automatic retries, unlimited bandwidth and a 99.9% uptime guarantee. Geotargeting differs by tier: Hobby and Startup are limited to US and EU locations, Business and above unlock country-level targeting, and the analytics dashboard only keeps the last 30 days of history on the two lower tiers.

**The credit math nobody puts next to the plan prices.** "100,000 credits" does not mean 100,000 pages. A standard page costs 1 credit, but Amazon costs 5, Google and Bing (including subdomains) cost 25, LinkedIn costs 30, and requests that trigger a bypass of protection like Cloudflare, Datadome or PerimeterX add 10 credits each. On the Hobby plan, that 100,000-credit allowance is 20,000 Amazon pages, or about 4,000 Google searches — before any protected-site surcharge. The dashboard's Domain Cost Estimator shows the exact cost for any URL, and a `max_cost` parameter caps what any single request can burn. If your targets are mostly search engines or LinkedIn, run that math before picking a tier; it usually pushes you one plan higher than the headline credit count suggests.

## Choosing a plan (and who shouldn't bother)

If your targets are ordinary websites with no heavy protection, Hobby at $49 covers 100,000 standard pages a month — that's genuinely plenty for most side projects, and the 20 concurrent threads matter more than the credit count once you parallelize.

Startup makes sense when geotargeting beyond US/EU isn't needed but volume is: ten times the credits for three times the price. Business and Scaling are where country-level targeting and unlimited analytics history arrive; Scaling's 200 threads exist for genuinely parallel pipelines. Professional and Advanced are for teams whose monthly spend already justifies them — if you're not sure, you're not there yet.

Enterprise exists for 22M+ monthly requests, and the official guidance is simply to talk to sales at that volume.

Also worth knowing before you commit: exceeding your concurrency limit returns a 429 asking you to throttle and retry, with no overage charge — requests are turned away rather than queued. When credits run out, scraping pauses by default until renewal; Hobby through Professional can opt into auto-upgrade to the next tier, while Scaling and above can enable pay-as-you-go at a fixed per-credit rate with a monthly cap you set.

For context on alternatives: ScrapingBee positions its scraping API around automatic UA assignment with matching headers, and ZenRows sells a dedicated UA rotator — both solve the same problem from the API side. If you'd rather stay self-hosted, nothing stops you from running your own rotation pool; the list above plus monthly refreshes is a legitimate approach for small, unprotected targets.

## FAQ

**Does rotating user agents alone prevent CAPTCHAs?**
No. Rotation helps with soft blocks, but CAPTCHAs usually come from IP reputation, TLS fingerprints or behavioral analysis. If you're seeing CAPTCHAs with clean UAs, the problem is elsewhere in the stack.

**What UA should I use with Python requests?**
Any current desktop Chrome string from the table above, sent via the `User-Agent` header. Better still, test it: `httpbin.org/user-agent` echoes back exactly what your request carried, so you can confirm the swap worked before pointing it at a real target.

**How often should I update my list?**
Monthly is the practical cadence most current guides recommend. If a target suddenly starts blocking you after months of stability, check whether your Chrome version drifted out of the real-world distribution before blaming your proxy setup.

## The practical bottom line

Hand-rolled works if you're scraping a handful of unprotected sites with plain HTTP libraries: take the Chrome/Windows string, pair the headers correctly, refresh monthly, and stop there. The moment targets include Amazon, Google, LinkedIn or anything behind Cloudflare or Datadome, the credit multipliers and fingerprint checks make DIY much more expensive than it looks — that's when an API like ScraperAPI earns its price, and you can 👉 [compare plan tiers and start with 1,000 free credits](https://www.scraperapi.com/?fp_ref=coupons) to see whether it fits your targets before paying anything. One last check before any paid plan: confirm the credit cost of your specific target URLs in the dashboard's estimator, since that number, more than the plan price, determines what you'll actually spend.
