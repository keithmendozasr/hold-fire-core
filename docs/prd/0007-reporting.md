# PRD-0007: Reporting

| Field | Value |
| --- | --- |
| Version | 0.1 |
| Date | 2026-11-28 |
| Author | Claude, requested by Keith Mendoza |
| Owner | Keith Mendoza |
| Status | Draft |
| Related ADRs | none yet (0001 proposes Database and data model) |

This feature PRD refines [PRD 0001](0001-product-vision.md) requirements 20, 21, 22 and 24, the flag count in [Grade disputes](0001-product-vision.md#grade-disputes), and the cost-per-grade report in requirement 25. Detail requirements keep 0001's numbers (20.1 is the first detail of requirement 20); the flag count is GD.6. Anything 0001 does not state is marked **(proposal)**. Recording grading time, token usage and cost data is in [0005](0005-grading-and-results.md); this PRD shows it. Build order: this PRD is built last, after Accounts and admin access ([0002](0002-accounts-and-admin-access.md)), Scenario authoring ([0003](0003-scenario-authoring.md)), Scenario run ([0004](0004-scenario-run.md)), Grading and results ([0005](0005-grading-and-results.md)) and test-grade in 0003, alongside History and grade disputes ([0006](0006-history-and-grade-disputes.md)).

## Summary

Admin console reports on how each scenario performs, how many flags it draws, what share of started attempts are finished, and what each grade costs, using aggregate numbers only.

## Problem

Keith needs to see whether a scenario is working (who ran it, how they scored, whether most people miss it), whether trainees finish what they start, and whether grading cost fits the price, so he can fix a bad scenario and set pricing on real usage.

## Goals

- For each scenario, Keith can see completions, average score, release-to-first-run time, the Shoot and No Shoot split, the wrong-call share and the flag count.
- Keith can track the 80% completion target and cost per grade.
- No report shows reasoning text.

## Non-goals

- Per-user reports or any report that shows reasoning text (0001 req 22).
- Exports, dashboards outside the admin console, or reports for trainers and agencies.
- Collecting new data. Reports use what [0004](0004-scenario-run.md), [0005](0005-grading-and-results.md) and [0006](0006-history-and-grade-disputes.md) store.

## Users and scenarios

- **Admin (Keith).** Opens the reports screen after a release to see how the new scenario is doing, spots a scenario with a high wrong-call share or many flags and opens it in authoring to check its image and description, and checks completion and cost per grade over the beta.

## Requirements

**Per-scenario performance (0001 req 20 and 21)**

- 20.1 For each scenario, the report shows the number of users who completed it **(proposal: completed means submitted and graded)**, the average score, and the average time between the scenario's release date and each user's first run of it.
- 21.1 For each scenario, the report shows the split of Shoot and No Shoot calls and the share of wrong calls.
- 20.2 Preview and test-grade runs never count ([0003](0003-scenario-authoring.md) 16.3, 16.4).
- 20.3 Figures can be shown per scenario version as well as for the scenario as a whole **(proposal: a version can grade differently after an edit)**.

**Privacy (0001 req 22)**

- 22.1 Reports use aggregate numbers only and never show reasoning text, "no decision" explanations or flag notes **(proposal: the last two extend 0001's rule to the other free text trainees write)**.

**Flags (0001 Grade disputes)**

- GD.6 Reports show the number of flagged attempts per scenario.

**Completion and cost (0001 req 24 and 25)**

- 24.1 Reports show the share of attempts that reached Ready and were then submitted, against the 80% target. Abandoned attempts are counted from the records [0004](0004-scenario-run.md) 6.2 stores.
- 25.1 Reports show cost per grade, computed from the token usage recorded in [0005](0005-grading-and-results.md) 25.1 or taken from the provider's usage and billing data (0001 leaves the source open).

**Engagement (proposal)**

- EN.1 Reports also show returning trainees (share of users who run scenarios in at least two separate weeks) and scenarios per active trainee per week, the engagement metrics in [0001 Success metrics](0001-product-vision.md#success-metrics) **(proposal: 0001 lists these as metrics but not as report requirements)**.

## Product guardrails

- **Training aid, not legal advice:** Not applicable; reports are admin-only.
- **Third-party names and logos:** Not applicable.
- **User-submitted text:** Reports contain no trainee-written text at all (22.1).
- **LLM output:** Not applicable; reports read stored, already validated results.

## Success metrics

- Keith can read every iteration 1 success metric that comes from usage data (completion, disputes, cost per grade, and with EN.1 the engagement metrics) from the admin console without a database query **(proposal)**.

## Test plan

Layers are those defined in [0001 Test plan](0001-product-vision.md#test-plan), plus **Security**: adversarial tests of Hold Fire's own rules **(proposal: run with the integration suite in CI)**. This plan refines 0001 T14. Admin-only access to the report APIs is checked by adding them to [0002](0002-accounts-and-admin-access.md) T2. That an abandoned attempt is stored so it counts is tested in [0004](0004-scenario-run.md) T2, and that usage is recorded is tested in [0005](0005-grading-and-results.md) T7; neither is retested here.

| # | Item (requirements) | Layers | What each layer checks, and why there |
| --- | --- | --- | --- |
| T1 | Aggregation math (20.1, 20.3, 21.1, GD.6, 24.1, 25.1, EN.1) | Unit | On fixed data: completions, average score, release-to-first-run time (first run only, ignoring repeats), Shoot and No Shoot split, wrong-call share, flag count, completion share, cost per grade, and engagement figures, including a scenario with no attempts and attempts that are "not graded" or abandoned. The math is pure, so it is tested here once. |
| T2 | What reports read and return (20.2, 22.1) | Integration | With seeded data in a real database: preview and test-grade runs are excluded, and the report responses contain no reasoning, "no decision" explanation or flag-note field. |
| T3 | Report screen (20.1, 21.1) | Component | The report table renders figures and an empty state for a scenario with no attempts yet. |

This feature adds no trainee-facing path, so it has no frontend E2E test, and its only security rule (admin-only) is in 0002's matrix.

**Exploratory charters**

- After a few weeks of beta data, check the report numbers for one scenario against a hand count (T1, T2).
- Read the report as Keith would after a release and note what question it can't answer yet.

## Open questions

- **"No decision" in reports.** Whether "no decision" attempts count as wrong calls, how they affect the average score and completion share, and whether they get their own column (touches [0004](0004-scenario-run.md), [0005](0005-grading-and-results.md) and [0006](0006-history-and-grade-disputes.md)). Owner: Keith.
- **What "completed" means.** Submitted, or submitted and graded, for the completions figure. Owner: Keith.
- **Deleted users.** Whether a deleted user's attempts leave the aggregates or stay as anonymous counts (raised in [0002](0002-accounts-and-admin-access.md)). Owner: Keith, with attorney input.
- **Release-to-first-run for late joiners.** Whether the time is measured from the release date even for users who joined after it, which inflates the average **(proposal: measure from the later of the release date and the user's account creation)**. Owner: Keith.
- **Cost source.** Computed from recorded token usage or taken from the provider's billing data (from 0001). Owner: Keith.

## Out of scope for this release

Exports, per-user reports, trainer and agency reports, and discount-code redemption reports (with billing).

## Change log

| Version | Date | Summary |
| --- | --- | --- |
| 0.1 | 2026-11-28 | Initial draft. |
