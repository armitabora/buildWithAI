---
doc: checklist
status: approved
---

# Build Checklist

Build mode: fast

## Slices

- [ ] **1. Scan a synthetic apartment lease end to end**
  Becomes usable: A local Streamlit app where a judge can load the sample lease, scan it, and see validated High/Medium findings with exact snippets in styled result cards.
  Why now: Delivers the unique kernel and complete demo flow in the first usable slice; this small, single-screen POC has no separate infrastructure to build first.
  PRD ref: `prd.md > The Core Journey`, `prd.md > Screens and Layout`, `prd.md > Lease Scanning and Results`, `prd.md > Editing and Rescanning`, `prd.md > States and Boundaries`.
  Spec ref: `spec.md > Stack`, `spec.md > Where It Runs and How Someone Tries It`, `spec.md > Look and Feel`, `spec.md > Components`, `spec.md > Data Model`, `spec.md > External Services and Dependencies`, `spec.md > Important Failure Modes`.
  Build: Generate only `requirements.txt`, `.streamlit/config.toml`, and `app.py`. Implement the single-page UI, synthetic 150+ word sample, session callbacks, 100-word cutoff, Gemini structured-output call, strict Pydantic validation, escaped custom cards, and the specified empty, no-findings, invalid-lease, and retry states. Read the API key from the ignored local `.env`; do not add `.env.example`.
  Verify (mechanical): Install with `python -m pip install -r requirements.txt`; run `python -m py_compile app.py`; start `streamlit run app.py` and verify the blank-input, short-input, sample-load, edit-clears-results, and missing-key paths. With a locally configured key, scan the synthetic sample and verify the schema, High/Medium findings, and exact snippets.
  Learner check: Open the local app, load the sample, scan it with your local API key configured, and confirm the findings look convincing and the snippets match the sample text.
  Commit: `Build lease scanner proof of concept`

## Hands-on Checkpoints

- [ ] Early usable behavior explored — slice 1; use the same hands-on session for the final review.
- [ ] Final kick-the-tires exploration and feedback completed — same session as the early check; do not duplicate the session.

## Final Review

- [ ] Review how the model labels the synthetic 60-day auto-renewal; the current prompt may not enforce the PRD's MEDIUM default.
- [ ] Final review complete — feedback resolved and learner confirms ready to ship

## Code Tour and App Map

- [ ] Learning activity complete — guided route, focused alternative, prior practice connected, or brief recap
- [ ] Optional edit and transfer reflection addressed — offered/declined/already covered/not applicable as appropriate
- [ ] `devpost/app-map.html` generated from finished code, checked, and shown, including a project-grounded practice to reuse

Activity and evidence: [what actually happened; real document/test/code references; unfinished work if interrupted]
Route and stops: [actual paths and symbols; guided stops completed, or reference-only route]
Edit outcome: [tried/kept/reverted/declined/not applicable; verification if changed]
Reflection: [offered/answered/declined/already covered — personal answer belongs only in the ignored profile]
Activity mode: [live app and editor, explicit static fallback, focused alternative, prior practice, or recap]

## Revisions
- Clarified the Gemini instruction so a 60-day auto-renewal is MEDIUM by default, matching `prd.md > Lease Scanning and Results`; verified against the synthetic sample, which returned two HIGH and two MEDIUM findings.
- Reverted that prompt clarification at the learner's request; the 60-day auto-renewal may again be labeled HIGH despite the PRD's MEDIUM guidance.

- Gemini rejected Pydantic's generated `additionalProperties: false` schema field — remove the unsupported field while retaining typed Pydantic validation and exact-snippet checking.