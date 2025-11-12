# Mortgage Rate Analysis Tool - Implementation Plan

This document outlines the step-by-step plan to build the one-page mortgage rate analysis website.

## Phase 1: Project Setup & Core Structure (Est. Time: 1-2 days)

The goal of this phase is to establish the foundational structure of the application.

1.  **Initialize Project**:
    *   Create the main project folder.
    *   Create initial files: `index.html`, `assets/css/style.css`, and `assets/js/main.js`.
    *   Initialize a Git repository to track changes.

2.  **Set Up Tech Stack**:
    *   **Tailwind CSS**: Load Tailwind through the official CDN snippet (`https://cdn.tailwindcss.com`) to keep the project build-free while enabling utility-first styling.
    *   **Libraries (served via CDN)**:
        *   **Dexie.js**: Add the Dexie.js library for simplified IndexedDB operations. This will handle all local data storage.
        *   **Chart.js**: Add the Chart.js library for creating interactive charts and graphs.
        *   **Math.js**: Add Math.js for robust mathematical calculations, especially for the mortgage calculator.

3.  **Create Basic HTML Layout**:
    *   Structure `index.html` with semantic HTML5 tags (`<header>`, `<main>`, `<footer>`, `<section>`, etc.).
    *   Define the primary sections as placeholders:
        *   Header with the application title.
        *   User Input section (for zip code, home price, income).
        *   Data Display section (for charts, tables, and results).
        *   Mortgage Calculator section.
        *   Recommendations panel.
        *   Footer with data source attribution and disclaimers.

4.  **Seed National Baseline Data**:
    *   Add a static `data/national-averages.json` file populated with the latest Freddie Mac (PMMS) national averages for primary loan products.
    *   Document a lightweight manual refresh process (e.g., run a one-off script locally or paste updated values monthly) since no automated build pipeline will run.

5.  **Single-file structure and in-file documentation**:
    *   The app will be shipped as a single `index.html` file with all JavaScript embedded. To keep the file navigable, include a clear comment-based table-of-contents at the top and separate, well-labeled sections such as `<!-- LIBRARIES -->`, `<!-- CONFIG -->`, `<!-- STORAGE / DB -->`, `<!-- SCRAPER API -->`, `<!-- GEMINI CLIENT -->`, `<!-- CALCULATOR -->`, `<!-- CHARTS -->`, `<!-- EXPORT/IMPORT -->`, and `<!-- BOOTSTRAP -->`.
    *   Use CDN-hosted libraries only. Avoid any bundler/ESM-only features that require a build step.

## Phase 2: Data Sourcing & Local Storage (Est. Time: 3-5 days)

This phase focuses on setting up the data pipeline, from fetching to local storage.

1.  **Data Sourcing Research**:
    *   Execute the research plan outlined in `DATA_SOURCING_GUIDE.md`.
    *   Identify 2-3 primary national sources (e.g., Freddie Mac) and 2-3 aggregator sites (e.g., Zillow, Bankrate) for scraping.
    *   Use Gemini/search to find local lenders for a few sample zip codes to validate the approach.

2.  **Develop Scraping Module (`assets/js/scraper.js`)**:
    *   Create a JavaScript module responsible for fetching data.
    *   **Important**: Since direct scraping from a client-side JS application is blocked by CORS policy, this "scraper" will be a fetch module that calls a proxy or a serverless function (e.g., on Netlify/Vercel) that performs the actual scraping. For initial local development, we can use a CORS-proxy.
    *   Implement functions to fetch data from the identified sources.
    *   Include error handling for failed requests or changes in website structure.

3.  **Implement Database Service (`assets/js/database.js`)**:
    *   Use Dexie.js to define the database schema.
    *   Create a table for `rates` with indexes on `zipCode` and `date`.
    *   Implement service functions:
        *   `addRate(rateData)`: Adds new rate data to the database.
        *   `getRatesByZip(zipCode)`: Retrieves all historical rates for a given zip code.
        *   `getLatestRate(zipCode)`: Gets the most recent entry for a zip code.
    *   Implement Export/Import handlers (UI + logic):
        *   `exportDatabase()`: Serializes Dexie data into a JSON file for download; must exclude the Gemini API key by default.
        *   `importDatabase(json, options)`: Validates imported JSON and either `merge` (skip duplicates) or `replace` (clear and import) based on user choice.
        *   Add import preview/validation UI in a Settings modal with clear success/failure reporting.

4.  **Build Serverless Proxy (`/functions/fetchRates.js`)**:
    *   Scaffold a lightweight Node.js serverless function (Netlify or Vercel) that accepts zip code queries.
    *   Use `node-fetch` (or native fetch) to retrieve HTML from target domains and `cheerio` to parse lender and rate data.
    *   Normalize responses into a consistent JSON shape and include metadata such as source, scrape timestamp, and fallback reason (if applicable).
    *   Add basic rate limiting, caching, and user-agent rotation to reduce the chance of being blocked.
    *   Document environment variables (e.g., site URLs) and local testing instructions.

