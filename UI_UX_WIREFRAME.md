# UI/UX Wireframe & User Flow

This document provides a textual wireframe and describes the user flow for the one-page application.

## I. Page Layout (Wireframe)

This describes the visual structure of the `index.html` page from top to bottom.

---

### **1. Header (`<header>`)**
*   **Left**: Logo and Title (e.g., "Mortgage Rate Analyzer")
*   **Right**: Navigation Links (e.g., "Calculator", "Charts", "About") that smoothly scroll to the relevant sections on the page.
*   **Far Right / Settings**: A small gear icon that opens a Settings panel for key management, export/import, and preferences.

---

### **2. Main Content (`<main>`)**

#### **Section A: User Input Panel (`<section id="input">`)**
*   A clean, card-like container.
*   **Title**: "Start Your Analysis"
*   **Input Fields**:
    1.  `Zip Code`: Text input, required.
    2.  `Home Price`: Text input with a slider, defaults to a national average (e.g., $400,000).
    3.  `Down Payment`: Text input with a slider, can be toggled between $ and %.
    4.  `Gross Annual Income`: Text input, optional, with a note explaining it's for the affordability calculation.
*   **Primary Button**: "Analyze Rates" - This button triggers the main application logic.

---

#### **Section B: Results Dashboard (`<section id="dashboard">`)**
*   On a brand-new visit, this section displays national average trends so the user immediately sees an example dashboard.
*   When a returning user has a stored zip code, preload that dataset before fetching fresh data.
*   If no data is available yet, show a placeholder message like "Enter your details above to see results."

*   **Sub-Section B1: Summary & Affordability**
    *   **Left Side**: A summary card showing the best rate found (e.g., "Best Rate: 6.75%").
    *   **Right Side**: The Affordability Scorecard (e.g., "✅ Affordable" with a brief explanation based on the 28/36 rule).

*   **Sub-Section B2: Historical Rate Chart (`<canvas id="rateChart">`)**
    *   **Title**: "Rate Trends for [Zip Code]"
    *   A Chart.js line chart showing mortgage rates over time.
    *   **Controls**: Buttons/dropdown to toggle between "30-Year Fixed", "15-Year Fixed", "Refinance", etc.

*   **Sub-Section B3: Lender Comparison Table**
    *   **Title**: "Today's Rates from Local Lenders"
    *   A sortable table with columns: `Lender Name`, `Rate (%)`, `APR (%)`, `Est. Monthly Payment`.
    *   Each row could be expandable to show more details.

---

#### **Section C: Mortgage Calculator (`<section id="calculator">`)**
*   **Title**: "Explore Your Mortgage"
*   An interactive calculator that is pre-filled with data from the user input panel but can be adjusted independently.
*   **Inputs**: Sliders for `Loan Amount`, `Interest Rate`, `Loan Term`.
*   **Outputs**: Dynamically updated display showing:
    *   `Monthly Payment (P&I)`
    *   `Total Cost of Loan`
    *   A pie chart visualizing Principal vs. Total Interest Paid.
*   Includes a toggle for a **Refinance Break-Even Calculator**.

---

#### **Section D: AI Recommendations (`<section id="ai-insights">`)**
*   **Title**: "AI-Powered Insights"
*   A container that initially shows a message like "Want a deeper analysis?"
*   **Button**: "Ask Gemini"
*   When clicked, a loading indicator appears.
*   The results from the Gemini API are then displayed in this container in a clean, formatted block.

---

### **3. Footer (`<footer>`)**
*   **Left**: "Data sourced from..." with links to the source websites.
*   **Center**: Disclaimer: "This tool is for informational purposes only and does not constitute financial advice."
*   **Right**: Copyright and/or link to your GitHub profile.

## Settings / Key Management (UI)

This is a modal or slide-over panel accessible from the header gear icon. It contains:

- **Gemini API Key**
    - Text field for pasting the Gemini API key.
    - Radio/toggle for storage preference: ( ) Session only (default)  ( ) Store locally (localStorage).
    - Explanatory text with a short security warning about local storage and XSS risk.
    - Buttons: `Save Key`, `Clear Key`.

- **Export / Import Data**
    - `Export Data` button that downloads the Dexie/IndexedDB dataset as a JSON file. The export does NOT include the Gemini key by default.
    - `Import Data` file input; after upload, show a preview and options: `Merge` (skip duplicates) or `Replace` (clear and import).
    - Validation messages and an import summary after completion.

- **Preferences**
    - Option to toggle whether the app preloads the last-used zip on startup.
    - Option to adjust data retention (e.g., prune historical data older than X months).

## Single-file code layout (documentation in `index.html`)

Because the app is a single HTML file with embedded JS, the file will be structured and commented to make sections easy to find. Suggested in-file section headers (comments) and contents:

- `<!-- UI MARKUP -->` — HTML for input panel, dashboard, charts, table, calculator, ai panel, settings modal, footer.
- `<!-- LIBRARY LOADS (CDN) -->` — CDN script tags for Tailwind, Chart.js, Dexie.js, Math.js.
- `<!-- CONFIG / CONSTANTS -->` — Defaults, national baseline path, API endpoints.
- `<!-- STORAGE / DB (Dexie) -->` — Dexie schema and helper functions.
- `<!-- SCRAPER API / FETCH LAYER -->` — Functions that call the serverless proxy for scraping (if proxy available) and fallback flows.
- `<!-- GEMINI CLIENT -->` — Client-side Gemini helper that reads key from session/localStorage, builds prompts, and sends requests.
- `<!-- CALCULATOR / AFFORDABILITY -->` — Mortgage formulas and utility functions.
- `<!-- CHARTS -->` — Chart.js setup and update functions.
- `<!-- EXPORT / IMPORT HANDLERS -->` — Export and import logic, validation, and UX hooks.
- `<!-- EVENT BINDINGS / APP BOOTSTRAP -->` — Main initialization logic: load national baseline, restore last zip, wire event listeners.

Each section will include a short header comment, and a table-of-contents at the top of the file will point to line numbers or anchors for quick navigation.

## II. User Flow

1.  **Landing**: User arrives on the page and sees the Header and User Input Panel.
2.  **Input**: User enters a zip code, and optionally adjusts the home price, down payment, and income.
3.  **Analysis**: User clicks "Analyze Rates".
    *   The app shows a loading spinner.
    *   It checks IndexedDB for recent data for that zip code.
    *   If no fresh data exists, it calls the serverless function to scrape for new data.
    *   The new data is saved to IndexedDB.
4.  **Display**: The loading spinner disappears, and the Results Dashboard populates with:
    *   The best rate and affordability score.
    *   The historical chart.
    *   The lender comparison table.
5.  **Interaction**:
    *   User can interact with the chart, sorting the table.
    *   User scrolls down to the Calculator, which is pre-filled, and can adjust values to see payments change in real-time.
6.  **AI Insight**: User clicks "Ask Gemini". The app sends the current context (rates, user inputs) to the Gemini API and displays the returned analysis.
