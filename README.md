# Scrapy Scraper API Complete Integration Guide: How to Connect ScraperAPI with Your Scrapy Spiders, Bypass IP Bans & CAPTCHAs, and Scale to Millions of Requests (With Pricing Breakdown & Free Trial)

If you've been writing Scrapy spiders for a while, you know the drill. You set up a beautiful spider, it runs fine for ten minutes, and then — blocked. Maybe it's a CAPTCHA. Maybe it's a 403. Maybe the target site just stopped responding like it was personally offended. Either way, you're now spending your Friday afternoon managing proxy lists instead of actually doing anything useful with the data you were trying to collect.

This is exactly the problem that **scrapy scraper api** integrations are designed to solve. Instead of babysitting proxies, rotating user agents, and debugging anti-bot systems, you let a dedicated service handle all of that — and just get back the HTML you actually wanted.

This guide walks through how to integrate ScraperAPI with Scrapy, what it actually does under the hood, how the pricing works, and whether it makes sense for your project.

---

## What Is Scrapy, and Why Does It Keep Getting Blocked?

Scrapy is an open-source Python framework built for high-performance web crawling. It's asynchronous, built on top of Twisted, and lets you run hundreds of concurrent requests without managing threads yourself. If you're doing any serious data collection project in Python, Scrapy is probably what you end up using eventually.

The problem isn't the framework itself — it's the internet. Websites have gotten much better at detecting automated traffic. They look at request patterns, response times, user-agent strings, IP reputation scores, and increasingly use services like Cloudflare, DataDome, or PerimeterX to block scrapers before they even get to the page.

The traditional workarounds — rotating free proxies, spoofing headers, adding random delays — work until they don't. And maintaining that infrastructure takes real time that could be spent on actually building things.

---

## What Does a Scrapy Scraper API Integration Actually Do?

When you integrate ScraperAPI with Scrapy, you're essentially routing your requests through ScraperAPI's infrastructure instead of sending them directly to the target site. From the perspective of your spider, you get back HTML like normal. From the perspective of the target site, the request looks like it came from a regular browser user somewhere in the world.

ScraperAPI handles:

- **Proxy rotation** — automatically cycles through a pool of over 40 million proxies across 50+ countries
- **CAPTCHA solving** — detects and handles CAPTCHAs so your requests don't fail silently
- **Browser rendering** — optionally renders JavaScript before returning the page, which is essential for React/Vue-based sites
- **Header management** — selects optimal headers for each target site to maximize success rates
- **Automatic retries** — retries failed requests on 500 status codes without charging for them

The result: you keep writing Scrapy spiders the same way you always have, but the requests actually succeed.

