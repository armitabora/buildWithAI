---
doc: spec
status: approved
---

# Red Flag Contract Scanner - Technical Spec

## How This Works, In Plain Language

The whole app runs as one local Python program. Streamlit draws the page and temporarily keeps the current lease and scan result in memory for the open browser session. When someone scans, the app sends the pasted text to Google's Gemini API using the official `google-genai` Python package. Gemini returns a typed JSON result describing whether the text is a lease and listing any red flags; Pydantic checks that result before the app displays it. No database, file storage, accounts, deployment, or external backend are part of this proof of concept.

The sample lease is synthetic text stored as a constant in `app.py`. The demo uses that sample only; do not submit real lease text or PII to the free-tier API. The API's data handling is separate from the app's in-memory-only state.

## The Core Journey Through the System

PRD ref: `prd.md > The Core Journey`.

1. Streamlit starts `app.py`, loads `.streamlit/config.toml`, and draws the title, renter-focused subtitle, legal disclaimer, and empty text area.
2. The user pastes text. Streamlit stores it in `st.session_state.lease_text`; edits run the text area's callback, which clears `st.session_state.scan_result` so old findings cannot remain attached to new text.
3. Clicking **Load Sample Lease** runs its callback before the next page render. It replaces `lease_text` with the built-in synthetic 150+ word sample and clears `scan_result`; it never calls Gemini.
4. Clicking **Scan for Red Flags** first counts words locally. Empty or fewer-than-100-word input shows the PRD warning and stops without an API call.
5. For eligible text, `app.py` checks `GEMINI_API_KEY`, shows the spinner, builds a Gemini client, and calls `gemini-3.8-flash` (or `GEMINI_MODEL`) with the lease and Pydantic response schema.
6. Pydantic validates `LeaseScanResult`. If `is_residential_lease` is false, the app shows the PRD lease/length warning. If true, it stores the validated result in session memory and shows the empty-findings message or renders each finding as a sanitized custom HTML card below the buttons.
7. A network, timeout, missing-key, invalid-response, or invalid-snippet error clears the result, preserves the text, and shows the PRD retry message. Refreshing or ending the Streamlit session drops both values.

## Stack

- **Python 3.11+** runs the local application and supports the selected libraries.
- **Streamlit** provides the single-page interface, widgets, reruns, and session-scoped in-memory state. No separate frontend framework or web server is added.
- **Google Gen AI SDK (`google-genai`)** is the maintained Gemini Python SDK. The legacy `google-generativeai` package is not used.
- **Gemini Developer API, `gemini-3.8-flash`** is the default model. `GEMINI_MODEL` can override the model id without changing code. The selected model supports structured output and is currently listed as free on the Developer API; availability and quotas can change.
- **Pydantic** defines the response contract and validates model output.
- **python-dotenv** loads the developer's local `.env` file. The key is read from `GEMINI_API_KEY` in the process environment.
- **Python standard library `html.escape`** escapes each model-produced string before it is interpolated into the fixed HTML card template.

