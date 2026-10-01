# Finora

### AI-Assisted Financial Management Prototype

Finora is a single-file, browser-based prototype of a chat assistant for small-business finance workflows: cash-flow questions, receivable tracking, a GST-due overview, a demo credit-health indicator, and CSV export. It runs entirely in the browser with **synthetic demo data** and can optionally connect to an LLM provider using a key you supply.

> **This is a prototype, not a financial product.** See [Financial Disclaimer](#financial-disclaimer).

## Overview

The app presents a chat interface backed by a small in-memory business model (`DEMO_BIZ_DATA`). Questions are answered either by a built-in keyword-based mock responder (no API key needed) or by an LLM provider, which receives the demo business data as a system prompt. Actions such as "file GST" or "send reminders" are **simulated**: the UI asks for confirmation and then records a local, clearly labelled demo result. Nothing is sent to any external service other than the LLM request itself.

## Features

- **Chat assistant** in Hindi / Hinglish / English with quick-reply buttons and a row of demo prompts.
- **Provider selector**: Mock / Demo Mode, Claude (Anthropic), GPT (OpenAI), Gemini (Google).
- **Business snapshot** sidebar: bank balance, receivables, GST due, month-to-date profit.
- **Demo credit health**: a 0-100 score computed from four weighted factors (GST filing history 30%, overdue ratio 30%, bank balance vs. GST due 20%, profitability 20%) with illustrative loan ranges.
- **Demo loan application flow**: prepares an application summary locally after a confirmation step. Nothing is submitted anywhere.
- **Mock bill upload**: choosing an image triggers a simulated extraction that returns random sample values. The image is never read, stored, or uploaded, and no OCR is performed.
- **Receivable tracking**: add an invoice by chat (e.g. "Sample Buyer ne 50000 dena hai, note karo").
- **CSV export** ("CA Export") of the demo data, and a plain-text chat export.
- **Weekly digest**: posted in-app on Sundays while the page is open, or on demand via the sidebar button. It can be switched off.
- **Correction feedback**: "GALAT HAI" lets you record a correction that is included in later LLM prompts.
- **Demo support modal**: saves a note to local storage only. There is no support team or backend.
- **Local persistence** of chat, demo business state, and settings in `localStorage`.
- **Responsive layout** with a collapsible sidebar on small screens.

## Architecture

Everything lives in `finora.html`: markup, CSS, and vanilla JavaScript with no build step and no dependencies.

```
UI (HTML/CSS) ──> event handlers ──> state object ──> localStorage ("finora_v1")
                        │
                        ├─ mock responder (keyword rules, reads demo data)
                        └─ provider call (fetch) ──> Anthropic / OpenAI / Google
```

- **State**: one `state` object (messages, demo business data, provider, API keys, settings) persisted as JSON.
- **Response protocol**: the model (or mock) can emit `CONFIRM_ACTION:{json}`, `INVOICE_ADD:{json}` or `CSV_EXPORT`; `parseResponse()` turns these into confirmation buttons, an invoice prompt, or an export.
- **Rendering**: dynamic buttons use `data-*` attributes and one delegated click listener rather than inline handlers; all text is HTML-escaped before insertion.

## Tech Stack

- HTML5, CSS3, vanilla JavaScript (ES2020)
- Browser `localStorage` and `fetch`
- Google Fonts (Space Mono, DM Sans), loaded from Google's CDN; the app works without them using fallback fonts

## Project Structure

```
Finora/
├── finora.html     # the application (markup, styles, scripts)
├── README.md
└── .gitignore
```

## Setup

No installation or build step is required.

1. Open `finora.html` in a modern browser (double-click, or `File > Open`).
2. Or serve it locally: `python3 -m http.server 8000` and visit `http://localhost:8000/finora.html`.

The app starts in the Claude provider with no key saved. Choose **Mock / Demo Mode** in the sidebar to use it without any key.

## Configuration

- **Provider and key**: pick a provider in the sidebar, paste your own API key, and press **Save**. **Clear** removes it. The key is stored in this browser's `localStorage`, unencrypted.
- **Model IDs**: set in the `PROVIDER_MODELS` constant near the provider code. Update them if a provider retires a model.
- **Demo data**: edit `DEMO_BIZ_DATA` to change the sample business, parties and figures.
- **Storage key**: `STORAGE_KEY` (`finora_v1`). Changing it starts the app from clean defaults.

## Supported AI Providers

| Provider | Endpoint used | Notes |
| --- | --- | --- |
| Mock / Demo Mode | none | Keyword-based canned replies; no network access |
| Claude (Anthropic) | `api.anthropic.com/v1/messages` | Called directly from the browser with the `anthropic-dangerous-direct-browser-access` header |
| GPT (OpenAI) | `api.openai.com/v1/chat/completions` | Bearer token header |
| Gemini (Google) | `generativelanguage.googleapis.com` | Key sent in the `x-goog-api-key` header |

The provider integrations are thin `fetch` wrappers. They are exercised in testing only against intercepted (mocked) HTTP responses, not against live provider accounts, so verify them with your own key. They require your key to be entered in the browser and are not suitable for shared or production deployments (see [Security](#security)).

## Demo Data

All business data is synthetic and defined in `DEMO_BIZ_DATA`: a fictional company ("Demo Trading Co."), a placeholder GSTIN (`00AAAAA0000A0Z0`), invented parties, and round-number financial figures. None of it corresponds to a real business, person, or account. The mock bill upload returns random sample values from a small list of fictional vendors.

## Security

- **No credentials are included** in the repository. API keys are only ever entered by the user at runtime.
- **Keys live in `localStorage`** in plain text, and requests go from the browser straight to the provider. This is acceptable for a local prototype with a restricted/test key; it is **not** safe on shared machines or for production. A real deployment should proxy provider calls through a backend that holds the key.
- **No authentication or authorization** is implemented. There are no accounts, sessions or admin features.
- **Chat history and demo business state** are stored locally in the browser and sent to your chosen provider as part of LLM requests (when not in Mock mode).
- **Output handling**: model and user text is HTML-escaped before rendering, dynamic buttons avoid inline handlers, and CSV cells that begin with spreadsheet formula characters are neutralised on export.
- The page loads fonts from Google Fonts, which makes a request to Google when opened online.

## Financial Disclaimer

Finora is a prototype/demo for financial organisation and AI-assisted workflows. It is not a bank, a lender, a chartered accountant, a tax-filing service, an official credit bureau, or a financial advisor. The credit score is an illustrative calculation, not an official score; loan ranges are illustrative and are not a lender decision or a guarantee of eligibility. GST filing, payment reminders and loan applications are simulated and do not contact any external party. Do not rely on its output for financial, tax or legal decisions.

## Limitations

- Single-file prototype with no automated test suite or build tooling.
- Demo data only; there is no way to import real accounting data.
- The mock responder recognises a limited set of keywords and phrases.
- LLM replies depend on the model following the `CONFIRM_ACTION` / `INVOICE_ADD` output format; malformed output is ignored.
- Sidebar "Chat History" entries from earlier sessions are labels only and cannot be reopened.
- UI copy is primarily Hinglish; there is no localisation layer.
- No backend, so no real notifications, filings, messaging, support, or lender integration.

## Future Improvements

- Backend proxy for provider calls so keys never reach the browser
- Split the file into modules and add unit tests for the scoring and parsing logic
- Real data import (CSV) in place of the fixed demo dataset
- Restorable chat sessions
- Localisation / language switcher

## License

No license has been selected yet. Until a `LICENSE` file is added, all rights are reserved by default.
