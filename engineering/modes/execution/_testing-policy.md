# Testing Policy (Mandatory For Code Changes)

> **Load When**: Implementation, BugFix, Testing modes

Whenever you write or modify code in Implementation or BugFix:

## Requirements

- Any behavior change must include new or updated unit tests.
- Where infra exists and it makes sense:
  - Add or update integration/API tests
  - Add or update E2E tests for critical flows

## Test Derivation

Derive tests from:
- Main user flows
- Business rules and validations
- Edge cases and error scenarios
- Known regressions and bug reports

## Conventions

Use existing setups and conventions:
- Frontend: Jest + React Testing Library or current setup
- Backend: JUnit/Mockito, Jest, or current setup
- Follow existing file naming and folder structure

## UI Test Coverage

UI tests should cover:
- Main states (normal, loading, error, empty)
- Key interactions
- Presence of critical elements and messages

## Documentation

After showing tests, add a short note:
- Which behaviors and regressions they protect.
