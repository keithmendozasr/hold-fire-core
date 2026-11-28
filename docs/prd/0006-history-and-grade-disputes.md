# PRD-0006: History and grade disputes

| Field | Value |
| --- | --- |
| Version | 0.1 |
| Date | 2026-11-28 |
| Author | Claude, requested by Keith Mendoza |
| Owner | Keith Mendoza |
| Status | Draft |
| Related ADRs | none yet (0001 proposes Database and data model) |

This feature PRD refines [PRD 0001](0001-product-vision.md) requirements 11 and 12 and its [Grade disputes](0001-product-vision.md#grade-disputes) section. Detail requirements keep 0001's numbers (11.1 is the first detail of requirement 11); details of grade disputes, which 0001 does not number, are GD.1, GD.2 and so on. Anything 0001 does not state is marked **(proposal)**. Build order: this PRD is built after Accounts and admin access ([0002](0002-accounts-and-admin-access.md)), Scenario authoring ([0003](0003-scenario-authoring.md)), Scenario run ([0004](0004-scenario-run.md)), Grading and results ([0005](0005-grading-and-results.md)) and test-grade in 0003, alongside Reporting ([0007](0007-reporting.md)).

## Summary

The trainee's list of past attempts and the detail of each one, and the beta's way to flag a grade as unfair, with an admin screen where Keith reviews every flag.

## Problem

Trainees learn from going back over what they missed, and repeats only help if each attempt is kept. Iteration 1 is a beta meant to prove grading quality, so a trainee who thinks a grade is wrong needs a quick way to say so, and Keith needs every such case in one place with what is needed to judge it.

## Goals

- A trainee can find any past attempt and reread their reasoning and the feedback.
- A trainee can flag any graded attempt as unfair, with an optional note, in a couple of taps.
- Keith sees every flag with the scenario, call, reasoning, score, feedback, prompt version and model ID, and can mark it reviewed.

## Non-goals

- A progression view (trends away from wrong calls). 0001 records wrong calls so this can be built later.
- Changing a score after review. Whether an admin can do this is open.
- Reports on flags beyond the per-scenario count, which is in [0007](0007-reporting.md).

## Users and scenarios

- **Trainee.** Opens history, scans their attempts, opens one to reread reasoning and feedback. From the result screen or a past attempt, flags the grade as unfair and adds a note saying why.
- **Admin (Keith).** Opens the flag review list, reads a flagged attempt with its prompt version and model ID, decides whether the grade was fair, and marks the flag reviewed with a note.

## Requirements

**History (0001 req 11 and 12)**

- 11.1 The trainee must see a list of their own past attempts with scenario title, date, call, score, and status (graded, "not graded", "no decision") **(proposal: status column)**.
- 11.2 Opening an attempt shows the trainee's reasoning and the feedback, against the scenario version it was graded on ([0003](0003-scenario-authoring.md) 16.7). Whether the image is shown again here is open.
- 11.3 A "not graded" attempt in history offers the retry from [0005](0005-grading-and-results.md) 10.3.
- 11.4 A trainee can see only their own attempts.
- 12.1 Each attempt at a repeated scenario is kept and listed separately.

**Grade disputes (0001 Grade disputes)**

- GD.1 From the result screen or a past attempt, the trainee can flag the grade as unfair and add an optional note saying why.
- GD.2 An attempt can be flagged once; the flag shows on the attempt afterwards **(proposal)**.
- GD.3 Flagged attempts appear in the admin console with the scenario, call, reasoning, score, feedback, prompt version and model ID. Reasoning and the note are shown under the reasoning-text permission, and each view is in the audit log ([0002](0002-accounts-and-admin-access.md) 17.4, 17.5, 19.2) **(proposal: 0001 lists reasoning on the flag screen and also puts it behind a logged permission; this applies both)**.
- GD.4 An admin can mark a flag reviewed, with a note. Reviewing is an admin action in the audit log **(proposal: 0001 says each flag is reviewed but not how that is recorded)**.
- GD.5 The number of flagged attempts per scenario is available to [0007](0007-reporting.md).

## Product guardrails

- **Training aid, not legal advice:** A past attempt shows the feedback with the same disclaimer as the result screen ([0005](0005-grading-and-results.md) 9.1).
- **Third-party names and logos:** Not applicable beyond the result screen rules.
- **User-submitted text:** Flag notes are untrusted text. They are stored and shown to admins only, and never sent to the grader or any other LLM **(proposal)**.
- **LLM output:** Not applicable; history shows stored, already validated results.

## Success metrics

- Disputes: the share of results flagged as unfair stays under 5%, and every flag is reviewed by Keith (0001 Success metrics).
- Every flag is marked reviewed within one week of being raised during the beta **(proposal)**.

## Test plan

Layers are those defined in [0001 Test plan](0001-product-vision.md#test-plan), plus **Security**: adversarial tests of Hold Fire's own rules **(proposal: run with the integration suite in CI)**. This plan refines 0001 T13. Admin-only access and audit entries for the flag review APIs, including the logged reasoning reads, are checked by adding them to [0002](0002-accounts-and-admin-access.md) T2 and T5.

| # | Item (requirements) | Layers | What each layer checks, and why there |
| --- | --- | --- | --- |
| T1 | Flag rules (GD.2, GD.4) | Unit | One flag per attempt, only on graded attempts, and the open to reviewed transition. Pure logic, tested once here. |
| T2 | History list and detail (11.1 to 11.3, 12.1) | Integration; Component | Integration: repeated attempts are stored and listed separately, and each attempt's detail returns the scenario version it was graded on. Component: a history row renders each status distinctly (graded, wrong call scored 0, "no decision", "not graded" with its retry). |
| T3 | Flagging and review list (GD.1, GD.3, GD.5) | Integration; Component | Integration: a flag with its note shows in the admin list with prompt version and model ID, and the per-scenario flag count includes it. Component: the flag control on the result and history views, with the optional note. |
| T4 | Ownership (11.4) | Security | A trainee cannot list, read or flag another trainee's attempt by ID. |

The main path in [0004](0004-scenario-run.md) T7 covers how the run, result and history screens are wired together, so this PRD adds no frontend E2E test. Escaping of flag notes on display is the frontend framework's job and is not tested here.

**Exploratory charters**

- Flag a grade from the result screen and from history on a phone, then review it as an admin (T3).
- Repeat a scenario three times and check history makes each attempt easy to tell apart (T2).

## Open questions

- **"No decision" in history.** How a "no decision" attempt is scored and shown in history, and whether it can be flagged (touches [0004](0004-scenario-run.md), [0005](0005-grading-and-results.md) and [0007](0007-reporting.md)). Owner: Keith.
- **Abandoned attempts.** Whether abandoned attempts appear in the trainee's history. Owner: Keith.
- **Score changes after review.** Whether an admin can change a score or regrade an attempt after reviewing a flag, and if so how that is recorded. Owner: Keith.
- **Telling the trainee.** Whether a trainee is told the outcome of a flag review. Owner: Keith.
- **Image in history.** Whether a past attempt shows the image again. Owner: Keith.
- **Retention.** How long past attempts are kept short of account deletion (from 0001). Owner: Keith, with attorney input.

## Out of scope for this release

A progression view. Trainer or agency views of students' history.

## Change log

| Version | Date | Summary |
| --- | --- | --- |
| 0.1 | 2026-11-28 | Initial draft. |
