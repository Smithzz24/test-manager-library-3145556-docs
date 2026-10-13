# test-manager-library-3145556-docs

> **Technical assessment · ADSO 3145556**
> Deadline: **Tuesday, October 13, 2026, at 5:00 p.m.** (17:00, Colombia time).

This repository contains the *Manager Library* specification. It is the only repository with content; the other five intentionally start empty.

## Instructions

Each trainee must **fork all six project repositories** and **build the project from the proposed specification**.

1. Fork each repository in the table at the end to your GitHub account.
2. Read the specification in [`test-manager-library-3145556-docs`](https://github.com/code-sena/test-manager-library-3145556-docs).
   It is provided in two versions: `spec-python/` and `spec-.net/`.
3. Develop in your forks.

## The challenge uses React and PHP (Laravel)

The specification is written for Python and .NET, but the challenge is **not** implemented in those languages:

| Layer | Challenge technology |
|---|---|
| Frontend | React |
| Backend | PHP with Laravel |

The business requirements defined by the specification—user stories, acceptance criteria, rules, API contract, and data model—must be preserved. Technology-specific requirements must be translated to React and Laravel.

For the confirmed mapping and unresolved decisions, see the [Laravel and React implementation guide](laravel-react-adaptation.md).

## The assessment is not about writing code

The main purpose is to assess **Spec-Driven Development (SDD)**: how well a specification is read, interpreted, and applied when moving to a different stack. Code is the means, not the end.

## Submission

Only work **pushed to the forks before 5:00 p.m.** (17:00) will be considered. Work submitted after that time will not be reviewed.
There is no extension: push partial progress in small commits during the assessment.

## Repository contents

| Folder | Contents |
|---|---|
| [`spec-python/`](spec-python/) | The Manager Library specification written for a Python backend |
| [`spec-.net/`](spec-.net/) | The same system specified for a .NET backend |

Both versions describe the **same product** (the same user stories, business rules, and endpoints); their technology decisions differ. Neither version uses the challenge stack.

## How to read the specification

In either folder, read the documents in this order:

1. `constitution.md` — non-negotiable principles.
2. `spec.md` — what the system must do: actors, user stories, acceptance criteria, and business rules.
3. `plan.md`, `architecture.md`, and `arquitectura-panoramica.md` — how to build it.
4. `data-model.md` and `api-contract.md` — the data model and API contract.
5. `tasks.md` — the work to do and the evidence required to consider it complete.
6. `adr/` — architecture decisions, including alternatives and consequences.

## The six repositories

| Repository | Purpose |
|---|---|
| [`test-manager-library-3145556-docs`](https://github.com/code-sena/test-manager-library-3145556-docs) | The specification: `spec-python/` and `spec-.net/` |
| [`test-manager-library-3145556-api`](https://github.com/code-sena/test-manager-library-3145556-api) | PHP (Laravel) backend |
| [`test-manager-library-3145556-app`](https://github.com/code-sena/test-manager-library-3145556-app) | React frontend |
| [`test-manager-library-3145556-page`](https://github.com/code-sena/test-manager-library-3145556-page) | Static public presentation site |
| [`test-manager-library-3145556-infra`](https://github.com/code-sena/test-manager-library-3145556-infra) | Containers, network, volumes, and an empty database engine |
| [`test-manager-library-3145556-tool`](https://github.com/code-sena/test-manager-library-3145556-tool) | Utilities: demonstration data seeder |