👉 [Start your free trial with 5,000 API credits — no credit card required](https://www.scraperapi.com/?fp_ref=coupons)

---

## Three Ways to Integrate ScraperAPI with Scrapy

There are three integration methods, and which one you choose mostly depends on how much you want to change your existing code.

### Method 1: API Endpoint (Most Flexible)

This method wraps your target URL into a ScraperAPI request URL. You add a helper function to your spider file and update the `Request` calls to use it.

python
import scrapy
from urllib.parse import urlencode

API_KEY = 'YOUR_API_KEY'

def get_scraperapi_url(url):
    payload = {'api_key': API_KEY, 'url': url}
    proxy_url = 'https://api.scraperapi.com/?' + urlencode(payload)
    return proxy_url

class QuotesSpider(scrapy.Spider):
    name = "quotes"

    def start_requests(self):
        urls = [
            'https://quotes.toscrape.com/page/1/',
            'https://quotes.toscrape.com/page/2/',
        ]
        for url in urls:
            yield scrapy.Request(url=get_scraperapi_url(url), callback=self.parse)


This gives you the most control over parameters. You can easily add `render=true`, `country_code`, `premium=true`, etc. directly in the payload dictionary.

### Method 2: Python SDK (Cleanest Code)

Install the SDK first:

bash
pip install scraperapi-sdk


Then your spider becomes:

python
import scrapy
from scraper_api import ScraperAPIClient

client = ScraperAPIClient('YOUR_API_KEY')

class QuotesSpider(scrapy.Spider):
    name = "quotes"

    def start_requests(self):
        urls = [
            'https://quotes.toscrape.com/page/1/',
            'https://quotes.toscrape.com/page/2/',
        ]
        for url in urls:
            yield scrapy.Request(client.scrapyGet(url=url), callback=self.parse)


This is the cleanest way to integrate if you're starting fresh. The SDK wraps the URL transformation for you.

### Method 3: Proxy Mode (Minimal Code Changes)

If you already have a large Scrapy project and don't want to touch the request logic, you can use ScraperAPI as a standard proxy. You pass the proxy address in the request `meta`:

python
import scrapy

class QuotesSpider(scrapy.Spider):
    name = "quotes"

    def start_requests(self):
        urls = [
            'https://quotes.toscrape.com/page/1/',
            'https://quotes.toscrape.com/page/2/',
        ]
        meta = {
            "proxy": "https://scraperapi:YOUR_API_KEY@proxy-server.scraperapi.com:8001"
        }
        for url in urls:
            yield scrapy.Request(url=url, callback=self.parse, meta=meta)


Scrapy skips SSL verification by default, so no additional configuration is needed for this to work.

---

## Configuring Your settings.py for ScraperAPI

Once you've chosen your integration method, you need to update `settings.py` to get the most out of your plan.

**Concurrency** — Set this to match the thread limit on your plan. Going higher than your plan allows will just result in throttled requests:

python
CONCURRENT_REQUESTS = 20  # Adjust to your plan's thread limit


**Retries** — ScraperAPI returns a 500 status on failed requests and doesn't charge for them. Set Scrapy to retry:

python
RETRY_TIMES = 3


**Disable robots.txt checking** — This is important. When you route through the API endpoint method, Scrapy will try to fetch `robots.txt` from ScraperAPI's domain instead of the target site, which doesn't work correctly:

python
ROBOTSTXT_OBEY = False


**Disable download delays** — These aren't needed when using ScraperAPI and will slow you down:

python
# Comment these out:
# DOWNLOAD_DELAY = 1
# RANDOMIZE_DOWNLOAD_DELAY = True


---

## Advanced ScraperAPI Parameters for Scrapy

When using the API endpoint method, you can pass additional parameters to customize how ScraperAPI handles each request:

| Parameter | What It Does |
|---|---|
| `render=true` | Enables JavaScript rendering — essential for dynamic/React sites |
| `country_code=us` | Routes through proxies in a specific country (great for geo-restricted content) |
| `premium=true` | Uses residential and mobile IPs — costs 10 extra credits per request |
| `session_number=123` | Reuses the same proxy for a session (useful for paginated scraping) |
| `keep_headers=true` | Passes your own custom headers through to the target |
| `device_type=mobile` | Uses mobile user-agent strings |
| `autoparse=true` | Returns structured JSON for select supported domains |

For example, to scrape a JavaScript-rendered page using US proxies:

python
def get_scraperapi_url(url):
    payload = {
        'api_key': API_KEY,
        'url': url,
        'render': 'true',
        'country_code': 'us'
    }
    return 'https://api.scraperapi.com/?' + urlencode(payload)


---

## ScraperAPI Plan Comparison: Which One Do You Actually Need?

Here's the full breakdown of every available plan. Credit costs vary by domain — standard pages cost 1 credit, Amazon costs 5, Google/Bing cost 25, LinkedIn costs 30, and sites with bot protection like Cloudflare add 10 extra credits per bypass.

| Plan | Monthly Price | Annual Price (10% off) | API Credits | Concurrent Threads | Geotargeting | Pay-As-You-Go | Link |
|---|---|---|---|---|---|---|---|
| **Free Trial** | $0 | — | 5,000 (trial) / 1,000 (ongoing) | 5 | — | ❌ |  [Start Free](https://www.scraperapi.com/?fp_ref=coupons) |
| **Hobby** | $49/mo | $44.10/mo | 100,000 | 20 | US & EU only | ❌ |  [Get Hobby](https://www.scraperapi.com/?fp_ref=coupons) |
| **Startup** | $149/mo | $134.10/mo | 1,000,000 | 50 | US & EU only | ❌ |  [Get Startup](https://www.scraperapi.com/?fp_ref=coupons) |
| **Business** | $299/mo | $269.10/mo | 3,000,000 | 100 | Global | ❌ |  [Get Business](https://www.scraperapi.com/?fp_ref=coupons) |
| **Scaling** ⭐ | $475/mo | $427.50/mo | 5,000,000 | 200 | Global | ✅ |  [Get Scaling](https://www.scraperapi.com/?fp_ref=coupons) |
| **Professional** | $975/mo | $877.50/mo | 10,500,000 | 300 | Global | ✅ |  [Get Professional](https://www.scraperapi.com/?fp_ref=coupons) |
| **Advanced** | $1,975/mo | $1,777.50/mo | 21,500,000 | 500 | Global | ✅ |  [Get Advanced](https://www.scraperapi.com/?fp_ref=coupons) |
| **Enterprise** | Custom | Custom | 22M+ | 500+ | Global | ✅ |  [Contact Sales](https://www.scraperapi.com/?fp_ref=coupons) |

All plans include: JS rendering, premium proxies, JSON auto-parsing, rotating proxy pools, custom header support, CAPTCHA and anti-bot detection, custom session support, desktop and mobile user agents, automatic retries, unlimited bandwidth, and 99.9% uptime guarantee.

Plans at the Business tier and above unlock global geotargeting and unlimited analytics history. Pay-As-You-Go becomes available at the Scaling tier and above, letting you continue scraping past your monthly credit limit at a predictable per-credit rate.

The analytics dashboard history is limited to the last 30 days on Hobby and Startup plans, and unlimited on Business and above.

---

## What Happens When You Run Out of Credits?

On Hobby, Startup, and Business plans, when you hit 100% of your monthly credits, your scraping stops — you'd need to upgrade or contact support. On Scaling and above, Pay-As-You-Go kicks in and you keep going at a fixed rate.

Unused credits don't roll over to the next billing cycle, so it's worth tracking usage through the dashboard. You can also set a `max_cost` parameter per request to avoid burning credits on unexpectedly expensive pages.

---

## Who Is Each Plan Actually For?

**Hobby ($49/mo):** Side projects, small datasets, testing new scrapers. 100,000 credits sounds like a lot until you're hitting Amazon — at 5 credits per request that's 20,000 product pages. Still solid for initial validation.

**Startup ($149/mo):** Low-to-medium volume workflows, small teams, early-stage data pipelines. 1 million credits handles meaningful production workloads for standard sites.

**Business ($299/mo):** The first plan with global geotargeting, which matters if you're scraping regionally specific content. 3 million credits and 100 threads is enough for most serious production scrapers.

**Scaling ($475/mo):** The most popular plan for a reason — 5 million credits, 200 threads, Pay-As-You-Go, global targeting. This is where most growing data teams end up.

**Professional and above:** High-volume pipelines where you need priority support, guaranteed routing, and enough capacity that running out mid-month would be a real problem.

---

## Beyond Scrapy: Other Ways to Use ScraperAPI

If you're not always using Scrapy, ScraperAPI works with essentially every approach to Python scraping — requests library, aiohttp, Playwright, Selenium. The same API key, same parameters, same pricing.

For teams that need data without writing any scraper code, ScraperAPI also offers:

- **Structured Data Endpoints** — pre-built parsers for Amazon, Google, Walmart, and other high-demand domains that return clean JSON
- **DataPipeline** — a no-code tool for scheduling and automating data collection jobs
- **Async Scraper Service** — for sending millions of requests asynchronously without managing queues yourself

The LangChain integration is particularly interesting for AI teams — it lets agents pull live web data on demand using the same ScraperAPI infrastructure.

👉 [Try ScraperAPI free — 5,000 credits, no credit card needed](https://www.scraperapi.com/?fp_ref=coupons)

---

## Common Questions

**Do I need ScraperAPI if my target site doesn't have anti-bot protection?**

Technically no. But IP bans from volume are still a problem even without CAPTCHA — if you're sending enough requests from one IP, most sites will rate-limit or block you eventually. ScraperAPI's proxy rotation handles that transparently.

**Does ScraperAPI work with Scrapy's auto-throttle?**

Auto-throttle is designed to slow down requests based on server response time — which defeats the purpose of paying for concurrent threads. It's better to disable auto-throttle and set `CONCURRENT_REQUESTS` manually.

**Is there a refund policy?**

Yes. ScraperAPI offers a 7-day no-questions-asked refund. If you try it and it doesn't work for your use case, you can get your money back immediately.

**Can I scrape any website?**

ScraperAPI handles the technical side of accessing public web pages. You're still responsible for complying with a site's terms of service and applicable laws. ScraperAPI is CCPA and GDPR compliant.

---

## Wrapping Up

The scrapy scraper api combination is genuinely one of the more practical setups for production web scraping. Scrapy handles the crawling logic — link discovery, data extraction, pipeline management — and ScraperAPI handles the part that usually breaks everything: actually getting through to the target page.

The integration itself takes maybe fifteen minutes. You change a few lines in your spider, update `settings.py`, and you're done. The free trial gives you 5,000 credits to test it on your actual target sites before committing to a plan.

If you've been spending time debugging proxy failures and CAPTCHA blocks instead of actually building with your data, it's worth trying.

👉 [Start your free trial at ScraperAPI](https://www.scraperapi.com/?fp_ref=coupons)
