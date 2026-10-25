# Examples

Four full before/after runs showing how the same skill adapts to different intents and
target models. Project details below are illustrative.

---

## 1. Bug fix · target: Claude Code (Claude)

**Input**
```text
/crafting-prompt users keep getting logged out randomly, fix it
```

**Context read**: `package.json` (Next.js 14, TypeScript, next-auth 4.24, Vitest),
`src/lib/auth.ts`, `src/middleware.ts`, `CLAUDE.md` (use pnpm, run `pnpm test`).

### Original
> users keep getting logged out randomly, fix it

### Crafted prompt
```text
<context>
Next.js 14 (App Router) + TypeScript app using next-auth 4.24 with JWT sessions.
Auth config: src/lib/auth.ts. Route protection: src/middleware.ts.
Users report being logged out at random times, which interrupts their work and is our
top support complaint.
</context>

<task>
Find the root cause of the unexpected logouts and fix it.
</task>

<approach>
1. Read src/lib/auth.ts and src/middleware.ts. Check session maxAge / updateAge, JWT
   expiry vs. refresh logic, cookie settings (domain, secure, sameSite), and whether the
   middleware treats a token-refresh window as "unauthenticated".
2. Explain the root cause in 2–3 sentences before changing code.
3. Make the smallest change that fixes it.
</approach>

<constraints>
- Don't change the session strategy (keep JWT) or the public auth API.
- Follow the conventions in CLAUDE.md; use pnpm.
</constraints>

<success_criteria>
- A Vitest test reproduces the logout condition and passes after the fix.
- `pnpm test` and `pnpm lint` pass.
</success_criteria>

<output_format>
Root cause → files changed (with one-line reason each) → test added → command output.
If the cause can't be determined from the code, stop and list what logs or repro steps
you need.
</output_format>
```

### What changed
| Dimension | Original | Crafted |
|---|---|---|
| Goal | "fix it" | Find root cause, then minimal fix |
| Context | none | Stack, versions, auth files, why it matters |
| Scope | open | Keep JWT strategy and public API |
| Approach | none | Where to look, explain before editing |
| Success criteria | none | Regression test + test/lint pass |
| Verification | none | `pnpm test`, `pnpm lint` |
| Model tuning | none | XML sections, explicit act-vs-suggest, stop condition |

### Prompt quality score
| | Clarity | Context | Scope | Constraints | Output | Done-when | Verification | Model fit | **Total** |
|---|---|---|---|---|---|---|---|---|---|
| Original | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | **1/16** |
| Crafted | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 | **16/16** |

### Assumptions
- "Randomly" means during active use, not only after long inactivity.

---

## 2. Research · target: OpenAI o3 (reasoning model)

**Input**
```text
/crafting-prompt --target o3 --no-context which message queue should we use, we're
on python and aws, need something reliable
```

### Original
> which message queue should we use, we're on python and aws, need something reliable

### Crafted prompt
```text
# Goal
Recommend a message queue for a Python service running on AWS.

# Requirements
- Reliability is the top priority: at-least-once delivery, durable storage, dead-letter
  handling.
- Python client library that is mature and actively maintained.
- Prefer managed AWS services or options that run well on AWS.

# Compare
Amazon SQS, Amazon SNS+SQS, Amazon MQ (RabbitMQ), Amazon MSK (Kafka), Redis Streams
(ElastiCache).

# Evaluate on
Delivery guarantees, ordering, throughput limits, operational overhead, cost at low and
high volume, Python ecosystem support.

# Output format
1. Comparison table (one row per option, one column per criterion).
2. Recommendation for our case, with the two main reasons.
3. When the recommendation would change (e.g. strict ordering, >10k msg/s, replay needs).
4. Open questions that would change the answer.
```

### What changed
| Dimension | Original | Crafted |
|---|---|---|
| Goal | implied | Explicit recommendation |
| Constraints | "reliable" | Reliability defined in concrete terms |
| Scope | open | Five named candidates |
| Output format | none | Table + recommendation + decision triggers |
| Model tuning | none | Short, goal-first, no step-by-step instructions (reasoning model) |

