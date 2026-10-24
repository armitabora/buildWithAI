---
doc: prd
status: approved
---

# Red Flag Contract Scanner - Product Requirements

A single-page web app that helps renters spot materially harmful clauses in residential apartment leases before signing, without producing a generic summary.
Source: `scope.md > The Unique Kernel`, `scope.md > Who It's For`, `scope.md > The Core Loop`.

## The Core Journey

1. The renter opens the single-page scanner and sees its purpose, the legal disclaimer, an empty lease text area, and the primary scan button.
2. The renter pastes the full residential lease. If the text has fewer than 100 words, the app warns them immediately and does not submit it for analysis.
3. For text that meets the minimum, the renter starts a scan. The app shows a loading indicator and keeps the lease visible in the input area.
4. If the text is a lease, the app displays only potential red flags below the scan button. Each result has a severity, short title, plain-English explanation, and exact quote from the lease. If no major red flags are found, the app displays the agreed reassuring message and legal-review reminder instead.
5. The renter compares each quoted snippet with the original lease. Editing the text clears all previous results; they must scan again to get results for the changed text.

Source: `scope.md > The Core Loop`, `scope.md > What "Working" Looks Like`, `scope.md > The POC Boundary`.

## Screens and Layout

One page, with no screen switching:

- At the top, a bold title: "🚩 Red Flag Contract Scanner".
- The subtitle reads: "For renters: Paste your residential apartment lease to spot predatory clauses like hidden fees or auto-renewals before you sign."
- A prominent disclaimer appears before the input: "Disclaimer: This tool provides AI-generated analysis and does not constitute formal legal advice."
- A large, spacious, editable text area accepts pasted lease text. The primary "Scan for Red Flags" button sits directly below it.
- A secondary "Load Sample Lease" button appears beside or above the scan button for a one-click demo.
- Scan results appear below the button while the text area remains visible above them, allowing renters to cross-reference each quote.

Source: `scope.md > The Core Loop`, `scope.md > The POC Boundary`.

## Look and Feel

Use a minimalist, spacious dark-mode aesthetic that feels trustworthy, calming, and professional, not like a cluttered dashboard. Use a calm blue accent for the primary action and clear red and yellow severity accents. The learner described the intended direction as a premium, secure legal tool; the interface should convey that feeling without making unverified security claims.

## Features and Behavior

### Lease Scanning and Results

- The supported document type is residential apartment leases pasted as text. File uploads and other contract types are outside the POC.
- Text under 100 words is rejected in the interface before an AI request is made, with: "⚠️ This text does not appear to be a residential lease, or it is too short to analyze. Please paste a full apartment lease."
- Text of 100 words or more is analyzed. The AI may still reject it if it determines the content is not a residential lease; show the same clear warning and do not present findings as lease analysis.
- The scanner looks for any clear clause that could materially harm a renter, including unusual or specific risks beyond the examples. It must not return a generic lease summary.
- During analysis, show an animated spinner and the message "Scanning for hidden clauses...".
- Show results directly below the scan button on the same page, leaving the original text area visible.
- Each finding is a distinct block or bullet with:
  - A High or Medium severity label.
  - A short issue title, such as "Forced Arbitration" or "Hidden Fee".
  - A one- or two-sentence plain-English explanation of why the clause is a trap.
  - The exact short quote from the pasted lease that triggered the finding, labeled "Snippet", so the renter can search for it in the original document.
- High severity means a risk of significant financial loss, waiver of fundamental legal rights, or unfair eviction. Examples include uncapped rent increases, forced arbitration, and landlord entry without notice.
- Medium severity means a highly restrictive or inconvenient term that does not immediately threaten the renter's housing or bank account. Examples include strict guest limits, 60-day auto-renewals, and responsibility for minor appliance maintenance.
- Do not show a Low severity category in this POC.
- If the scan finds no major red flags, display: "✅ No major red flags detected." Follow it with: "This lease appears standard based on our AI scan, but please read the document carefully before signing, as this does not replace professional legal review."

### Editing and Rescanning

