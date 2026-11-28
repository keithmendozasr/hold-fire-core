# CLAUDE.md

Guidance for Claude Code working in this repository.

## Project

**Hold Fire** (holdfire.app) is a web app for training shoot/no-shoot decisions: the user reviews a scenario, makes a call with written reasoning, and an LLM grades it with feedback.

Product requirements (PRDs) and architecture decisions (ADRs) live in `docs/`. Treat an accepted PRD or ADR as settled; anything not covered there, or still in draft, is open, so check with the code owners before building on it.

## Workflow rules

- Commits have to be signed.
- Use the latest stable versions of libraries and tools, but hang back on fresh X.Y.0 releases of libraries with a history of quick follow-up patches, and say so when you do.
- At the start of a work session, list dependencies (including GitHub Actions versions) that have upgrades available. Do not apply them; just raise them.
- A doc in `docs/` keeps its version while its PR is open. Bump the version and add a change-log row only when a revision merges to main.

## Testing

- Add or update automated tests as part of every change, following the test plan in the PRD that covers the work.
- If no test plan covers the work you are asked to do, stop and ask the code owners for permission before starting.
- Follow the testing pyramid. A test plan item can span layers, but each layer checks something the others cannot: never repeat the same check at another layer.
- Test our own logic, not the libraries. For example, don't test that the HTTP framework rejects malformed JSON or a non-GUID before our code runs.
- Record exploratory test results in the PR description's "Exploratory testing" checklist.

## License

This repo is public and source-available under BUSL-1.1.

- License terms change only with code-owner approval.
- Only bring in dependencies, assets, or copied code whose licenses permit redistribution here; flag copyleft or unclear licenses to the code owners instead.
- This file ships with the code. Keep secrets and private details out of it.

## Site

- `site/` is deployed on its own and served publicly: keep it self-contained, and put nothing in it that shouldn't be public.
- `404.html` must use root-absolute paths, since it is served at any missing URL.

## Brand

- Reuse the existing design tokens and self-hosted fonts; no new colors, no font CDNs.
- Use the accent color once per screen.
- Steel is too low-contrast for text on the background; use it only for lines and borders.

## Product guardrails

- Hold Fire is a **training aid, not legal advice**. Keep that disclaimer visible up front and never present grades or feedback as legal conclusions.
- Keep third-party organizations' names and logos off the product.
- Treat user-submitted text as untrusted: fence it off from LLM prompts, never interpolate it as instructions.
- Validate LLM output against a schema and reject or retry anything that does not conform.