## Phase 3: Core Feature Development (Est. Time: 4-6 days)

This phase involves building the main user-facing features.

1.  **User Input & Control Flow (`main.js`)**:
    *   Develop the main application logic.
    *   On first load, render national average data from the bundled dataset so the dashboard is populated without user input.
    *   Persist the user's last successful zip code lookup (via `localStorage`) and automatically reload that dataset on return visits before fetching updates.
    *   On new user input (e.g., entering a zip code and clicking "Search"), trigger the data fetching process.
    *   Check IndexedDB first. If recent data exists, display it. If not, call the scraping module, store the new data, and then display it.

2.  **Develop the Mortgage Calculator (`assets/js/calculator.js`)**:
    *   Create a module for all mortgage-related calculations.
    *   Implement functions based on the formulas in `AFFORDABILITY_MODEL.md`:
        *   `calculateMonthlyPayment(price, downPayment, rate, term)`
        *   `calculateTotalCost(monthlyPayment, term)`
        *   `calculateBreakEvenPoint(refiCosts, monthlySavings)`
    *   Integrate this module with the UI inputs (sliders/text fields).

3.  **Implement Data Visualization (`assets/js/charts.js`)**:
    *   Create a module to manage Chart.js instances.
    *   Develop a function to generate a line chart for historical rate trends.
    *   Develop a function to create a bar chart comparing rates from different lenders.
    *   Ensure charts are responsive and update dynamically when new data is loaded.

## Phase 4: Advanced Features & UI Polish (Est. Time: 3-4 days)

This phase adds the final layers of functionality and refines the user experience.

1.  **Develop Affordability & Ranking System**:
    *   Implement the affordability logic from `AFFORDABILITY_MODEL.md`.
    *   Create a UI component that displays the affordability score and a clear explanation (e.g., "This home price is within your budget based on the 28/36 rule").
    *   Rank lenders based on a composite score (rate, fees, etc.) and display results in a clean, sortable table.

2.  **Integrate Gemini AI (`assets/js/gemini.js`)**:
    *   Create a small client-side module to interact with the Gemini API (model: `gemini-2.5-flash-latest`). This module will accept a user-provided key and make requests directly from the browser.
    *   Add a secure key-entry UI in Settings that lets users paste their Gemini API key. Default to session-only storage (in-memory) and offer an explicit toggle to persist the key to `localStorage` with a clear security warning.
    *   The client-side Gemini module should gracefully handle missing or invalid keys, provide clear error messages, and let the user remove or rotate keys via the settings menu.
    *   Add a "Get AI Analysis" button to the UI. When clicked, the client gathers the current context (rates, user inputs) and calls Gemini directly from the browser; display the response in the recommendations panel.
    *   Ensure there's a clear loading indicator, rate-limiting UI feedback, and robust error handling. Document the security tradeoffs of client-side keys in the README.
    *   Settings modal (UX):
        *   Provide controls for the Gemini API key (paste, save, clear) and a storage preference toggle (Session / Persist).
        *   Provide Export / Import controls in the same settings modal with `Export` (download) and `Import` (file chooser) actions and options to `Merge` or `Replace`.

3.  **UI/UX Refinement**:
    *   Apply styling using Tailwind CSS to match the wireframe in `UI_UX_WIREFRAME.md`.
    *   Ensure the layout is fully responsive and looks great on mobile devices.
    *   Add smooth transitions, loading spinners, and clear user feedback.
    *   Add disclaimers about data accuracy and financial advice.

## Phase 5: Testing & Deployment (Est. Time: 2-3 days)

The final phase is for ensuring quality and deploying the application.

1.  **Testing**:
    *   **Manual Testing**: Test all user flows on major browsers (Chrome, Firefox, Safari) and on different screen sizes.
    *   **Data Validation**: Test with various zip codes, including ones with no local data, to ensure fallbacks work correctly.
    *   **Calculator Validation**: Cross-reference calculator results with online mortgage calculators to ensure accuracy.

2.  **Deployment**:
    *   Confirm all CDN references load over HTTPS and the `data/national-averages.json` file reflects the latest manual refresh.
    *   Deploy the static files to GitHub Pages.
    *   Confirm the single-file `index.html` loads correctly from the deployed origin and that CDN resources are reachable over HTTPS.
    *   Set up the serverless function/proxy for the scraper on a platform like Netlify or Vercel and connect it to the deployed app.
    *   Perform final testing on the live site.

## Constraints & Risks (cross-phase)

- **No-build single-file constraint**: All code must run without compilation. This simplifies deployment but restricts library choices and may increase the size of `index.html`. Use CDNs and comment-driven organization to mitigate inspectability and maintainability.
- **Client-side Gemini keys**: Storing API keys in the browser (even optionally) is a security risk. Default to session-only storage and make explicit the risk in the UI and README.
- **Scraping legal risk**: Many aggregator sites disallow automated scraping. Use the proxy only for scraping where legally permissible and prefer authoritative APIs (Freddie Mac, FRED) for baseline and historical data.
