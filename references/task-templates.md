# Task templates

Intent-specific checklists. Use the checklist for the primary intent to find gaps, then
add the listed blocks to the prompt. Not every item applies to every request — include
what adds information.

---

## Research / investigation

- **Question**: the exact question(s) to answer, and the decision the answer feeds into.
- **Scope**: options to compare, time frame, ecosystem/versions, what's out of scope.
- **Criteria**: how to evaluate (performance, cost, licence, maturity, community, fit
  with current stack).
- **Sources**: preferred sources (official docs, benchmarks), recency requirement, ask
  for citations.
- **Output**: comparison table + recommendation + trade-offs + confidence/unknowns.
- **Done when**: a clear recommendation with reasons, or a stated "insufficient
  evidence" with what's missing.

## New feature / new task

- **User story**: who needs it and why; the behaviour from the user's point of view.
- **Acceptance criteria**: concrete, testable statements.
- **Where**: modules/files to touch, existing patterns to follow (point at a similar
  feature).
- **Interfaces**: API shape, data model changes, config, feature flags.
- **Non-goals**: what not to build now.
- **Edge cases**: empty input, errors, permissions, concurrency, i18n.
- **Verification**: tests to add, commands to run, manual check.

## Bug fix

- **Symptom**: what happens vs. what should happen.
- **Reproduction**: steps, input, environment, frequency.
- **Evidence**: error message, stack trace, logs (pasted inside delimiters).
- **Suspected area**: files/functions, recent changes.
- **Fix expectations**: find the root cause, not a symptom patch; smallest safe change.
- **Regression guard**: add a test that fails before and passes after.
- **Done when**: reproduction no longer fails, test suite passes.

## Refactor / code cleanup

- **Target**: files/modules and the specific smells (duplication, long functions, dead
  code, naming, coupling).
- **Invariant**: behaviour must not change; public APIs preserved (or list allowed
  changes).
- **Style**: project conventions, linter/formatter config.
- **Safety net**: run existing tests before and after; add characterization tests if
  coverage is thin.
- **Size**: incremental steps / small commits; no unrelated changes.
- **Output**: summary of changes and why each improves the code.

## Optimization

- **Metric**: what to improve (latency p95, throughput, memory, bundle size, query time,
  cost) and the current baseline.
- **Target**: the goal number or relative improvement.
- **Where**: hot paths, endpoints, queries; profiling data if any.
- **Method**: measure first, then change; one change at a time with before/after numbers.
- **Constraints**: correctness, readability, no new dependencies (if relevant).
- **Done when**: target met and verified with a benchmark/profile; tests pass.

## Roadmap / planning

- **Vision/goal**: the outcome and the time horizon.
- **Current state**: what exists today (from the codebase/context).
- **Constraints**: team size, deadlines, budget, dependencies, tech stack.
- **Structure**: phases/milestones with goals, deliverables, dependencies, risks, and
  rough effort.
- **Prioritization**: the method (impact vs. effort, MoSCoW, risk-first).
- **Output**: phased table or checklist; open questions and assumptions.

## Code review / audit

- **Target**: diff, PR, files or module.
- **Focus**: correctness, security, performance, readability, tests — in priority order.
- **Standards**: project conventions, relevant guidelines (OWASP, accessibility).
- **Output**: findings ranked by severity with file:line, why it matters, suggested fix;
  separate "must fix" from "nice to have".
- **Noise control**: skip pure style nits unless asked.

## Tests

- **Target**: units/functions/endpoints to cover.
- **Type**: unit, integration, e2e; framework and file location conventions.
- **Cases**: happy path, edge cases, error paths, boundaries.
- **Rules**: no network/real DB unless integration; use existing fixtures/factories.
- **Done when**: tests pass, cover the listed cases, and fail if the behaviour breaks.

## Documentation

- **Audience**: new contributor, end user, API consumer, ops.
- **Scope**: what to document; what exists already.
- **Format**: README section, docstrings, ADR, API reference; tone and length.
- **Accuracy**: derive from code; mark anything uncertain.

## Migration / upgrade

- **From → to**: exact versions or technologies.
- **Inventory**: affected files, APIs, config; breaking changes from the changelog.
- **Strategy**: big-bang vs. incremental, compatibility layer, feature flags.
- **Rollback**: how to revert safely.
- **Verification**: tests, build, smoke checks per step.

## Explanation / learning

- **Subject**: code, concept or system to explain.
- **Audience level**: beginner / intermediate / expert in this stack.
- **Depth and format**: overview vs. line-by-line, diagrams, analogies, examples.
- **Check**: end with a short summary or questions to confirm understanding.

---

## Mixed intents

When a request mixes intents (e.g. "investigate why it's slow and fix it"), order them as
phases in the prompt — investigate → propose → implement → verify — and add a checkpoint
between investigation and implementation for coding agents.