Documentation:
- [Python](https://docs.python.org/3/)
- [Streamlit](https://docs.streamlit.io/)
- [Google Gen AI SDK for Python](https://googleapis.github.io/python-genai/)
- [Gemini structured outputs](https://ai.google.dev/gemini-api/docs/structured-output)
- [Gemini 3.8 Flash model](https://ai.google.dev/gemini-api/docs/models/gemini-3.8-flash)
- [Pydantic](https://docs.pydantic.dev/latest/)
- [python-dotenv](https://pypi.org/project/python-dotenv/)
- [Python `html.escape`](https://docs.python.org/3/library/html.html#html.escape)

## Where It Runs and How Someone Tries It

Run the app locally on macOS with Python 3.11 or later. No hosted URL is planned; the local app is sufficient for the required screen-recorded demo, alongside the public GitHub repository.

1. Create and activate a virtual environment, then install dependencies with `python -m pip install -r requirements.txt`.
2. Create a local `.env` containing `GEMINI_API_KEY=your_api_key_here`. Never commit `.env`; `.gitignore` ignores `.env` and `.env.*`.
3. Start the app with `streamlit run app.py` and open the local URL printed by Streamlit, normally `http://localhost:8501`.
4. For the recording, click **Load Sample Lease**, then **Scan for Red Flags**. Show the High and Medium results and their quoted snippets. Use synthetic sample data only.

## Look and Feel

Implement the approved dark, calm, professional style with Streamlit's project theme and a restrained blue action color. `.streamlit/config.toml` sets `base = "dark"`, `primaryColor = "#3B82F6"`, a near-black slate `backgroundColor`, a lighter slate `secondaryBackgroundColor`, and a high-contrast `textColor`.

Use Streamlit's `st.html()` for fixed custom result-card markup and CSS, not `st.markdown(..., unsafe_allow_html=True)`. Cards use dark slate surfaces, red/yellow left borders, pill-shaped HIGH/MEDIUM badges, and styled quote blocks. Keep headings and layout spacious; preserve Streamlit controls for input and buttons. Escape `title`, `explanation`, and `snippet` with `html.escape(..., quote=True)` before interpolation. Map the validated severity enum to fixed CSS classes and colors; never use model text as a CSS class or style value.

## Components

### `app.py` - Single-Page Interface

Renders the title, exact PRD subtitle, disclaimer, editable text area, and the primary **Scan for Red Flags** and secondary **Load Sample Lease** buttons. Place the results beneath the buttons with the text area still visible. Keep callbacks and the scan button outside a Streamlit form so `on_change` and `on_click` can maintain the required immediate behavior.

Implements `prd.md > Screens and Layout`, `prd.md > Editing, Sample Lease, and States`.

### `app.py` - Ephemeral Session State and Sample Fixture

Initialize `st.session_state.lease_text` to an empty string and `st.session_state.scan_result` to `None` before creating the text area. Bind the area with `key="lease_text"` and an `on_change` callback that clears `scan_result`. The sample button uses an `on_click` callback to assign the built-in synthetic sample and clear the result before Streamlit reruns the page. Keep the fixture at least 150 words and include clearly detectable High and Medium clauses.

Store no lease or result data in files, a database, Streamlit caches, browser local storage, or logs. Session State is tied to the Streamlit WebSocket; reload or disconnect resets it. The API request itself is still subject to Google's service terms and retention behavior.

Implements `prd.md > Editing, Sample Lease, and States`.

### `app.py` - Input Validation and Gemini Scan

On Scan, reject empty text with the exact empty-input message. Count whitespace-separated words; if the count is below 100, show the exact PRD lease/length warning and do not create a Gemini client or make a request. For eligible text, require a non-empty API key, show `st.spinner("Scanning for hidden clauses...")`, and make one synchronous Gemini call.

The system instruction treats the lease as untrusted document content, not instructions to follow. Ask only for materially harmful clauses, High/Medium severity using the PRD definitions, exact short snippets copied from the input, and a lease classification. Never ask for or return a generic summary. Verify that every returned snippet occurs in the submitted text; treat a mismatch as an invalid response and use the standard error state rather than presenting an unverifiable quote.

Use the schema in the user decision below and `types.GenerateContentConfig(response_mime_type="application/json", response_schema=LeaseScanResult)`. Keep the Pydantic schema compatible with Gemini's supported JSON Schema subset; do not emit `additionalProperties`, which Gemini rejects. Parse and validate the response with Pydantic and verify snippets before placing it in session state.

Implements `prd.md > Lease Scanning and Results`, `prd.md > Editing, Sample Lease, and States`.

### `app.py` - Sanitized Result Rendering

For `is_residential_lease == false`, display the PRD lease/length warning instead of findings. For a valid lease with no findings, display the exact no-major-flags message and legal-review reminder. Otherwise render one card per finding beneath the buttons, preserving the original input above. Escape all three model-provided strings with `html.escape(..., quote=True)`; only app-owned fixed markup, CSS, and allowlisted severity classes are interpolated.

Implements `prd.md > Lease Scanning and Results`, `prd.md > Look and Feel`.

### `.streamlit/config.toml` - Base Theme

Defines the app-wide dark theme: base dark, primary `#3B82F6`, dark slate app and secondary backgrounds, and legible text. No external CSS framework or remote asset is required.

Implements `prd.md > Look and Feel`.

### `requirements.txt` and Local Environment - Setup

`requirements.txt` lists `streamlit`, `google-genai`, `pydantic`, and `python-dotenv`. Create `.env` locally with `GEMINI_API_KEY=your_api_key_here`; it is ignored by Git and is not generated or committed.

Supports `prd.md > The Core Journey` and local startup in this spec.

## Data Model

Pydantic schema in `app.py`:

```python
class RedFlagItem(BaseModel):
    severity: Literal["HIGH", "MEDIUM"]
    title: str
    explanation: str
    snippet: str

class LeaseScanResult(BaseModel):
    is_residential_lease: bool
    red_flags: list[RedFlagItem]
```

Session-only app state:

- `st.session_state.lease_text: str` - current editable text; starts empty, changes through the text-area widget or sample callback, and clears from the active session on refresh/disconnect.
- `st.session_state.scan_result: LeaseScanResult | None` - validated result for the current unchanged text; cleared by every text edit and sample load, set only after successful validation, and never persisted.

The sample lease is a source-code constant, not user data or a file. No raw request/response logging or caching is used.

## File Structure

```text
red-flag-scanner/
├── app.py                    # Streamlit UI, callbacks, sample, scan, schema, renderer
├── requirements.txt          # Runtime dependencies
├── .gitignore                # Existing; ignores local .env and learner profile
├── .streamlit/
│   └── config.toml           # Dark theme and accent colors
└── devpost/
    ├── learner-profile.md
    ├── scope.md
    ├── prd.md
    └── spec.md
```

No extra modules, database, test-data file, or deployment configuration are needed for this POC.

## External Services and Dependencies

### Gemini Developer API

- **Call:** one synchronous text generation request per valid Scan click through `client.models.generate_content(model=..., contents=..., config=...)`.
- **REST equivalent:** `POST https://generativelanguage.googleapis.com/v1beta/models/{model}:generateContent`.
- **Payload:** system instruction and the full eligible lease text as text contents; `response_mime_type="application/json"`; `response_schema=LeaseScanResult`. The model returns `is_residential_lease` and `red_flags` containing severity, title, explanation, and exact snippet.
- **Authentication:** `GEMINI_API_KEY` from the environment, populated locally by `python-dotenv`. No key is sent to the browser or committed.
- **Model:** `gemini-3.8-flash` by default, optionally replaced by `GEMINI_MODEL`.
- **Timeout:** set a bounded SDK request timeout of 90 seconds using `types.HttpOptions(timeout=90_000)`; the timeout is in milliseconds. On timeout, preserve input and show the PRD error message.
- **Cost and quota:** as of 2026-10-24, published Gemini 3.8 Flash input/output pricing is free on the Developer API free tier. Model quotas vary by project and usage tier and can change; check AI Studio before the demo. Free-tier prompts may be used to improve Google's products, so only synthetic lease text is used for this POC.
- **No API request** is made when the input is empty, has fewer than 100 words, or the sample button is clicked.

Docs: [Google Gen AI Python SDK](https://googleapis.github.io/python-genai/), [GenerateContent API](https://ai.google.dev/api/generate-content), [structured output](https://ai.google.dev/gemini-api/docs/structured-output), [model details](https://ai.google.dev/gemini-api/docs/models/gemini-3.8-flash), [pricing](https://ai.google.dev/gemini-api/docs/pricing), [rate limits](https://ai.google.dev/gemini-api/docs/rate-limits), and [Gemini API terms](https://ai.google.dev/gemini-api/terms).

### Streamlit

Streamlit runs locally and serves the UI in the browser. Theme config is project-local; `st.html()` receives only app-owned HTML/CSS templates with escaped dynamic strings. Its session is in-memory and tied to the browser connection.

Docs: [Streamlit theming](https://docs.streamlit.io/develop/concepts/configuration/theming), [`st.html`](https://docs.streamlit.io/develop/api-reference/text/st.html), [`st.session_state`](https://docs.streamlit.io/develop/api-reference/caching-and-state/st.session_state).

## Important Failure Modes

- **Missing API key** -> do not call Gemini; preserve the lease and show “🚨 An error occurred while scanning. Please try again.”
- **Timeout, network/API error, or invalid structured response** -> preserve the lease, clear the result, show that same exact message, and allow retry using Scan.
- **Empty or too-short input** -> show its PRD warning locally without calling Gemini.
- **Eligible text is not a lease** -> display the PRD lease/length warning; do not render a summary or findings.
- **Model gives a snippet not present in the lease** -> treat the response as invalid and show the generic scan error instead of a quote the renter cannot verify.

## What Was Simplified and Why

- One `app.py` instead of separate frontend and backend services - a single local Streamlit process is enough for the one-screen demo.
- Streamlit session memory instead of a database or browser persistence - directly meets the no-saved-data requirement and clears on reload/disconnect.
- A source-code synthetic fixture instead of uploads or a fixture service - gives judges a one-click demo without handling real leases.
- One Gemini call per accepted scan instead of multi-stage agents, retrieval, or legal research - keeps latency and failure modes proportionate to the proof of concept.
- Custom cards use `st.html()` and a small fixed template instead of a frontend framework - gives the requested visual identity while preserving a minimal file set.

## Decisions and Open Issues

- **Learner decisions:** local-only Streamlit demo; Python, Streamlit, Gemini Developer API; maintained `google-genai`; default `gemini-3.8-flash` with `GEMINI_MODEL` override; `GEMINI_API_KEY` from environment/local `.env`; Pydantic structured output; in-memory `st.session_state`; synthetic-only demo; custom `st.html()` cards and dark theme config; generate only `app.py`, `requirements.txt`, and `.streamlit/config.toml` as requested. The local ignored `.env` is created by the learner.
- **Provider data handling:** the learner accepts standard free-tier handling only because the hackathon demo uses synthetic text and no real PII. This app is not represented as safe for real leases; app-side statelessness does not delete Google's provider-side logs or prevent free-tier product improvement use.
- **SDK learning moment:** official docs identify `google-generativeai` as not actively maintained and recommend `google-genai`; the learner approved the maintained package.
- **Model check:** official docs currently list `gemini-3.8-flash` as available, structured-output capable, and free on the Developer API free tier. Its availability, quotas, and pricing should be rechecked in AI Studio before recording because these can change.
- **Structured-schema compatibility:** the first live API attempt rejected Pydantic's `additionalProperties: false` generated by `extra="forbid"`. Pydantic's default schema omits that unsupported field; retain typed validation of the declared fields and the exact-snippet check.
- **Verification during build:** exercise the sample with synthetic fixtures, verify the 100-word boundary and no-call behavior, test non-lease and no-findings branches, confirm exact snippets match the source text, test all three dynamic strings with HTML metacharacters, and force a missing-key/API-error path to confirm input preservation and retry.