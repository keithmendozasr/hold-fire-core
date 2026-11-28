# PRD-0002: Accounts and admin access

| Field | Value |
| --- | --- |
| Version | 0.1 |
| Date | 2026-11-28 |
| Author | Claude, requested by Keith Mendoza |
| Owner | Keith Mendoza |
| Status | Draft |
| Related ADRs | none yet (0001 proposes Identity and sign-in, and Database and data model) |

This feature PRD refines [PRD 0001](0001-product-vision.md) requirements 13, 14, 15, 17 and 19. Detail requirements keep 0001's numbers, so 13.1 is the first detail of 0001 requirement 13 and references to 0001 still resolve. Anything 0001 does not state is marked **(proposal)**. Build order: this PRD is built first, before Scenario authoring (0003), Scenario run (0004), Grading and results (0005), test-grade in 0003, then History and grade disputes (0006) and Reporting (0007).

## Summary

Sign-in, admin-created beta accounts, trainee deletion requests that an admin carries out, the admin console shell that every other feature adds its screens to, user management, the audit log, and logged access to trainees' reasoning text.

## Problem

0001 requires an account for everything, including a first scenario, and in iteration 1 Keith enrolls beta users himself ([0001 User flow](0001-product-vision.md#user-flow), step 2). Every other feature needs a signed-in user, and every admin screen needs one place to live and one set of access rules. Reasoning text is sensitive ([0001 Accounts and data](0001-product-vision.md#accounts-and-data)), so who can read it has to be controlled and recorded before any attempt is stored.

## Goals

- Keith can enroll a beta user, and that user can sign in, sign out and recover access without help.
- A trainee can ask for their account to be deleted, and an admin can carry it out completely.
- The admin console exists as a shell, available only to admins, that Scenario authoring, Grading, Grade disputes and Reporting add their screens to.
- Every admin action is in an audit log, and every read of a trainee's reasoning text is too.

## Non-goals

- Self sign-up. Whether people can create their own accounts is open in 0001.
- Discount codes (0001 requirement 18). They depend on payments, and billing is deferred.
- Trainer, agency or cohort roles. 0001 lists these as later work.
- Third-party sign-in (for example "Sign in with ..." buttons). 0001 does not ask for it, and its branding would conflict with the no-third-party-logos guardrail.

## Users and scenarios

- **Admin (Keith).** Creates a beta account for a new user, finds a user to check their status and attempt count, disables or re-enables an account, carries out a pending deletion request, and reads the audit log. When reviewing a user's actions (for example a "no decision" explanation or a flagged grade), opens the reasoning text through the separate, logged permission.
- **Trainee.** Receives an enrollment message, sets up sign-in, signs in and out, recovers access, and can request deletion of their account from their account settings.

## Requirements

**Account lifecycle (0001 req 13)**

- 13.1 An admin must be able to create a trainee account. The new user gets a way to set up sign-in (for example an invitation link that sets a password or starts passwordless sign-in) **(proposal: the mechanism is decided in the Identity ADR)**.
- 13.2 Every page and API of the app except the public site must require a signed-in user ([0001 User flow](0001-product-vision.md#user-flow), step 2).
- 13.3 The trainee must be able to sign out, and signing out must end the session on the server, not only in the browser **(proposal)**.
- 13.4 The trainee must be able to recover access through password reset or passwordless sign-in. Reset and sign-in links must be single-use and expire **(proposal: expiry time decided in the Identity ADR)**.
- 13.5 A disabled account (17.2) must not be able to sign in, and its existing sessions must end when it is disabled **(proposal)**.
- 13.6 The reset and sign-in screens must not reveal whether an email address has an account **(proposal)**.

**Deletion (0001 req 14)**

- 14.1 A trainee must be able to request deletion of their account from their account settings. The request is recorded and deletes nothing by itself.
- 14.2 Pending deletion requests must be listed in the admin console.
- 14.3 When an admin carries out a deletion, the account, the trainee's stored reasoning text, "no decision" explanations, grade-dispute notes and attempt history must be deleted. Other users' data is untouched.
- 14.4 The trainee must be told that their request was received and that an admin will carry it out **(proposal: with a stated turnaround, see Open questions)**.

**Admin console shell (0001 req 15)**

- 15.1 The admin console must be available only to admin accounts. The server enforces this on every admin API; hiding links in the browser is not the control.
- 15.2 The console provides navigation that feature PRDs add screens to: users (this PRD), scenarios and test-grade ([0003](0003-scenario-authoring.md)), prompt versions ([0005](0005-grading-and-results.md)), flag review ([0006](0006-history-and-grade-disputes.md)) and reports ([0007](0007-reporting.md)).

**User management (0001 req 17)**

- 17.1 An admin must be able to find a user and see their account status and attempt count.
- 17.2 An admin must be able to disable and re-enable an account.
- 17.3 An admin must be able to carry out a deletion request as set out in 14.3.
- 17.4 Admins must not see a user's reasoning text by default. Reading it needs a separate permission, held per admin account. Granting or removing that permission is itself an admin action in the audit log **(proposal)**.
- 17.5 The reasoning-text permission covers every place reasoning appears in the admin console, including "no decision" explanations (0001 req 4) and flagged attempts ([0006](0006-history-and-grade-disputes.md)) **(proposal)**.

**Audit log (0001 req 19)**

- 19.1 Every admin write action must be recorded with who, what, when and the record it touched.
- 19.2 Every read of a user's reasoning text under 17.4 must be recorded the same way.
- 19.3 The audit log must be readable in the admin console and must not be editable or deletable through any API **(proposal)**.
- 19.4 Audit entries must not copy reasoning text into the log **(proposal)**.

## Product guardrails

- **Training aid, not legal advice:** Not applicable here. The landing page disclaimer lives on the public site, and the acknowledgment before the first scenario is in [0004](0004-scenario-run.md).
- **Third-party names and logos:** Sign-in, reset and enrollment screens and messages carry only Hold Fire branding. If a hosted identity provider is chosen, its name and logo must not appear on them **(proposal)**.
- **User-submitted text:** Nothing in this feature is sent to an LLM. Reasoning text is only read here, under 17.4.
- **LLM output:** Not applicable.

## Success metrics

All proposed; 0001 sets no account metrics.

- Every pending deletion request is carried out within the agreed turnaround (see Open questions).
- Zero admin API calls succeed for a non-admin account, as shown by the authorization tests and the audit log.
- Every read of reasoning text has a matching audit entry.

## Test plan

Layers are those defined in [0001 Test plan](0001-product-vision.md#test-plan), plus **Security**: adversarial tests of Hold Fire's own access rules, such as forged roles, tampered sessions and other users' record IDs **(proposal: run with the integration suite in CI)**. Security tests try to get around a rule; integration tests check the rule as designed, so the two never repeat each other. This plan refines 0001 T10 and T11. If identity is a hosted provider, tests cover our use of it (roles, account status, sessions we issue), never the provider's own sign-in logic.

| # | Item (requirements) | Layers | What each layer checks, and why there |
| --- | --- | --- | --- |
| T1 | Account status rules (13.5, 14.1, 17.2, 17.3) | Unit | The account state machine (active, disabled, deletion requested, deleted) allows only the intended transitions. Pure logic, so it is tested once here and the integration tests below do not repeat each transition. |
| T2 | Admin authorization matrix (15.1, 17.4, 17.5) | Integration | One table-driven test calls every admin API as anonymous, trainee, disabled user, admin without the reasoning permission and admin with it, and checks allow or refuse for each. Feature PRDs add their admin APIs to this table instead of writing their own authorization tests. |
| T3 | Enrollment and sign-in (13.1, 13.2, 13.4, 13.5) | Integration; Frontend E2E | Integration: an admin-created account can sign in, a disabled account is refused and its sessions end, and reset or sign-in links work once and then fail. Frontend E2E: one run where an admin enrolls a user and that user signs in for the first time, which only a real browser shows end to end. |
| T4 | Deletion (14.1 to 14.3) | Integration | A request alone deletes nothing; an admin deletion removes the account, reasoning, "no decision" explanations, dispute notes and attempts; a second user's data is untouched. |
| T5 | Audit log (19.1 to 19.4) | Integration | Table-driven over the admin write actions and the reasoning-read endpoints: each writes one entry with who, what, when and target, no entry contains reasoning text, and no API can change or delete an entry. Feature PRDs add their actions to this table. |
| T6 | Access-rule bypass (13.3, 13.5, 13.6, 15.1) | Security | A trainee session with a forged or edited role is refused on admin APIs; a session used after sign-out or after the account is disabled is refused; a replayed reset link is refused; reset responses are the same for known and unknown emails. |
| T7 | Deletion request screen (14.1, 14.4) | Component | The account settings screen asks for confirmation and then shows that the request was received and an admin will carry it out. |

**Exploratory charters**

- Walk through the admin console as a new admin: create a user, disable and re-enable them, carry out a deletion (T2, T4).
- As a trainee, try to reach admin pages and admin API URLs directly (T2, T6).
- Request deletion as a trainee and read what you are told (T7).

## Open questions

- **Self sign-up.** Whether people can create their own accounts, or only an admin creates them (from 0001). Owner: Keith.
- **Password or passwordless.** Which sign-in method iteration 1 uses; 0001 allows either. To be settled in the Identity ADR. Owner: Keith.
- **Age gate.** Minimum age and whether it is checked at enrollment (from 0001). Owner: Keith, with attorney input.
- **Retention of reasoning text.** How long attempts are kept short of account deletion (from 0001). Owner: Keith, with attorney input.
- **Deletion turnaround.** How quickly an admin must carry out a deletion request, and what the trainee is told **(proposal)**. Owner: Keith, with attorney input.
- **Reports after deletion.** Whether a deleted user's attempts are removed from report aggregates or kept as anonymous counts (touches [0007](0007-reporting.md)). Owner: Keith.
- **Reasoning permission with one admin.** While Keith is the only admin, who grants the reasoning-text permission, and whether it needs a reason recorded with each read **(proposal)**. Owner: Keith.

## Out of scope for this release

Discount codes (0001 requirement 18), deferred with billing. Trainer and agency roles. Third-party sign-in.

## Change log

| Version | Date | Summary |
| --- | --- | --- |
| 0.1 | 2026-11-28 | Initial draft. |
