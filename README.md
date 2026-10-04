# serp scraping proxy: choosing the right IP type for Google rank tracking, local SERPs, and a sane cost per 1,000 queries

If your rank tracker suddenly returns empty pages, throws CAPTCHAs at 3 a.m., or reports that your client ranks #1 in a city you've never targeted, the problem is usually not your parser. It's the IP your requests are leaving from.

Google is a hostile target in a way that ordinary e-commerce sites are not. A residential proxy that sails through a product page can stall the moment you point it at `google.com/search`. This guide walks through what actually determines success on SERPs, which proxy type fits which job, the bandwidth math nobody shows you, and where 9Proxy's per-IP and per-GB pricing lands in all of it.

## Why SERPs break scrapers in the first place

Three things changed the game, and all three make proxy choice more important than parser code.

Google killed `num=100`. A single query that used to return 100 results now returns 10, so your request count for the same coverage multiplies. It also expects JavaScript rendering for search results now, which means most naive `requests.get()` pipelines get an empty shell back and report "no results found." And AI Overviews load on a delay, mostly for mobile audiences, so a scraper that reads the DOM the instant it loads sees an empty block and concludes the feature isn't there.

On top of that, the IP layer is where you get screened. Datacenter ranges get CAPTCHA'd after roughly 5–10 requests. Residential and mobile IPs hold up far longer, and mobile CGNAT addresses are shared with hundreds of real subscribers, which makes blanket banning them expensive for Google.

Even good providers don't win cleanly here. In AIMultiple's benchmark of 5,000 Google Search URLs sent through residential IPs at fixed intervals, the best performers landed around **52–54% success rate** with average response times near **1.5–1.8 seconds**. That's the realistic ceiling. Anyone promising you 99% success on Google is either measuring a different target or a different definition of success.

## Which proxy type for which SERP job

The "best proxy" question is under-specified. It depends on volume and on how precise your location needs to be.

| Proxy type | Plain-English version | Use it for | Don't use it for |
| --- | --- | --- | --- |
| Datacenter | Cheap cloud IP, fast, obviously a robot | Testing your parser logic, low-volume checks on easy engines like DuckDuckGo | Client-facing reports, local SEO, anything at volume on Google |
| ISP / static residential | A fixed IP with residential reputation, datacenter speed | Daily tracking of a bounded keyword set in one region, sticky sessions | Thousands of keywords across many cities |
| Rotating residential | Real home IPs that change per request or per session | Large keyword lists, multi-city rank tracking, production volume | Tiny one-off checks where the overhead isn't worth it |
| Mobile | Carrier IPs, expensive, hardest to block | AI Overviews, heavily defended queries, hyper-local accuracy | Budget-sensitive bulk collection |

A practical threshold: under roughly 500 keywords a day in one country, sticky ISP-style sessions are usually enough. Cross 500 keywords, or start serving multiple clients across multiple cities, and rotating residential becomes the default. Datacenter stays in the sandbox.

The other thing worth knowing: Google shows AI Overviews far more reliably to mobile audiences, so if AI Overview tracking is the actual requirement, residential will under-deliver no matter how clean the pool is. That's a mobile-job, and no residential provider fixes it.

## Sticky or rotating: the decision that quietly ruins reports

This one trips up more pipelines than bad IPs do.

For keyword research and bulk collection — you want coverage, and you don't care which IP served which query. Rotate on every request.

For rank tracking, consistency is the data. If Monday's "best crm software" result comes from an IP in Dallas and Tuesday's comes from an IP in Seattle, you're not measuring a ranking change, you're measuring geography. Hold one sticky session per location for somewhere between 10 and 30 minutes, run the queries, release it, move to the next location.

Pacing matters as much as rotation. Fixed intervals are one of the easiest patterns for Google to flag, so use randomized delays — somewhere in the 4 to 12 second range per request is the commonly cited window — plus jitter. Some teams also rotate the exit IP roughly every 5 minutes and vary city and carrier at the same time.