### Prompt quality score
| | Clarity | Context | Scope | Constraints | Output | Done-when | Verification | Model fit | **Total** |
|---|---|---|---|---|---|---|---|---|---|
| Original | 1 | 1 | 0 | 1 | 0 | 0 | N/A | 0 | **3/14** |
| Crafted | 2 | 2 | 2 | 2 | 2 | 2 | N/A | 2 | **14/14** |

### Assumptions
- Volume is unknown; the prompt asks for low- and high-volume cost.

---

## 3. Roadmap · target: Gemini

**Input**
```text
/crafting-prompt --target gemini we need a roadmap to move our monolith to microservices
```

**Context read**: `pom.xml` (Java 17, Spring Boot 3.2, PostgreSQL), top-level packages
`billing`, `catalog`, `orders`, `users`, `notifications`.

### Original
> we need a roadmap to move our monolith to microservices

### Crafted prompt
```text
## Current system
- Java 17, Spring Boot 3.2 monolith, single PostgreSQL database.
- Top-level modules: billing, catalog, orders, users, notifications.

## Goal
A phased roadmap to move from this monolith to microservices without downtime.

## Constraints
- Incremental migration (strangler-fig style); the monolith keeps running throughout.
- Each phase must deliver something deployable on its own.
- Assume a small team (3–5 engineers); flag phases that need more.

## Based on the system above, produce:
1. Recommended extraction order of the five modules, with the reason for each position
   (coupling, data ownership, business risk).
2. A table of phases: Phase | Goal | Modules | Key work | Dependencies | Risks | Rough effort.
3. Cross-cutting foundations needed before phase 1 (API gateway, service discovery,
   observability, CI/CD, data-splitting strategy).
4. Exit criteria for each phase.
5. Assumptions and open questions.

Output format: Markdown headings and tables only.
```

### What changed
| Dimension | Original | Crafted |
|---|---|---|
| Context | none | Stack, DB, real module names from the codebase |
| Constraints | none | Zero downtime, incremental, team size |
| Output format | "a roadmap" | Ordered sections + phase table |
| Done when | none | Exit criteria per phase |
| Model tuning | none | Context first, instructions at the end, explicit format prefix |

### Prompt quality score
| | Clarity | Context | Scope | Constraints | Output | Done-when | Verification | Model fit | **Total** |
|---|---|---|---|---|---|---|---|---|---|
| Original | 1 | 0 | 1 | 0 | 0 | 0 | N/A | 0 | **2/14** |
| Crafted | 2 | 2 | 2 | 2 | 2 | 2 | N/A | 2 | **14/14** |

### Assumptions
- Zero downtime is required; team size is 3–5 engineers.

---

## 4. Code cleanup · target: local Llama (open-weight)

**Input**
```text
/crafting-prompt --target llama --short clean up utils.py its a mess
```

**Context read**: `pyproject.toml` (Python 3.11, ruff, pytest), `app/utils.py` (480 lines).

### Original
> clean up utils.py its a mess

### Crafted prompt
```text
Refactor the Python 3.11 file app/utils.py below. Behaviour must stay exactly the same.

Rules:
1. Remove unused functions and imports.
2. Merge duplicated logic into shared helpers.
3. Split functions longer than 40 lines.
4. Add type hints and a one-line docstring to every public function.
5. Keep all public function names and signatures unchanged.
6. Follow ruff defaults (PEP 8, max line length 88).

Output format:
- First, a bullet list of changes (one line each).
- Then the full refactored file in one ```python block.

File:
"""
<paste app/utils.py here>
"""

Remember: behaviour and public signatures must not change.
```

### What changed
| Dimension | Original | Crafted |
|---|---|---|
| Goal | "clean up" | Six concrete refactoring rules |
| Constraints | none | Behaviour and signatures preserved |
| Output format | none | Change list + full file |
| Model tuning | none | Flat numbered rules, key rule repeated at the end, inline file |

### Prompt quality score
| | Clarity | Context | Scope | Constraints | Output | Done-when | Verification | Model fit | **Total** |
|---|---|---|---|---|---|---|---|---|---|
| Original | 1 | 0 | 1 | 0 | 0 | 0 | 0 | 0 | **2/16** |
| Crafted | 2 | 2 | 2 | 2 | 2 | 1 | 1 | 2 | **14/16** |
