# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the App

No build process. Open `DocScan.html` directly in any modern browser (Chrome, Edge, Firefox). The app requires an Anthropic API key entered at runtime — it is stored in `localStorage` under the key `docscan_apikey`.

## Architecture

DocScan is a **single-file HTML5 application** (`DocScan.html`, ~775 lines) with all CSS and JavaScript embedded inline. There are no external dependencies beyond CDN-loaded libraries:

- **pdf.js 3.11.174** — renders PDF first pages to canvas for AI extraction
- **Anthropic Claude API** — `claude-sonnet-4-20250514` via direct `fetch` to `https://api.anthropic.com/v1/messages`

### Three UI Panels

| Panel | Purpose |
|---|---|
| Capture | File upload (PDF/image), AI extraction, editable form |
| Records | Table of saved entries, inline edit/delete |
| Export | Statistics + 4 export formats (CSV, JSON, vCard, SQL) |

### Data Flow

1. User drops a PDF or image onto the Capture panel
2. Images are resized to ≤1400px width and converted to base64 JPEG; PDFs are rendered via pdf.js to a canvas, then converted the same way
3. Base64 image is sent to Claude API with a structured extraction prompt requesting JSON with four fields: `empresa`, `morada`, `email`, `telefone`
4. Extracted JSON populates the form; user can edit before saving
5. Saved records go into an in-memory `records` array and are persisted to `localStorage` (`docscan_win` key)

### Key JavaScript Globals (all inline in `<script>`)

- `records` — array holding all saved documents
- `currentFileData` — base64 image string of the currently loaded file
- `extractData()` — sends the image to Claude API and parses the JSON response
- `saveRecord()` / `deleteRecord()` / `updateRecord()` — CRUD on `records` + localStorage
- `exportCSV()` / `exportJSON()` / `exportVCard()` / `exportSQL()` — export formatters

### Storage Keys

- `docscan_win` — JSON-serialized `records` array
- `docscan_apikey` — Anthropic API key

## Claude API Usage

The extraction call uses the Messages API with vision:

```
model: claude-sonnet-4-20250514
max_tokens: 400
anthropic-version: 2023-06-01
```

The prompt is in Portuguese and asks for JSON only. Response parsing strips markdown fences before `JSON.parse`.

## UI Language

All UI text and comments are in **Portuguese (PT)**. Keep new text in Portuguese when modifying visible labels, status messages, or prompts.
