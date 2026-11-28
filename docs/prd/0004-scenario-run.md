# PRD-0004: Scenario run

| Field | Value |
| --- | --- |
| Version | 0.1 |
| Date | 2026-11-28 |
| Author | Claude, requested by Keith Mendoza |
| Owner | Keith Mendoza |
| Status | Draft |
| Related ADRs | none yet (0001 proposes Media storage and delivery, and Database and data model) |

This feature PRD refines [PRD 0001](0001-product-vision.md) requirements 1 to 6 and steps 1 to 9 of the [User flow](0001-product-vision.md#user-flow). Detail requirements keep 0001's numbers (4.1 is the first detail of requirement 4); details of user flow steps, which 0001 does not number as requirements, are UF1.1, UF3.1 and so on. Anything 0001 does not state is marked **(proposal)**. Build order: this PRD is built third, after Accounts and admin access ([0002](0002-accounts-and-admin-access.md)) and Scenario authoring ([0003](0003-scenario-authoring.md), without test-grade), and before Grading and results ([0005](0005-grading-and-results.md)). Until 0005 is built, a submitted attempt is stored and waits for grading.

## Summary

The trainee's side of a scenario: the library, the one-time disclaimer acknowledgment, reading the context, Ready, the timed image, the 5-second call, "no decision", the written reasoning and submit, plus the rules that abandon an attempt.

## Problem

The training value rests on a short, fair, unrepeatable look at the scene followed by a quick call. If the image can be seen again, or the timers are unfair on a slow phone, the attempt stops measuring how the trainee reads a scene under time pressure.

## Goals

- A trainee can pick a scenario and run it from context to submit, on a desktop or phone browser.
- The image is seen once, for its display time, and cannot be seen again until the attempt is submitted.
- Every attempt that reaches Ready ends in exactly one recorded state: submitted, "no decision" submitted, or abandoned.

## Non-goals

- Grading and the result screen, which are in [0005](0005-grading-and-results.md).
- Preventing screenshots of the image (accepted risk in 0001 Non-goals).
- Video or tap-to-shoot timing (0001 Non-goals).
- Native apps. The web app must work well in a phone browser (0001 Non-goals).

## Users and scenarios

- **Trainee.** Opens the library, picks a scenario or takes the next one not yet done, acknowledges the disclaimer the first time, reads the context, taps Ready, sees the image, chooses Shoot or No Shoot within 5 seconds (or types why they did not choose), writes their reasoning and submits.
- **Trainee who walks away.** Leaves the page or goes inactive for 5 minutes; the attempt is abandoned and they start over next time.

## Requirements

**Disclaimer and library (0001 User flow steps 1 and 3)**

- UF1.1 Before the first scenario, the trainee must acknowledge the training-aid disclaimer once ([0001 Disclaimers](0001-product-vision.md#disclaimers-and-legal-guardrails)). The acknowledgment is stored with the account and the run cannot start without it **(proposal: storage and server enforcement)**.
- UF3.1 The library lists live scenarios only (visibility rule from [0003](0003-scenario-authoring.md) 16.5 and 16.6).
- UF3.2 Scenarios the trainee has already done are marked.
- UF3.3 The trainee can take the next scenario they have not done **(proposal: the oldest live scenario they have not done)**.

**Run (0001 req 1 to 6)**

- 1.1 The scenario's context text is shown with no time limit and a Ready control.
- 2.1 After Ready, the image is shown full-screen for the scenario's display time and then hidden. The display time used is recorded with the attempt.
- 3.1 The server must refuse the image before Ready, after it hides, and before the attempt is submitted, including on browser back, reload or direct URL. The mechanism (for example a single-use, attempt-bound link issued at Ready) is decided in the Media storage ADR **(proposal)**.
- 3.2 The image must not be preloaded in a way that lets the trainee inspect it before Ready.
- 4.1 The call clock starts when the image hides. The trainee must choose Shoot or No Shoot within 5 seconds.
- 4.2 Past 5 seconds the attempt is recorded as "no decision", and the trainee must type why they did not choose. That text is saved with the attempt for admin review under the reasoning-text permission ([0002](0002-accounts-and-admin-access.md) 17.5).
- 4.3 The server records when the image was shown, when it hid and when the call was made, and decides whether the call was in time **(proposal: with a small tolerance for network delay, see Open questions)**.
- 4.4 Submission requires a call (or "no decision" with its explanation) and written reasoning.
- 5.1 Reasoning must be at least 20 and at most 2,000 characters (0001's proposed limits), enforced on the server and shown in the field before submit.
- 6.1 If the trainee leaves mid-attempt, or is inactive for 5 minutes at any step, the attempt is abandoned and not scored. It cannot be resumed.
- 6.2 An attempt abandoned after Ready is stored as abandoned, so the completion share in [0007](0007-reporting.md) (0001 req 24) can count it.

## Product guardrails

- **Training aid, not legal advice:** The disclaimer is acknowledged before the first scenario (UF1.1). Scenario text and images avoid legal framing, per [0003](0003-scenario-authoring.md).
- **Third-party names and logos:** Images and context text are checked for third-party marks at authoring time ([0003](0003-scenario-authoring.md) CP.1). The run screens carry only Hold Fire branding.
- **User-submitted text:** Reasoning and "no decision" text are stored as data and never shown to other users. They reach the grader only through the fenced prompt in [0005](0005-grading-and-results.md).
- **LLM output:** Not applicable; this feature makes no LLM call.

## Success metrics

- Completion: at least 80% of attempts that reach Ready are submitted (0001 Success metrics; reported in [0007](0007-reporting.md)).
- Zero confirmed ways to see the image again before submit (from the T3 tests and charters).
- No trainee reports of an unfair 5-second call caused by load time or zoom during the beta **(proposal)**.

## Test plan

Layers are those defined in [0001 Test plan](0001-product-vision.md#test-plan), plus **Security**: adversarial tests of Hold Fire's own rules **(proposal: run with the integration suite in CI)**. This plan refines 0001 T1 to T3.

| # | Item (requirements) | Layers | What each layer checks, and why there |
| --- | --- | --- | --- |
| T1 | Attempt state machine (1.1, 2.1, 4.1, 4.2, 4.4, 6.1) | Unit | With a fake clock: every state and timeout, including the 5-second call, "no decision", the 5-minute inactivity abandon at each step, and that an abandoned attempt cannot resume. Timing logic is pure, so this is the exact and cheap place to test it. |
| T2 | Server enforces the run (4.3, 4.4, 6.2, UF1.1) | Integration | The attempt API refuses out-of-order steps (submit before Ready, a call after the window, starting before the disclaimer is acknowledged), stores the display time and timestamps, and stores an attempt abandoned after Ready as abandoned. It checks that the API uses the T1 rules, not the rules again. |
| T3 | Image not viewable again before submit (3.1, 3.2) | Integration; Security; Exploratory | Integration: the image endpoint refuses before Ready, after hide and before submit. Security: a replayed image link, another user's attempt link, a guessed storage path, and a cached copy from our response headers are all refused or absent. Exploratory: back, reload, direct URL and browser dev tools. |
| T4 | Reasoning length (5.1) | Unit; Component | Unit: the server-side rule at its boundaries. Component: the field shows the message and blocks submit. |
| T5 | Call screen timeout (4.2) | Component | With a fake clock, the call screen switches to the "why didn't you choose" field when the 5 seconds run out. |
| T6 | Library (UF3.1 to UF3.3) | Unit; Integration | Unit: the "next scenario" choice. Integration: the library API lists live scenarios only and marks the ones this trainee has done. |
| T7 | Main path (UF1.1, 1.1 to 4.4) | Frontend E2E | One run in a real browser: acknowledge the disclaimer, Ready, image, call, reasoning, submit. Once [0005](0005-grading-and-results.md) is built, the same run continues to the result screen with the stub LLM. This is the only end-to-end test of the run. |

**Exploratory charters**

- Try to see the image again before submitting (T3).
- Run a scenario on a phone over a slow connection, and check the 5-second call is still fair with large text or zoom turned on (T1, T7).
- Leave mid-attempt in different ways (close the tab, switch apps, lock the phone) and check what counts as leaving (T1, T2).

## Open questions

- **"No decision" scoring.** How a "no decision" attempt is scored (for example 0, like a wrong call), and whether its explanation or reasoning is sent to the grader. Touches this PRD, [0005](0005-grading-and-results.md), [0006](0006-history-and-grade-disputes.md) and [0007](0007-reporting.md). Owner: Keith.
- **Clock authority.** Whether the server or the browser decides that a call was in time, and how much network tolerance is fair **(proposal: the server decides, with a small tolerance)**. Owner: Keith, then the backend ADR.
- **What counts as leaving.** Whether switching tabs or apps abandons an attempt, or only closing or navigating away. Owner: Keith.
- **"No decision" explanation limits.** Whether the explanation has the same 20 to 2,000 character limits as reasoning. Owner: Keith.
- **Spaced resurfacing.** Whether earlier scenarios come back on a schedule alongside new releases; 0001 assigns this question to this PRD. Owner: Keith.
- **Replay.** Whether a trainee can repeat a scenario at will, and whether repeats count toward progress (from 0001). Owner: Keith.

## Out of scope for this release

Video, multi-image and tap-to-shoot scenarios. Screenshot prevention. Native apps.

## Change log

| Version | Date | Summary |
| --- | --- | --- |
| 0.1 | 2026-11-28 | Initial draft. |
