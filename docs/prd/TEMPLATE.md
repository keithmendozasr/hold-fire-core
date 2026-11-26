# PRD-NNNN: Title

- **Status:** Draft | Accepted | Superseded by PRD-NNNN
- **Owner:** Name
- **Date:** YYYY-MM-DD
- **Related ADRs:** ADR-NNNN (or "none")

> Copy this file to `docs/prd/NNNN-short-slug.md`. A PRD stays open until its status is **Accepted**; do not build on a Draft without checking with the code owners. Delete this note and any guidance text in italics when you fill in the template.

## Summary

_One or two sentences: what is being built and for whom._

## Problem

_What user problem or opportunity does this address, and why now?_

## Goals

- _Outcome this PRD commits to._

## Non-goals

- _Something a reader might assume is in scope but is not._

## Users and scenarios

_Who uses this, and what do they do? Describe the key flows._

## Requirements

_Numbered so they can be referenced from ADRs, issues, and tests._

1. _The system must..._

## Product guardrails

_Confirm how this feature honors the guardrails in `CLAUDE.md`, or mark each "not applicable"._

- **Training aid, not legal advice:** _How the disclaimer stays visible, and how grades and feedback avoid reading as legal conclusions._
- **Third-party names and logos:** _Confirm none appear in the product._
- **User-submitted text:** _How it is fenced off from LLM prompts and never interpolated as instructions._
- **LLM output:** _The schema it is validated against, and the reject or retry behavior._

## Success metrics

_How will we know it worked? Prefer measurable signals._

## Open questions

- _Unresolved decision, with who needs to answer it._

## Out of scope for this release

_Deferred work, if any, and where it is tracked._