Two more details that separate a working SERP pipeline from a broken one: keep the User-Agent aligned with a current Chrome version and set `Accept-Language`, `Accept-Encoding`, and `Sec-CH-UA` to match, and set `gl` (country) plus a `uule` parameter so results stay consistent across runs regardless of which exit IP you drew. If you're doing local SEO, `uule` is not optional.

> One location per sticky session. Mixing cities inside a session produces ranking "changes" that are really just different SERPs.

## The cost model most guides skip

Most residential providers bill per gigabyte, and SERP scraping is a bandwidth business whether you think of it that way or not.

Here's the arithmetic that matters. At $2/GB:

| Page weight | Pages per GB | Cost per 1,000 pages |
| --- | --- | --- |
| 3 MB (full page, nothing blocked) | ~341 | ~$5.86 |
| 500 KB (scripts kept, images blocked) | ~2,048 | ~$0.98 |
| 80 KB (aggressive resource blocking, DOM only) | ~12,800 | ~$0.16 |

Read that again. Blocking images, fonts, and media changes your per-thousand cost by more than 30×. That is a bigger lever than any provider negotiation. A premium provider on a lean 80 KB page beats a budget provider on a fat 3 MB page, every time.

This is also where the billing model choice becomes the actual decision rather than a footnote. If your pages are heavy and unpredictable — modern SERPs are both — a per-gigabyte counter is a live grenade in your monthly budget. If your volume is high-rotation but light per request, per-GB is cheaper.

9Proxy sells both, which is unusual, and the difference between the two products is structural rather than cosmetic:

- **IP-based**: fixed number of residential IPs, unlimited bandwidth, unused IPs don't expire. An IP is deducted only when a connection is successfully established, not when a session is prepared.
- **GB-based**: pay for traffic only, generate unlimited endpoints, rotate or hold sticky sessions per your config, 180-day validity on standard GB packs, no expiry on Enterprise GB.

For SERP work, that split maps cleanly onto the two jobs above. Heavy pages and long-lived sticky sessions point at IP-based. Lightweight DOM-only extraction with heavy rotation points at GB-based.