- The renter can edit the lease text in place at all times.
- Any edit after a successful scan immediately clears the prior results so they cannot be confused with the changed text.
- To scan a replacement or revised lease, the renter deletes or replaces the text and selects "Scan for Red Flags" again. There is no separate Start Over or Clear button.
- Selecting "Load Sample Lease" replaces the input with a built-in, realistic, synthetic residential lease of at least 150 words. The sample includes at least one High and one Medium red-flag clause, contains no real personal information, clears any previous results, and does not start a scan. The renter must click "Scan for Red Flags" to analyze it.

Source: `scope.md > The Core Loop`, `scope.md > What "Working" Looks Like`, `scope.md > The POC Boundary`.

## States and Boundaries

- **First use / empty input** - Show the title, subtitle, disclaimer, empty text area, and scan button. Clicking Scan with no text immediately shows "⚠️ Please paste your lease text first!" and does not attempt analysis; the user remains on the same screen.
- **Too-short input** - Fewer than 100 words triggers the lease/length warning immediately in the interface, without an AI request.
- **Non-lease input** - Text that passes the word-count cutoff but is determined not to be a residential lease is rejected with the lease/length warning rather than analyzed as a lease.
- **Scanning** - Show the animated spinner and "Scanning for hidden clauses..." while processing.
- **Findings** - Keep the pasted lease visible and display the findings below the button.
- **No findings** - Show the agreed success message and legal-review reminder; do not imply that the lease is guaranteed safe.
- **Scan error** - On a network or API error, keep the pasted lease text, show "🚨 An error occurred while scanning. Please try again.", and allow retry with the same Scan button.
- **Edited lease** - Clear stale findings immediately when the input changes; a fresh scan is required for new findings.
- **Sample lease loaded** - Populate the editable input with the synthetic sample, clear previous results, and wait for the renter to start the scan.
- **Refresh or leave** - Do not save lease text or scan results. Both disappear when the user refreshes or leaves the app and are not available when they return.

Source: `scope.md > The Core Loop`, `scope.md > What "Working" Looks Like`, `scope.md > The POC Boundary`.

## Product Decisions

- Limit the POC to residential apartment leases to keep the analysis focused and accurate; Terms of Service, gym memberships, and freelance contracts are excluded (`scope.md > Explicitly Cut`).
- Use a firm 100-word frontend cutoff to provide immediate feedback and avoid unnecessary analysis requests.
- Let the AI reject longer text that is not a residential lease.
- Flag any clear clause that could materially harm a renter; the examples are guidance, not an exhaustive list.
- Use High and Medium severity with the learner's stated distinction and examples.
- Keep the input and results together on one page; clear results on edits to prevent stale findings from being mistaken for analysis of new text.
- Provide a one-click synthetic sample lease for the demo; loading it clears old results but never starts an AI request automatically.
- Use only synthetic lease text during the hackathon demo; no real PII is submitted to the Gemini free tier.
- Keep the app stateless with respect to lease content: do not save text or results, and discard both on refresh or exit.
- Display the agreed AI-analysis and legal-advice disclaimers prominently, including the additional caution when no major flags are found.

## What We're Building

A single-page apartment-lease scanner with pasted-text input, a 100-word frontend cutoff, lease validation, AI-generated High/Medium findings with exact source snippets, a no-findings result, loading and error feedback, retry, stale-result clearing, and no saved lease or result data.

## Deferred From the POC

- Support for Terms of Service, gym memberships, and freelance contracts is deferred to keep the first version focused on leases (`scope.md > Explicitly Cut`).
- Saved leases, accounts, and scan history are excluded because the learner wants lease text and results discarded rather than retained.
- A separate clear or start-over action is excluded because the renter can edit or delete the text directly in the input.

## Possible Later Enhancements

Other contract types may be considered after the apartment-lease proof of concept (`scope.md > Later`).

## Non-Goals

- Summarizing the lease: the app exists to identify and explain red flags, not produce a generic summary (`scope.md > The Unique Kernel`).
- Providing formal legal advice or guaranteeing that a lease is safe.
- Retaining lease text or scan results between visits.
- Scanning document types other than residential apartment leases in the POC.

## Open Questions

- Before the technical approach is finalized, verify how the selected AI provider handles submitted lease text, including retention and use. The app's stateless behavior does not by itself establish provider-side deletion; do not make a stronger privacy claim in the UI than verified provider behavior supports.