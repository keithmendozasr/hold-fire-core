# PRD-0003: Scenario authoring

| Field | Value |
| --- | --- |
| Version | 0.1 |
| Date | 2026-11-28 |
| Author | Claude, requested by Keith Mendoza |
| Owner | Keith Mendoza |
| Status | Draft |
| Related ADRs | none yet (0001 proposes Database and data model, and Media storage and delivery) |

This feature PRD refines [PRD 0001](0001-product-vision.md) requirement 16 and the [Content pipeline](0001-product-vision.md#content-pipeline-scenarios-and-images). Detail requirements keep 0001's numbers (16.1 is the first detail of requirement 16); details of the content pipeline, which 0001 does not number, are CP.1, CP.2 and so on. Anything 0001 does not state is marked **(proposal)**. Build order: this PRD is built second, after Accounts and admin access ([0002](0002-accounts-and-admin-access.md)) and without test-grade (16.4). Test-grade is built after Grading and results ([0005](0005-grading-and-results.md)), since it uses the grader, and before History and grade disputes ([0006](0006-history-and-grade-disputes.md)) and Reporting ([0007](0007-reporting.md)).

## Summary

The admin screens and storage Keith uses to write a scenario, upload its image, preview it as a trainee would, test-grade it, schedule its release and publish it, with every grading-relevant edit creating a new version.

## Problem

0001 has Keith writing every scenario and releasing one every one to two weeks, with a buffer of finished scenarios ahead of schedule. That cadence needs scenarios to go from draft to live without a code change. The details the grader scores live in the image and its description, so authoring has to make the image review step hard to skip, and versioning has to keep past grades tied to what they were graded against.

## Goals

- Keith can take a scenario from draft to scheduled to live entirely in the admin console.
- A published scenario's grading-relevant edits never change what a past attempt was graded against.
- The buffer of scheduled scenarios is visible at a glance.
- Images are stored and served by Hold Fire only, in storage that can take video later.

## Non-goals

- Generating images inside Hold Fire. The image generator is a tool Keith uses outside the product (0001 Content pipeline).
- Video, multiple images per scenario, or user-submitted scenarios (0001 Non-goals).
- The trainee-facing library and run, which are in [0004](0004-scenario-run.md).
- Editing the grading prompt, which is in [0005](0005-grading-and-results.md).

## Users and scenarios

- **Admin (Keith).** Writes the context, intended scene, correct decision and expected answer; generates an image outside Hold Fire; uploads it; rewrites the image description to match what the image actually shows; works through the image review checklist; writes the sample answers; previews the scenario; test-grades it with a few answers; sets a release date; and publishes. Later edits a live scenario, which creates a new version, or unpublishes it.

## Requirements

**Scenario record and versioning (0001 req 16)**

- 16.1 An admin must be able to add and edit a scenario with the fields listed in [0001 Content pipeline](0001-product-vision.md#content-pipeline-scenarios-and-images). Display time defaults to 3 seconds (0001 req 2).
- 16.2 An admin must be able to upload the scenario's image. It is stored in Hold Fire's own storage and is never reachable by a public URL; the trainee sees it only through the run's image controls in [0004](0004-scenario-run.md) (0001 req 3). Accepted formats and size limit **(proposal: JPEG, PNG or WebP, limit set in the Media storage ADR)**. Image metadata is stripped on upload **(proposal)**.
- 16.3 An admin must be able to preview a scenario as a trainee would see it, including the image for its display time. A preview creates no attempt and does not count in reports **(proposal)**.
- 16.4 An admin must be able to test-grade a scenario: submit test answers and see the score, feedback, and covered and missed points that the grader returns for the scenario's current version under the active prompt version. Test-grade results are not attempts and do not count in reports or history **(proposal)**. Built after [0005](0005-grading-and-results.md).
- 16.5 An admin must be able to set a release date. A scheduled scenario becomes live on that date without further action, and the console lists scheduled scenarios by release date so the buffer is visible (0001 Release cadence and Risks).
- 16.6 An admin must be able to publish and unpublish a scenario. An unpublished scenario leaves the library, but trainees' past attempts at it stay in their history **(proposal)**.
- 16.7 Editing a published scenario's image, image description, expected answer, sample answers or correct decision must create a new version. Past attempts keep pointing to the version they were graded against.
- 16.8 Publishing must be refused unless every field the grader needs is filled in: context text, image, image description, correct decision, expected answer and all sample answers **(proposal)**.

**Content pipeline**

- CP.1 The image review checklist from 0001 Content pipeline step 3 must appear in the console, and publishing must be refused until an admin has confirmed it for the current image **(proposal: 0001 states the checklist but not that it gates publishing)**.
- CP.2 The image prompt is stored with the scenario, so the image can be regenerated with another generator (0001 Risks).
- CP.3 Nothing is loaded from the image generator at run time (0001 Storage).
- CP.4 The chosen generator's terms must be confirmed to allow commercial use and redistribution before the first scenario is published (0001 Image licensing). This is a release gate, not a product feature.
- CP.5 Every authoring action (create, edit, upload, schedule, publish, unpublish, test-grade) is an admin action recorded in the audit log (0001 req 19, [0002](0002-accounts-and-admin-access.md) 19.1).

## Product guardrails

- **Training aid, not legal advice:** Preview shows the scenario as a trainee sees it, so it shows the disclaimer wherever the run does ([0004](0004-scenario-run.md)). Expected answers are written as tactics and judgment, not legal guidance, and are part of the attorney review in 0001.
- **Third-party names and logos:** The image review checklist (CP.1) requires no readable brand names, logos or third-party marks in the image. The same rule applies to the context text.
- **User-submitted text:** Scenario fields are trusted admin content and go to the grader as trusted content (0001 Grading design). Test-grade answers typed by an admin are treated exactly like a trainee's answer: fenced, never as instructions, using the same prompt builder as [0005](0005-grading-and-results.md).
- **LLM output:** Test-grade uses the same schema validation and retry as [0005](0005-grading-and-results.md). A result that fails validation is shown as "not graded", never with a score.

## Success metrics

- Release cadence: a new scenario goes live every one to two weeks (0001 Goals).
- Buffer: at least one finished, scheduled scenario ahead of the next release date at all times **(proposal: buffer size to be set by Keith)**.
- Zero scenarios published with an unconfirmed image checklist.

## Test plan

Layers are those defined in [0001 Test plan](0001-product-vision.md#test-plan), plus **Security**: adversarial tests of Hold Fire's own rules **(proposal: run with the integration suite in CI)**. This plan refines 0001 T12. Admin-only access and audit entries for the authoring APIs are checked by adding them to [0002](0002-accounts-and-admin-access.md) T2 and T5, not by separate tests here.

| # | Item (requirements) | Layers | What each layer checks, and why there |
| --- | --- | --- | --- |
| T1 | Which edits create a version (16.7) | Unit; Integration | Unit: the rule that decides whether an edit to a published scenario creates a new version, over every field. Integration: after a versioning edit, an existing attempt still returns the version it was graded against, and new runs use the new one. |
| T2 | Scenario visibility over time (16.5, 16.6) | Unit | The visibility rule with a fake clock: draft, scheduled before and after its release date, live, unpublished. Pure logic; the library API in [0004](0004-scenario-run.md) uses it and does not retest it. |
| T3 | Publish gates (16.8, CP.1) | Integration | The server refuses to publish a scenario missing any required field or with an unconfirmed checklist, and a new image upload clears the checklist confirmation. The server has to enforce this, so the test goes through the API. |
| T4 | Image upload and storage (16.2, CP.3) | Integration; Security | Integration: an upload is stored in Hold Fire storage, our format and size rule refuses other files, and metadata is stripped. Security: a file whose content does not match its declared type, an oversized file, and a guessed storage path are all refused, and no stored image has a public URL. |
| T5 | Preview has no side effects (16.3) | Integration | A preview creates no attempt and changes no report figure. |
| T6 | Test-grade (16.4) | Integration; Component | Integration, with the stub LLM: the request uses the scenario's current version and the active prompt version, the admin's answer goes through the same prompt builder as a trainee's, and no attempt is stored. (Fencing itself is tested in [0005](0005-grading-and-results.md) T3.) Component: the test-grade panel shows score, feedback, and covered and missed points, and shows "not graded" on a failed result. |
| T7 | Author to live (16.1, 16.2, 16.5) | Frontend E2E | One run in a real browser: create a scenario, upload an image, confirm the checklist, publish, and see it in the trainee library. This is the only test that the authoring screens and the library are wired together. |

**Exploratory charters**

- Author a real scenario end to end with a generated image, working the review checklist honestly against the image (T3, T7).
- Edit a published scenario that has attempts, then check those attempts still show and grade against the old version (T1).
- Schedule three scenarios and check the buffer view and the day each goes live (T2).

## Open questions

- **Sample answer set.** Whether each scenario carries 0/3/5 sample answers, as 0001 states, or adds a 1 example (0/1/3/5). Touches the scenario record here and the grader in [0005](0005-grading-and-results.md). Owner: Keith.
- **Image generator.** Higgsfield or an alternative such as kie.ai, decided partly by commercial-use terms (from 0001). Owner: Keith.
- **Second reviewer.** Whether someone besides Keith reviews expected answers before publishing (from 0001). Owner: Keith.
- **Other versioned fields.** Whether edits to context text and display time also create a new version. The grader receives the context text, so a change to it changes grading **(proposal: yes)**. Owner: Keith.
- **Unpublish and repeats.** Whether an unpublished scenario can still be reopened from history or resurfaced for practice ([0004](0004-scenario-run.md)). Owner: Keith.
- **Test-grade records.** Whether test-grade results are kept (for example for the audit log or for calibration) or discarded after viewing. Owner: Keith.

## Out of scope for this release

Video and multi-image scenarios, in-product image generation, user-submitted scenarios.

## Change log

| Version | Date | Summary |
| --- | --- | --- |
| 0.1 | 2026-11-28 | Initial draft. |
