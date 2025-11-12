# Export / Import Data Guide

This document specifies how users can export and import the local dataset stored in IndexedDB (via Dexie.js). This enables moving data between machines or making backups.

## Goals
- Let users export their current dataset (rates, metadata, and app settings) to a single JSON file.
- Let users import a previously exported JSON file and merge or replace local data safely.
- Keep the import process robust against malformed or incompatible files.

## Export format (JSON)
The exported file will be a single JSON object with a small header and arrays of data.

Example structure:
```json
{
  "version": "1.0",
  "exportedAt": "2025-11-12T00:00:00Z",
  "app": {
    "lastZip": "90210",
    "settings": {
      "currency": "USD",
      "dateFormat": "iso"
    }
  },
  "rates": [
    {
      "zipCode": "90210",
      "date": "2025-11-12",
      "lender": "Example Bank",
      "loanType": "30-Year Fixed",
      "rate": 6.85,
      "apr": 6.95,
      "source": "zillow.com",
      "scrapeTimestamp": "2025-11-12T08:00:00Z"
    }
  ]
}
```

- `version`: schema version of the export (bump if format changes).
- `exportedAt`: timestamp of export.
- `app`: optional metadata (lastZip, UI prefs).
- `rates`: array containing rate records. Each record should match the local DB schema.

## Export UX
- Add an "Export Data" button in settings or the footer.
- When clicked:
  1. Query Dexie for the relevant tables (`rates`, maybe `lenders`/`history`).
  2. Build the export JSON object and serialize using `JSON.stringify(obj, null, 2)`.
  3. Trigger a browser download using a blob and `URL.createObjectURL` (filename like `mra-export-2025-11-12.json`).

Note: by default the export will not include any stored Gemini API keys or other sensitive secrets. If you wish to export keys, the UI must explicitly offer that option and show a strong confirmation/warning.

## Import UX
- Add an "Import Data" control that accepts a `.json` file.
- When a user uploads a file:
  1. Parse JSON and validate top-level fields: `version`, `exportedAt`, and `rates` array.
  2. Validate each `rate` record has required fields (zipCode, date, lender, rate). If any record fails validation, show a preview and allow the user to cancel.
  3. Offer the user a choice: `Merge` or `Replace`.
     - `Merge`: Insert new non-duplicate records. Define duplicates as same `zipCode` + `date` + `lender`. If duplicate exists, skip or offer "keep latest" based on `scrapeTimestamp`.
     - `Replace`: Clear relevant tables and insert from file.
  4. Use Dexie transactions for safe writes.
  5. Show a summary of imported vs skipped records.

## Validation rules (suggested)
- `zipCode` must be a 5-digit string or numeric that can be zero-padded.
- `date` must be ISO `YYYY-MM-DD` or a parseable date string.
- `rate` and `apr` must be numbers.
- `scrapeTimestamp` should be parseable as ISO; if missing, set to `exportedAt`.

## Security considerations
- Only parse files selected by the user (do not fetch remote files automatically).
- If imported JSON is large, process in chunks to avoid blocking the UI.

## Example import implementation notes (Dexie)
- Use `db.transaction('rw', db.rates, async () => { ... })`.
- For merging, use `bulkPut` with an `id` composed of `zipCode|date|lender` or check duplicates with `where` queries.

## Summary
- Export/import enables portability and recovery.
- Use a robust schema and strict validation.
- Provide a simple, clear UI that lets users choose `merge` or `replace` and shows a summary of results.