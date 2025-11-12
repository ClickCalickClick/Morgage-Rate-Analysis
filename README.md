# Mortgage Rate Analysis Tool - README

## Project Goal

A one-page, client-side website to track, compare, and analyze mortgage and refinance rates for any US zip code. The tool will provide graphical representations of historical data, an affordability calculator, lender comparisons, and optional AI-powered recommendations to help users make informed financial decisions.

This project is designed to be simple, cheap to run, and easily deployable.

## Core Features

*   **Zip Code-Based Rate Tracking**: Fetches current mortgage and refinance rates based on a user's zip code.
*   **Historical Data Visualization**: Displays rate trends over time in an interactive line chart.
*   **Lender Comparison**: Ranks and compares different lenders based on rates and estimated fees.
*   **Affordability Calculator**: Helps users determine what they can afford based on their income, home price, and down payment, using established financial rules.
*   **Mortgage Calculator**: Calculates monthly payments, total cost of loan, and refinance break-even points.
*   **Local-First Data Storage**: Uses IndexedDB in the browser to store historical data, making subsequent visits for the same zip code faster.
*   **National Baseline Snapshot**: Ships with a static `data/national-averages.json` so first-time visitors see current national trends without waiting for a lookup; the file is refreshed manually as needed.
*   **Optional AI-Powered Insights**: Integrates with Gemini AI (`gemini-2.5-flash-latest`) to provide deeper analysis, market context, and personalized recommendations on demand. Gemini calls are made directly from the browser using a user-provided API key.

## Tech Stack

*   **Frontend**: HTML5, CSS3, Vanilla JavaScript (ES6+)
*   **Styling**: [Tailwind CSS](https://tailwindcss.com/) loaded via the official CDN snippet to provide utility-first, responsive design without a build step.
*   **Charting**: [Chart.js](https://www.chartjs.org/) for creating beautiful and interactive charts.
*   **Local Database**: [IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API) via the [Dexie.js](https://dexie.org/) wrapper for simplified database operations.
*   **Calculations**: [Math.js](https://mathjs.org/) or custom JavaScript functions for financial calculations.
*   **AI Integration**: [Google AI SDK](https://ai.google.dev/docs) for connecting to the Gemini API (model: `gemini-2.5-flash-latest`).
*   **Deployment**: Static site hosting compatible with GitHub Pages, Netlify, Vercel, or can be run locally by opening `index.html`.
*   **Build Process**: None required; everything runs as plain HTML/JS with CDN-delivered dependencies.

## Gemini key storage and security

By default the app will operate in a client-only mode: users paste their Gemini API key into Settings. The app will default to keeping the key in memory for the session; an explicit "Store key locally" option will persist the key to `localStorage` only if the user accepts a security warning. The app will provide a menu to change or remove the stored key. Do not store sensitive keys unless you understand the risks.

---
*This is a planning document. The final implementation may vary.*
