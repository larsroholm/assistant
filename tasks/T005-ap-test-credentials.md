---
id: T005
title: Datasolvr-test - rigtige credentials og rettigheder
customer: AP
priority: 🔴
status: waiting
due: 2026-09-30
effort:
source: AstridSby (GitHub issue)
created: 2026-09-28
completed:
---

## Context

GitHub issue: https://github.com/datasolvr/appension/issues/143

Datasolvr-t-001 should run on real credentials instead of the ones from
Postman. The test environment should have the same rights as staging and
production, not simply all rights — so that the rights used in production
are validated as sufficient and correct ahead of time.

Blocks T006 (#152), which needs this resolved before rights can be added to
the test environment.

Field values from GitHub: Priority `should-have`, Stage `In Progress`.

## Definition of done

- Datasolvr-t-001 runs on real credentials, not Postman's.
- Test environment rights match staging/production exactly.

## Log

- 2026-09-28: Task created from GitHub issue #143, assigned to larsroholm.
- 2026-09-28: Waiting on AP Pension IT, same as T006.

## Links

- https://github.com/datasolvr/appension/issues/143