👉 [See how 9Proxy's IP-based and GB-based models price out for SERP workloads](https://bit.ly/9-Proxy)

## 9Proxy plans and pricing

Worth flagging: 9Proxy adjusted pricing for IP-based and Bundle packages on June 1, 2026. GB-based pricing was left untouched. The numbers below reflect the post-adjustment structure.

### IP-based residential (unlimited bandwidth, non-expiring unused IPs)

| Package | Price per IP | Total | Buy |
| --- | --- | --- | --- |
| 100 IPs | $0.24 | $24 | [Get 100 IPs](https://bit.ly/9-Proxy) |
| 500 IPs | $0.144 | $72 | [Get 500 IPs](https://bit.ly/9-Proxy) |
| 1,000 + 500 bonus IPs | $0.084 | $126 | [Get 1,500 IPs](https://bit.ly/9-Proxy) |
| 2,500 IPs | $0.084 | $210 | [Get 2,500 IPs](https://bit.ly/9-Proxy) |
| 5,000 IPs | $0.072 | $360 | [Get 5,000 IPs](https://bit.ly/9-Proxy) |
| 15,000 IPs | $0.048 | $720 | [Get 15,000 IPs](https://bit.ly/9-Proxy) |
| 25,000 IPs | $0.035 | $863 | [Get 25,000 IPs](https://bit.ly/9-Proxy) |
| 50,000 IPs | $0.029 | $1,438 | [Get 50,000 IPs](https://bit.ly/9-Proxy) |

### Business IP packages (high volume)

| Package | Price per IP | Total | Buy |
| --- | --- | --- | --- |
| 100,000 IPs | $0.023 | $2,300 | [Get 100,000 IPs](https://bit.ly/9-Proxy) |
| 200,000 IPs | $0.021 | $4,140 | [Get 200,000 IPs](https://bit.ly/9-Proxy) |
| 500,000 IPs | $0.018 | $8,625 | [Get 500,000 IPs](https://bit.ly/9-Proxy) |

### GB-based residential (180-day validity)

| Package | Price per GB | Total | Buy |
| --- | --- | --- | --- |
| 5 GB | $3.00 | $15 | [Get 5 GB](https://bit.ly/9-Proxy) |
| 50 GB + 5 GB bonus | $2.10 | $105 | [Get 55 GB](https://bit.ly/9-Proxy) |
| 100 GB | $1.50 | $150 | [Get 100 GB](https://bit.ly/9-Proxy) |
| 200 GB | $1.00 | $200 | [Get 200 GB](https://bit.ly/9-Proxy) |
| 1,000 GB | $0.80 | $800 | [Get 1,000 GB](https://bit.ly/9-Proxy) |
| 2,000 GB | $0.75 | $1,500 | [Get 2,000 GB](https://bit.ly/9-Proxy) |

### Enterprise GB packages (no expiry)

| Package | Price per GB | Total | Validity | Buy |
| --- | --- | --- | --- | --- |
| 3,000 GB | $0.72 | $2,160 | Unlimited | [Get 3,000 GB](https://bit.ly/9-Proxy) |
| 6,000 GB | $0.70 | $4,200 | Unlimited | [Get 6,000 GB](https://bit.ly/9-Proxy) |
| 10,000 GB | $0.68 | $6,800 | Unlimited | [Get 10,000 GB](https://bit.ly/9-Proxy) |

### Bundle packages (IPs + bandwidth)

| Package | Price | Notes | Buy |
| --- | --- | --- | --- |
| 100 IPs + 5 GB | $30 | Starter-scale, good for a pilot | [Get the starter bundle](https://bit.ly/9-Proxy) |
| 1,500 IPs + 50 GB | $180 | Mid-scale agency workloads | [Get the mid bundle](https://bit.ly/9-Proxy) |
| 5,000 IPs + 500 GB | $720 | All-in package, discounted from list | [Get the pro bundle](https://bit.ly/9-Proxy) |

The per-IP curve is steep at the low end. 100 IPs costs $0.24 each; 50,000 IPs costs $0.029 each. If you're doing daily rank tracking across many cities, the practical question isn't "which tier is cheapest" but "how many IPs does one full sweep actually consume, and how much of that gets reused."

## What matters for SERP work specifically

Not every feature list item matters when the target is Google. These do:

**Targeting depth.** Country, state, city, ZIP, and ISP-level selection. ISP-level is the interesting one — pairing a city with a specific carrier produces a profile that looks like a real local user rather than a generic residential IP that happens to geolocate there.

**The 60-second replacement policy.** If an IP dies within 60 seconds of going live, it gets replaced free. Residential IPs churn by nature — they're real devices that go offline — and this is the difference between a pipeline that self-heals and one where somebody gets paged.

**Today List.** Any IP used in the last 24 hours can be reused at no extra cost if it comes back online. For daily rank tracking, where you're hitting the same keyword sets repeatedly, this is where the cost savings actually show up. One vendor-written guide puts the reduction in IP needs at 20–30%.

**Auto-refresh and auto-rotation.** Offline IPs get replaced within about 60 seconds; rotation runs on a schedule you set. Both exist to remove manual intervention from long-running jobs.

**Two access paths.** GB-based runs directly from the dashboard with username/password or IP whitelisting. IP-based proxies route through the 9Proxy desktop app via local port forwarding, with optional proxy authentication. That distinction matters if you're deploying to cloud infrastructure — the app-based path is a desktop requirement, and third-party reviews have flagged it as the main friction point for multi-device setups.

**Protocol and integration support.** HTTP, HTTPS, and SOCKS5, plus a public API for automated pipelines and session control. SOCKS5 support is what lets you drop it into anti-detect browsers, proxychains, or a custom script without protocol conversion.

**Support.** 24/7 via Telegram, email, and tickets.

## Putting it together: a working SERP setup

The pipeline shape is standard — request layer, parsing layer, storage layer — and the proxy is just a connection string inside the request layer. What changes per provider is the connection details.

1. **Decide the billing model first.** Heavy pages, sticky sessions, unpredictable bandwidth → IP-based. Light DOM-only extraction with high rotation → GB-based. Mixed → bundle.
2. **Pin your locations.** Pick country, then city or ISP. Build `uule` strings for each target location so results are reproducible.
3. **Set the session mode per job.** Rotate per request for keyword research; sticky 10–30 minutes per location for rank tracking.
4. **Block resources.** Kill images, fonts, and media in your browser automation. This is worth more than any pricing negotiation.
5. **Add jitter.** Randomize delays in the 4–12 second band. Fixed intervals get flagged.
6. **Detect CAPTCHA and rotate on it.** Treat a CAPTCHA response as a signal that the exit IP is burned, not as a retryable error.
7. **Reuse smartly.** Let the Today List pull back IPs that were recently live before you generate new ones.

👉 [Start with a 9Proxy account and test against your own keyword set](https://bit.ly/9-Proxy)

## Where 9Proxy isn't the right answer

Fairness matters more than cheerleading here.

**The pool is 20M+ IPs across 90+ countries.** That's a focused pool, not the largest on the market. For mainstream countries it's plenty. For highly specialized geographies, larger networks will have more depth.

**There are no mobile proxies.** If your requirement is AI Overview tracking or the most aggressively defended queries, residential tops out and you need carrier IPs. No amount of residential quality substitutes.

**The IP-based product requires the desktop app.** If your scraping runs on Linux cloud instances, the GB-based path is the one that fits, since it works straight from the dashboard.

**Free trials are promotional, not always-on.** You'll typically need to ask support for a trial code. Plan for a small paid test instead of assuming a free tier exists.

**Residential IPs die.** This is true of every residential provider, not just this one. What differs is how quickly the failure gets absorbed — which is what the replacement policy and auto-refresh are for.

Independent reviews land in a similar place: iTWire's assessment highlights affordable entry points and consistent operation for scraping with low reported downtime, while flagging the mandatory app and promotion-dependent trials as friction. A separate long-run test reported roughly 99.5% success rate and ~0.6s average response time on non-Google targets, which is a different — and easier — measurement than the SERP benchmarks above. Treat both numbers as context, not as a guarantee for Google specifically.

## FAQ

**Do I need residential proxies for SERP scraping, or is datacenter enough?**
Datacenter for parser development and low-volume checks on easy engines. Residential for anything production or Google-facing. Datacenter IPs get CAPTCHA'd after 5–10 requests.

**Sticky or rotating for rank tracking?**
Sticky. One session per location, 10–30 minutes, then move on. Rotating per request is for keyword research and bulk collection where you don't need per-IP consistency.

**How much does SERP scraping cost per 1,000 queries?**
Depends almost entirely on page weight. At $2/GB, a 3 MB page costs about $5.86 per 1,000; an 80 KB DOM-only page costs about $0.16. Block resources before you shop for cheaper GB.

**Can I use one proxy package for multiple clients?**
You can, but don't route every client through the same credentials. Give each client its own sub-user and traffic bucket so one client's block pattern doesn't contaminate another's data.

**Does 9Proxy support local SERP targeting?**
Yes — country, state, city, ZIP, and ISP-level targeting, which is what local pack and map pack tracking needs to be meaningful.

## The short version

For SERP scraping, the proxy type is the decision and the pricing model is the follow-up. Residential for organic results and volume, mobile if AI Overviews are the actual target, datacenter never for client-facing data. Then pick IP-based if your bandwidth is unpredictable and your pages are heavy, GB-based if you're rotating constantly on lightweight extractions, and a bundle if both are true.

The one thing to internalize before you spend anything: your per-thousand-query cost is decided more by how many kilobytes you request than by which provider you chose. Fix the page weight first. Then the rate card is a rounding error.

👉 [Check 9Proxy's current packages and get your account set up](https://bit.ly/9-Proxy)
