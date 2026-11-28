# PRD-0005: Grading and results

| Field | Value |
| --- | --- |
| Version | 0.1 |
| Date | 2026-11-28 |
| Author | Claude, requested by Keith Mendoza |
| Owner | Keith Mendoza |
| Status | Draft |
| Related ADRs | none yet (0001 proposes Grading pipeline, and Portability and provider boundaries) |

This feature PRD refines [PRD 0001](0001-product-vision.md) requirements 7 to 10, 23 and 25 and the [Grading design](0001-product-vision.md#grading-design). It does not restate the grading design; it turns it into requirements. Detail requirements keep 0001's numbers (7.1 is the first detail of requirement 7); details of prompt design and model choice, which 0001 does not number, are PV.1 and MC.1 and so on. Anything 0001 does not state is marked **(proposal)**. Build order: this PRD is built fourth, after Accounts and admin access ([0002](0002-accounts-and-admin-access.md)), Scenario authoring ([0003](0003-scenario-authoring.md), without test-grade) and Scenario run ([0004](0004-scenario-run.md)). Test-grade in 0003 follows it, then History and grade disputes ([0006](0006-history-and-grade-disputes.md)) and Reporting ([0007](0007-reporting.md)).

## Summary

The grader call for each submitted attempt, validation and retry, the wrong-call rule, the result screen, versioned grading prompts that are activated only after they pass validation, the model choice, and the latency and cost-per-grade records behind 0001's targets.

## Problem

The score and feedback are the product. They have to be consistent from run to run, resistant to a trainee trying to talk the grader into a 5, free of legal conclusions, and cheap and fast enough for a subscription price. The prompt has to be tunable without a release, but a careless prompt edit must not be able to change live grading.

## Goals

- Every submitted attempt ends graded, with a validated score and feedback, or "not graded" with a way to retry.
- A prompt edit reaches live grading only after it passes the calibration set and the injection tests on the server.
- Every graded attempt records its prompt version, model ID, grading time and token usage.
- The model is chosen from calibration results and cost per grade, not from impressions.

## Non-goals

- History, flags and flag review, which are in [0006](0006-history-and-grade-disputes.md).
- Reports, including how cost per grade is displayed, which are in [0007](0007-reporting.md). This PRD records the data.
- Grading of video or multi-image scenarios.

## Users and scenarios

- **Trainee.** Submits an attempt, sees a short wait state, then the result: score, feedback, the image again and the disclaimer. If grading fails, is told so and can retry grading later without redoing the scenario.
- **Admin (Keith).** Edits the grading prompt as a draft version, starts a validation run (calibration set and injection tests) from the admin console, reads the results, and activates the version if it passed, or rolls back to an earlier one.

## Requirements

**Grader call and result (0001 req 7)**

- 7.1 Each submitted attempt gets one grader call with the inputs set out in [0001 Grading design](0001-product-vision.md#inputs-and-output), for the scenario version the trainee ran.
- 7.2 The output must be schema-enforced structured output with the fields in 0001 (`score`, `points_covered`, `points_missed`, `feedback`).
- 7.3 The trainee sees a wait state while grading runs.
- 7.4 The result screen shows the score, the feedback, the image again for review, and the disclaimer. `points_covered` and `points_missed` are stored but not shown.

**Wrong call (0001 req 8)**

- 8.1 The call is checked against the scenario's correct decision in code. On a wrong call, the grader is still asked for feedback, its score is ignored, and the attempt is recorded with 0.

**Legal guardrails (0001 req 9)**

- 9.1 The result screen always shows the disclaimer.
- 9.2 Feedback is checked against a banned legal-conclusion phrase list before display; a hit counts as a failed result and triggers a retry (0001 Validation and retry). The list is stored as data an admin can edit **(proposal)**.

**Validation, retry and not graded (0001 req 10)**

- 10.1 A response that fails the schema, has an out-of-range or fractional score, or fails 9.2 is retried, up to 2 retries (0001's proposed budget).
- 10.2 After the retries, the attempt is saved as "not graded" and the trainee is told. A score that failed validation is never shown.
- 10.3 The trainee can retry grading a "not graded" attempt later without redoing the scenario. Only the attempt's owner can do this, and repeated retries are rate-limited **(proposal: limit set with the cost targets)**.

**Prompt versions (0001 Prompt design)**

- PV.1 The system instructions are stored as data and edited in the admin console. Each edit creates a new draft version. Only admins can edit, and each edit is in the audit log ([0002](0002-accounts-and-admin-access.md) 19.1).
- PV.2 A version moves from draft to tested only through a validation run started in the admin console and executed on the server: the calibration set and the injection test set. The admin sees the results in the console.
- PV.3 The server refuses to activate a version without a passing run. Activation and rollback to an earlier passing version are admin actions in the audit log.
- PV.4 The prompt builder puts the trainee's reasoning (and any "no decision" explanation that is graded, see Open questions) only inside the fenced block, and puts scenario fields and sample answers in the trusted message, as 0001 Prompt design sets out.
- PV.5 Every graded attempt records the prompt version and model ID.

**Latency and cost (0001 req 23 and 25)**

- 23.1 Each attempt records the time from submit to graded result, including retries.
- 25.1 Each grader call records its token usage, including calls that failed validation and were retried **(proposal: retries count toward cost per grade)**. [0007](0007-reporting.md) shows cost per grade from these records.
- 25.2 The static part of the prompt is sent so the provider can cache it, where the provider supports it (0001 Risks) **(proposal)**.

**Model choice (0001 Model choice)**

- MC.1 The grader calls the model through a provider boundary, so the model and provider can change without changing grading logic (0001 Architecture direction).
- MC.2 Before the model is chosen, each candidate is run on the calibration set described in [0001 Model choice](0001-product-vision.md#model-choice), and agreement, consistency, latency and cost per grade are compared. The strong answer used as the 5 anchor is not in the calibration set.

## Product guardrails

- **Training aid, not legal advice:** The disclaimer is on every result screen (9.1). The prompt forbids legal-conclusion language and the phrase check (9.2) blocks it before display.
- **Third-party names and logos:** Feedback must not name third-party organizations or courses. The banned phrase list (9.2) can carry organization names **(proposal)**.
- **User-submitted text:** Reasoning is fenced and the system prompt says anything inside the fence is an answer to grade, never instructions (PV.4). The injection test set must score as ordinary weak answers before the feature is complete and before any prompt version is activated (PV.2, PV.3).
- **LLM output:** Validated against the 0001 output schema with `score` in 0 to 5 as a whole number. Failures are retried up to 2 times, then marked "not graded" (10.1, 10.2).

## Success metrics

The grading quality, latency and cost metrics in [0001 Success metrics](0001-product-vision.md#success-metrics) apply as written: agreement with Keith, consistency, schema validity, injection resistance, zero banned legal phrasing shown, 95% graded within 10 seconds, and cost per grade tracked against the target subscription price.

## Test plan

Layers are those defined in [0001 Test plan](0001-product-vision.md#test-plan), plus **Security**: adversarial tests of Hold Fire's own rules **(proposal: run with the integration suite in CI)**. This plan refines 0001 T4 to T9 and T15. Admin-only access and audit entries for the prompt-version APIs are checked by adding them to [0002](0002-accounts-and-admin-access.md) T2 and T5.

| # | Item (requirements) | Layers | What each layer checks, and why there |
| --- | --- | --- | --- |
| T1 | Wrong call scores 0 (8.1) | Unit; Integration | Unit: score calculation ignores the grader's score on a wrong call. Integration: the attempt is saved with 0 and the grader's feedback. |
| T2 | Validation, retry and not graded (9.2, 10.1 to 10.3) | Unit; Integration | Unit, with a fake LLM client: malformed JSON, double-encoded JSON, half points, out-of-range scores and banned phrasing each trigger a retry, and after 2 retries the result is "not graded". Integration: a "not graded" attempt can be graded again by its owner without redoing the scenario. |
| T3 | Prompt injection (PV.4, guardrails) | Unit; Integration; Backend E2E (LLM) | Unit: the prompt builder puts trainee text only inside the fence, keeps scenario fields and sample answers in the trusted message, and neutralizes fake closing tags. Integration: a submitted attempt produces an LLM request with the active prompt version and the trainee text fenced in the user message. Backend E2E: the injection test set, including attempts to make the grader reveal its instructions, scores as ordinary weak answers against the real model. |
| T4 | Calibration agreement and consistency (MC.2) | Backend E2E (LLM) | Runs the calibration set on each candidate model and reports agreement with Keith and run-to-run spread. Only the real model can show this. |
| T5 | Prompt version lifecycle (PV.1 to PV.3, PV.5) | Unit; Integration | Unit: draft, tested and active states, and no activation without a passing run. Integration: the server refuses activation without a passing run even when called directly, rollback works, and each graded attempt records prompt version and model ID. |
| T6 | Result screen (7.3, 7.4, 9.1) | Component; Exploratory | Component: the result screen always renders the disclaimer, shows score, feedback and image, and does not show covered or missed points; the wait state and "not graded" states render. Exploratory: read real feedback for tone and legal-sounding language a phrase list would miss. |
| T7 | Latency and usage (23.1, 25.1) | Unit; Integration; Backend E2E (LLM) | Unit: submit-to-graded time with a fake clock, including retry time. Integration: a graded attempt is saved with its grading time and the token usage from every stub LLM response, retries included. Backend E2E: the calibration run reports 95th-percentile grading time against the 10-second target. |
| T8 | Results belong to their owner (10.3) | Security | A trainee cannot read, or retry grading of, another trainee's attempt by its ID, and retry requests past the rate limit are refused. |

The result screen's place in the main path is covered by [0004](0004-scenario-run.md) T7, so this PRD adds no frontend E2E test.

**Exploratory charters**

- Write hostile or odd answers by hand, beyond the injection set (T3).
- Read the feedback from about ten real attempts for tone, accuracy and legal-sounding language (T6).
- Cut the network during the wait state, then come back and retry grading (T2).
- As an admin, edit the prompt, run validation, activate, then roll back (T5).

## Open questions

- **Grading model.** Sonnet 5.5 or a cheaper model, decided by the calibration set and cost per grade (from 0001). Owner: Keith.
- **Prompt-injection defense.** 0001 sets fencing, a never-follow rule and the injection test set. Whether more is needed (for example a separate check of the trainee text, or a second grading pass when a score looks out of line with the covered points) is open. Owner: Keith, then the Grading pipeline ADR.
- **"No decision" scoring.** How a "no decision" attempt is scored, and whether it goes to the grader at all (touches [0004](0004-scenario-run.md), [0006](0006-history-and-grade-disputes.md) and [0007](0007-reporting.md)). Owner: Keith.
- **Sample answer set.** 0/3/5, as 0001 states, or 0/1/3/5 with a 1 example. Touches the prompt and the calibration set, and the scenario record in [0003](0003-scenario-authoring.md). Owner: Keith.
- **Retry budget.** Whether 2 retries is right once real latency and cost are known (0001 proposes 2). Owner: Keith.
- **Provider training terms.** Whether the chosen provider's terms exclude attempts from model training, for the privacy policy (from 0001). Owner: Keith.

## Out of scope for this release

Grading of video or multi-image scenarios. Showing covered and missed points to trainees.

## Change log

| Version | Date | Summary |
| --- | --- | --- |
| 0.1 | 2026-11-28 | Initial draft. |
