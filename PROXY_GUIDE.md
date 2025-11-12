# Serverless Proxy Guide — Options, Tradeoffs, and Recommendation

This document helps you choose and implement a free/easy serverless proxy for scraping mortgage rate data and optionally proxying Gemini calls. It assumes the client is a single-page, no-build app and therefore needs a server-side middleman to fetch third-party pages.

## Why a proxy is required
- Browsers enforce CORS and will block direct cross-origin fetches to other domains (e.g., `zillow.com`).
- A serverless proxy fetches the remote page (no CORS restriction on the server), scrapes/parses it, normalizes the data, and returns JSON to the browser.

## Candidate Providers (free tiers)
1. **Netlify Functions (recommended)**
   - Runtime: Node.js serverless functions (AWS Lambda under the hood).
   - Pros: Very easy to deploy from a repo, supports Node packages like `node-fetch` and `cheerio`, free tier is generous for low-frequency scraping, built-in caching via headers, can be deployed alongside static site.
   - Cons: Cold-starts are possible but acceptable for low-volume traffic.

2. **Vercel Serverless Functions**
   - Runtime: Node.js serverless (Edge functions also available).
   - Pros: Also supports Node packages, good developer ergonomics, integrates with GitHub, free tier suitable for small usage.
   - Cons: Similar cold-start behavior; default serverless execution limits should be checked.

3. **Cloudflare Workers**
   - Runtime: V8 isolates (not Node). Uses `fetch` for outbound requests and provides `HTMLRewriter` for parsing HTML.
   - Pros: Extremely fast and globally distributed; free tier is solid; great for small fetch/transform tasks.
   - Cons: Not Node: cannot directly use `cheerio` or other Node-native libraries. HTML parsing is done with `HTMLRewriter` (suitable but different code style). For complex scraping, you'll need to re-implement parsers.

4. **Fly.io / Render**
   - Pros: Can run small containers; flexible.
   - Cons: Slightly more setup than Netlify/Vercel. Free tiers exist but are more involved.

## Recommendation
- Start with **Netlify Functions** (or **Vercel** if you prefer) because they let you use standard Node tooling (`cheerio` for parsing, `node-fetch` for requests) and deploy easily alongside a static site. For low-frequency usage (a user requests a zip once per session), Netlify's free tier is sufficient.

## Basic architecture (Netlify example)
- Endpoint: `/api/getRates?zip=90210`
- Behavior:
  1. Validate the `zip` parameter.
  2. Check an in-function cache (or a simple KV if available) for a recent cached response (TTL, e.g., 10–60 minutes depending on source).
  3. If cache miss, fetch the target aggregator page(s) (e.g., Zillow Bankrate pages) using `node-fetch`.
  4. Parse HTML with `cheerio` and extract lender name, rate, APR, loan type, timestamp.
  5. Normalize into JSON: include `source`, `scrapeTimestamp`, and any `fallbackReason`.
  6. Return JSON to the client with appropriate `Cache-Control` headers.

## Security and operational considerations
- **Rate limiting**: Implement per-IP and per-endpoint rate limits in the function to avoid abuse and reduce scraping frequency. Netlify does not include rate-limiting out of the box, but you can implement a lightweight in-function limiter using an in-memory sliding window and a cache store (or use Cloudflare in front).
- **Caching**: Cache scraped results for a TTL to avoid repeated hits to the same third-party page. TTL can vary by source (e.g., 10–60 minutes). Cache can be in-memory for short TTLs or backed by an external store (paid) for longer-term caching.
- **User-Agent rotation**: Use a small list of reasonable user-agents when requesting aggregator pages to reduce blocking risk. Keep rotation conservative and respectful.
- **Respect `robots.txt` and terms of service**: Before scraping a domain, check `robots.txt` and the site's terms. If a site disallows scraping, do not target it. See the legal checklist strategy below.
- **Error handling & fallbacks**: If scraping fails, return a clear error and fallback to county/state or national averages. Include `source` and `isFallback` flags in the response.

## Gemini (AI) considerations — client-side keys
The project owner has chosen to keep Gemini API usage client-side: users will supply their Gemini API key locally and the app will call Gemini directly from the browser. The serverless proxy will be used only for scraping/avoiding CORS and not for relaying Gemini requests.

Key points and recommended UI behavior:
- **Local key entry**: Provide a settings UI where a user can paste their Gemini API key.
- **Storage options**: Default to session-only storage (in-memory while the tab is open). Offer an explicit toggle to "Store key locally" which writes the key to `localStorage` only after showing a clear security warning.
- **Key rotation / removal**: Expose a menu to change or remove the stored key. Removing clears `localStorage` and any in-memory reference.
- **Security warning**: Clearly document that storing the key in the browser exposes it to anyone with access to the device/profile and to XSS risks if the app later includes third-party scripts.

Why we are not proxying Gemini:
- The app is designed to run embedded and self-contained with no server-held secrets. Proxying Gemini would require a server-side secret; the owner prefers a local-only key model.

Developer notes:
- Verify that the Gemini endpoint permits direct browser-origin requests (CORS). If Gemini blocks direct browser calls, revisit the approach.
- Keep the key UI small and auditable; never send the key to any server from the client.

## Implementation checklist (Netlify)
1. Create Netlify account and link the GitHub repo.
2. Add a `netlify/functions/getRates.js` function that:
   - Validates input.
   - Caches & rate-limits.
   - Fetches target pages and parses with `cheerio`.
   - Returns normalized JSON.
3. Add environment variables in Netlify UI for any sensitive settings (e.g., GEMINI_API_KEY if proxying Gemini).
4. Deploy and test with a small set of zip codes.
5. Monitor logs and adjust caching/TTL to reduce scraping volume.

## When to use Cloudflare Workers instead
- If you need latency-sensitive scraping at global scale and are comfortable writing scrapers using `HTMLRewriter`, Cloudflare Workers are excellent. They have a steeper learning curve for scraping because they lack Node modules.

## Summary
- Netlify/Vercel: easiest, Node ecosystem compatible, recommended for this project.
- Cloudflare Workers: fastest & global but requires adapting scraping logic to the Workers APIs.
- Keep scraping conservative and respect robots.txt and site TOS. If a site blocks scraping, remove it from the target list and rely on other sources or national averages.