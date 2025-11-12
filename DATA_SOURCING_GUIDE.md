# Data Sourcing Guide

This document outlines the strategy for gathering mortgage rate data for the project. The primary constraint is that the method must be free. This leads to a web scraping-focused approach.

## Guiding Principles

1.  **Client-Side Limitations**: Direct web scraping from a browser (client-side JavaScript) is not feasible due to **CORS (Cross-Origin Resource Sharing)** policies. Websites do not allow `fetch` requests from other domains.
2.  **Proxy/Serverless Solution**: To circumvent CORS, the actual scraping must be done on a server. A lightweight and free solution is to use a **serverless function** (e.g., on Vercel or Netlify) that acts as a proxy. The client-side app will call this function, which then scrapes the target website and returns the data.
3.  **Reliability and Maintainability**: Scraping is fragile. Websites change their layout, breaking the scraper. Our strategy must prioritize reliable sources and have a clear plan for maintenance.

## Data Sourcing Strategy

### Step 1: Identify Primary Data Sources (AI-Assisted Research)

We will use a combination of traditional search and AI (specifically Gemini `gemini-2.5-flash-latest`) to identify the best sources.

**Prompt for Gemini/Search:**
> "You are `gemini-2.5-flash-latest`. I am looking for free sources of US mortgage rate data. I need both national averages and data specific to zip codes or local areas. I also need to find data for local lenders. What are the most reliable public websites, APIs, or financial data aggregators for this? For each source, can you identify the specific web pages that list the rates?"

**Potential Sources to Investigate:**
*   **National Averages (High Reliability)**:
    *   **Freddie Mac Primary Mortgage Market Survey® (PMMS®)**: Publishes weekly national average rates. This is a highly reliable, though not granular, source.
    *   **Federal Reserve Economic Data (FRED)**: Provides historical data on mortgage rates. Excellent for populating the initial database with long-term trends.
*   **Local/Zip-Code Level Data (Medium Reliability, Scrape-Heavy)**:
    *   **Zillow Mortgages**: `zillow.com/mortgage-rates/`
    *   **Bankrate**: `bankrate.com/mortgages/mortgage-rates/`
    *   **NerdWallet**: `nerdwallet.com/mortgages/mortgage-rates`
    *   These sites aggregate rates from various lenders based on zip code. They are prime targets for scraping but are complex and may have anti-scraping measures.
*   **Local Lenders**:
    *   Finding local credit unions and banks requires a targeted search. We can use a prompt like: **"List the top 5 local mortgage lenders and credit unions in zip code [zip_code] and find their websites."** This is likely a manual or semi-automated step to build a directory of local lender websites to scrape.

### Step 2: Develop the Scraping Logic

For each target site, we will need a specific scraper. The process is:
1.  **Inspect the Page**: Use browser developer tools to identify the HTML elements (and their CSS selectors) that contain the rate data (e.g., rate percentage, APR, lender name).
2.  **Write the Scraper Function**: Using a library like `Cheerio` (on the serverless function), load the page's HTML and extract the text/data from the identified selectors.
3.  **Structure the Data**: Format the scraped data into a consistent JSON structure, e.g.:
    ```json
    {
      "lenderName": "Example Bank",
      "rate": 6.85,
      "apr": 6.95,
      "loanType": "30-Year Fixed",
      "source": "zillow.com"
    }
    ```

### Step 3: Fallback Strategy

Not every zip code will have data, and scrapers can fail.
*   **If a zip-code-specific scrape fails**: Fall back to scraping for the wider county or state.
*   **If all local scrapes fail**: Fall back to the latest national average from a reliable source like Freddie Mac.
*   **UI Indication**: The user interface must clearly state the source of the data (e.g., "Live rates for 90210" vs. "Showing national average").
