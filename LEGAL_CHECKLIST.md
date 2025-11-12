# Legal Scraping Checklist (Populated)

This checklist summarizes robots.txt and Terms observations for priority sources. It is not legal advice — it is a research summary to help you reduce risk. When in doubt, do not scrape; prefer official APIs or licensed data.

Fields per site:
- site: domain
- robots.txt: link + short summary
- terms_of_service: link + short note about automated access / scraping
- recommended_cadence: conservative cadence or guidance (or "avoid")
- notes: implementation notes and fallbacks

---

- site: zillow.com
  robots.txt: https://www.zillow.com/robots.txt — extensive Disallow rules; many endpoints blocked; sitemaps present.
  terms_of_service: https://www.zillow.com/corp/Terms.htm — explicitly prohibits "automated queries (including screen and database scraping, spiders, robots, crawlers, bypassing 'captcha' or similar precautions, or any other automated activity with the purpose of obtaining information from the Services)" (see Section 5). Zillow also restricts reuse and redistribution of data.
  recommended_cadence: AVOID automated scraping. Use Zillow only for manual checks or rely on Zillow-provided APIs/partners under license.
  notes: Zillow explicitly forbids automated scraping in its Terms. Do not target Zillow with an automated scraper. Use Freddie Mac/FRED or licensed partners instead.

- site: realtor.com (Move, Inc.)
  robots.txt: https://www.realtor.com/robots.txt — includes a legal notice: "scraping data from this website is unauthorized without the express written permission from Move Sales, Inc." plus many Disallow rules.
  terms_of_service: https://www.realtor.com/terms-of-service/ (site references and legal notice) — realtor.com forbids unauthorized scraping and recommends contacting them for permission.
  recommended_cadence: AVOID automated scraping unless you obtain express written permission.
  notes: realtor.com is explicitly guarded against scraping. Do not target without permission.

- site: bankrate.com
  robots.txt: https://www.bankrate.com/robots.txt — disallows many API and widget paths; allows some resources; sitemap available.
  terms_of_service: https://www.bankrate.com/terms/ (site lists Terms; pages exist) — Bankrate is an aggregator and generally restricts automated extraction in practice; while robots.txt does not block everything, TOS and anti-scraping measures are common.
  recommended_cadence: Prefer AVOID; if you must, use only public API endpoints (if any) and very conservative cadence (>=60 minutes) with caching, and confirm TOS before scraping.
  notes: Bankrate has a data center and rate pages; prefer official data center or public averages rather than scraping rate pages.

- site: nerdwallet.com
  robots.txt: https://www.nerdwallet.com/robots.txt — disallows API and many dynamic endpoints; sitemaps present.
  terms_of_service: NerdWallet terms and legal pages exist (e.g., https://www.nerdwallet.com/blog/terms-of-use) — automated scraping is not explicitly permitted; site architecture relies on APIs and dynamic content which may be protected.
  recommended_cadence: Prefer AVOID; if used, contact NerdWallet for permission or rely on user-provided data and national sources.
  notes: Aggregator content; anti-scraping protections likely. Use as fallback only.

- site: freddiemac.com (Freddie Mac)
  robots.txt: https://www.freddiemac.com/robots.txt — allows access to many public assets and disallows admin and certain admin pages; PMMS data is public on the Freddie Mac site and they publish PMMS (Primary Mortgage Market Survey) data.
  terms_of_service: https://www.freddiemac.com/terms — Freddie Mac provides public data; for PMMS consider using their published CSVs/APIs and cite source.
  recommended_cadence: Use official releases — PMMS updates weekly. Recommended cadence: 7 days (or update when Freddie Mac publishes new data).
  notes: Freddie Mac is the preferred authoritative national source for average mortgage rates. Prefer their official datasets rather than scraping HTML pages.

- site: fred.stlouisfed.org (FRED)
  robots.txt: https://fred.stlouisfed.org/robots.txt — generally allows robots; includes explicit crawl-delay rules and lists restrictions for certain endpoints. FRED provides APIs and downloadable series data.
  terms_of_service: https://fred.stlouisfed.org/legal/ (FRED legal pages) — FRED provides open data with clear terms and an API; use the API when possible.
  recommended_cadence: Use API; daily/weekly updates depending on series. Avoid scraping HTML when API is available.
  notes: FRED is an excellent source for historical series and official macro time series; use the API and attribute FRED.

---

General guidance
- When a site explicitly disallows automated access in its Terms (e.g., Zillow, Realtor), do NOT scrape.
- Prefer official APIs and published CSV datasets (Freddie Mac PMMS, FRED API) for national/regional data.
- For aggregator sites that are not explicitly permissive, request permission or use manual/user-provided flows (ask the user to paste a rate page or CSV) rather than automated scraping.
- If you must scrape a permissive site, implement conservative caching (TTL >= 10–60 minutes), strict rate-limiting, and honor `robots.txt` and `crawl-delay` directives.

This checklist is a working document — update entries with direct links to the specific Terms sections and any correspondence if you obtain permission.
