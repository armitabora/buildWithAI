---
doc: scope
status: approved
---

# Red Flag Contract Scanner

A pre-signing sanity check that helps renters spot risky clauses in residential apartment leases without receiving a generic contract summary.

## The Unique Kernel
It focuses on the traps a renter might miss before signing and lists those clauses, rather than summarizing the lease. The distinctive moment is seeing a concrete, high-stakes clause called out with a severity warning.

## Who It's For
Young professionals, college students, and everyday renters who are about to sign an apartment lease and feel overwhelmed by dense legal language. They want a quick sanity check before signing or clicking "I Agree."

## The Core Loop
The renter pastes lease text into the web app and clicks Scan. The app returns a clean list of specific red flags with severity labels, skipping generic summarization.

## Inspiration & Identity
A simple, focused web interface. More specific visual direction is not established.

## Why This Matters to the Learner
The learner wants to build the Red Flag Scanner for the hackathon and practice a structured, plan-first workflow with AI.

## What "Working" Looks Like
A one-minute demo shows a user pasting a dense, realistic 10-page apartment lease and receiving results within seconds. The results include a High severity warning about keeping a deposit for normal wear and tear and a Medium severity warning about a 60-day auto-renewal clause, without a generic summary.

## The POC Boundary
One web app for pasted residential apartment lease text. It scans for concrete renter-risk clauses and presents the findings as a clean list with High or Medium severity. The demo should prove that it can surface specific traps from a realistic lease.

## Later
Other contract types may be considered after the apartment lease proof of concept.

## Explicitly Cut
- Terms of Service, gym memberships, and freelance contracts: excluded to keep the proof of concept focused and the lease analysis as accurate as possible.