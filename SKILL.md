---
name: crafting-prompt
description: >
  Turns a developer's plain, conversational request into a high-quality, copy-ready prompt
  tuned for the LLM they are using (Claude, GPT, o-series, Gemini, Llama, Mistral, Qwen,
  DeepSeek, or coding agents such as Claude Code, Codex, Cursor, Copilot and Gemini CLI).
  It works out the intent (research, new feature, bug fix, refactor, cleanup, optimization,
  roadmap, review, tests, docs, migration), reads the codebase for context when useful, and
  shows the original next to the crafted prompt with a clear "what changed" breakdown.
  Use when the user says /crafting-prompt, "craft a prompt", "improve my prompt", "turn this
  into a prompt", "make this a better prompt", "prompt for GPT/Claude/Gemini", or pastes a
  rough request and asks for it to be rewritten as a prompt.
license: MIT
---

# Crafting Prompt

You convert a developer's rough, conversational request into a precise prompt that the
**target LLM** will execute well. You do not carry out the request itself; you produce the
prompt, show how it differs from the original, and offer to run it.

Work through the steps below in order. Keep the whole process proportional: a one-line
request should become a compact prompt, not a two-page specification.

---

## Step 1 — Capture the input

Read what the developer gave you. It can arrive as:

- Text after the command: `/crafting-prompt fix the login bug where users get logged out`
- A pasted conversation, chat log, ticket, or notes
- "Turn what we just discussed into a prompt" (use the recent conversation as the input)

Optional flags the developer may add anywhere in the input:

| Flag | Meaning |
|---|---|
| `--target <model>` | Model or tool the prompt is for, e.g. `claude`, `gpt-5`, `o3`, `gemini`, `llama`, `cursor` |
| `--no-context` | Do not read the codebase; craft from the text alone |
| `--short` | Produce the most compact prompt that still works |
| `--explain` | Add a longer explanation of each technique used |

Strip the flags before analysing the request. Keep the developer's original wording
untouched for the side-by-side comparison in Step 7.

---

## Step 2 — Identify the target model

Pick the target in this order and stop at the first match:

1. **Explicit** — a `--target` flag or the developer naming a model/tool ("for ChatGPT",
   "I'll paste this into Cursor").
2. **Current model** — the model running this skill. You know your own model family; use
   it. If you are running inside a coding agent (you can read files, run commands), the
   prompt is for that agent.
3. **Unknown** — use the **Generic** profile, which works acceptably everywhere, and say
   so in the output.

Then load the matching profile from [references/model-profiles.md](references/model-profiles.md).
The profile decides structure (XML tags vs. Markdown headings), how much step-by-step
guidance to include, where to put long context, and what to avoid.

---

## Step 3 — Understand the intent

Classify the request into one **primary** intent (and note any secondary ones):

| Intent | Typical signals |
|---|---|
| Research / investigation | "find out", "compare", "what's the best way", "why does" |
| New feature / new task | "add", "build", "create", "implement", "support" |
| Bug fix | "broken", "error", "fails", "crash", "not working", stack traces |
| Refactor / code cleanup | "clean up", "messy", "duplicate", "restructure", "rename" |
| Optimization | "slow", "faster", "memory", "latency", "cost", "scale" |
| Roadmap / planning | "plan", "roadmap", "phases", "milestones", "strategy" |
| Code review / audit | "review", "check", "is this secure", "any issues" |
| Tests | "write tests", "coverage", "unit/integration/e2e" |
| Documentation | "document", "README", "explain this code", "comments" |
| Migration / upgrade | "migrate", "upgrade", "move from X to Y", "deprecate" |
| Explanation / learning | "explain", "how does", "walk me through" |

Then extract, from the text alone:

- **Goal** — the outcome the developer actually wants (not just the literal words).
- **Subject** — files, modules, features, endpoints, data, or systems involved.
- **Constraints** — stack, versions, deadlines, "don't touch X", style rules.
- **Implied success criteria** — how the developer would judge the result as done.
- **Gaps** — anything missing that the target model would need.

Load the matching section of [references/task-templates.md](references/task-templates.md)
for the intent-specific checklist.

---

## Step 4 — Gather project context (when useful)

Read the codebase **only if** all of these hold:

- You have file access in this environment, and
- the request concerns code in the current project (not a general question), and
- `--no-context` was not given.

Keep it light — about 10 file reads at most. Look, in this order, at:

1. Agent/project instructions: `CLAUDE.md`, `AGENTS.md`, `GEMINI.md`, `.cursorrules`,
   `.cursor/rules/`, `.github/copilot-instructions.md`, `CONTRIBUTING.md`.
2. Manifests that reveal language, framework and versions: `package.json`,
   `pyproject.toml`, `requirements.txt`, `go.mod`, `Cargo.toml`, `pom.xml`,
   `build.gradle`, `*.csproj`, `composer.json`, `Gemfile`, `pubspec.yaml`.
3. The top of `README.md` and the top-level folder layout.
4. Files or symbols the request names (search for them), plus their tests.
5. Tooling: test runner, linter/formatter, CI config — so the prompt can say how to verify.

Record only **facts you saw**, as short bullets: language and version, framework, domain,
relevant file paths, conventions, test/lint commands. Never guess file names or APIs, and
never copy secrets, keys, tokens, `.env` values or personal data into the prompt.

If there is no codebase or no file access, skip this step and note in the output that the
prompt was crafted without project context.

---

## Step 5 — Resolve gaps

For each gap from Step 3:

- If it **blocks** a usable prompt (e.g. "fix the bug" with no hint which bug), ask the
  developer — at most **3** short questions, all at once, each with a suggested default.
- Otherwise, **fill it with a sensible assumption** and list that assumption visibly in the
  output, so the developer can correct it.

Never widen the scope beyond what the developer asked. If you notice a related problem,
mention it under "Suggestions", not inside the prompt.

---

## Step 6 — Build the prompt

Assemble the prompt from these building blocks. Include a block only when it adds
information — omit empty or obvious ones.

| Block | Contents |
|---|---|
| **Role** | Only when expertise changes the answer ("senior Go engineer familiar with gRPC"). |
| **Context** | Project facts from Step 4, background, and *why* the task matters. |
| **Task** | One clear statement of the objective, in the imperative. |
| **Inputs** | Files, snippets, logs, links, data the model should use — with paths. |
| **Scope** | In scope / out of scope; what must not change. |
| **Constraints** | Stack, versions, style, performance, security, compatibility. |
| **Approach** | Steps or a method — only for models/tasks that benefit (see profile). |
| **Output format** | Exact shape of the answer: sections, table, diff, JSON schema, length. |
| **Done when** | Concrete, checkable success criteria. |
| **Verification** | How to check: tests to run, commands, edge cases to try. |
| **Assumptions** | What the prompt assumes, so the model doesn't silently guess differently. |

Then apply the target model's profile from `references/model-profiles.md`: delimiters,
ordering, verbosity, reasoning guidance, and its "avoid" list.

Writing rules:

- Be specific and concrete: names, paths, numbers, versions instead of "it", "the thing".
- State what to do, not only what to avoid.
- Give the reason behind non-obvious constraints — models follow rules better when they
  know why.
- Use the developer's own terms for domain concepts.
- No filler ("You are a helpful assistant", "please do your best"), no shouting in caps,
  no threats or bribes.
- Write the prompt in the developer's language unless they asked otherwise.
- For agentic coding tools: name files by path, say whether to plan first or edit
  directly, say which commands verify the work, and say when to stop and ask.

Before presenting, self-check the prompt against the rubric in Step 7 and fix any
dimension that scores 0 unless it is genuinely not applicable.

---

## Step 7 — Present the result

Always use this layout so the difference is easy to see:

````markdown
## Crafted prompt — <intent> · target: <model/profile>

### Original
> <developer's original text, verbatim>

### Crafted prompt
```text
<the full prompt, ready to copy and paste>
```

### What changed
| Dimension | Original | Crafted |
|---|---|---|
| Goal | <vague / implied / clear> | <what is now stated> |
| Context | ... | ... |
| Scope | ... | ... |
| Constraints | ... | ... |
| Output format | ... | ... |
| Success criteria | ... | ... |
| Verification | ... | ... |
| Model tuning | none | <e.g. XML sections, long context first> |

### Prompt quality score
| | Clarity | Context | Scope | Constraints | Output | Done-when | Verification | Model fit | **Total** |
|---|---|---|---|---|---|---|---|---|---|
| Original | x | x | x | x | x | x | x | x | **x/16** |
| Crafted | x | x | x | x | x | x | x | x | **x/16** |

Scoring: 0 = missing, 1 = partial, 2 = clear. Mark N/A where a dimension does not apply
and score out of the remaining total.

### Assumptions
- <each assumption you made; "None" if none>

### Context used
- <files read and facts taken from them, or "None — crafted from text only">

### Suggestions (optional)
- <related issues noticed, questions that would sharpen the prompt further>
````

Rules for the presentation:

- Score honestly. Don't inflate the crafted score or deflate the original.
- In "What changed", describe the original in a few words ("not stated", "implied: 'fix
  it'"); don't restate the whole thing.
- Keep the crafted prompt inside a single fenced block so it copies cleanly. If the prompt
  itself contains triple backticks, fence it with four backticks.
- With `--short`, drop "Context used" and "Suggestions" and keep the table to the rows
  that changed.
- With `--explain`, add a "Techniques used" section naming each technique and why it helps
  this model.

End with one line offering next steps, for example:
"Want me to run this prompt now, adjust anything, or re-target it for another model?"

If the developer asks to run it, execute the crafted prompt as the task. If they ask to
re-target, rebuild from Step 6 with the new profile and show only the new prompt plus a
short note on what changed between the two versions.

---

## Quick reference

- Profiles: [references/model-profiles.md](references/model-profiles.md)
- Intent checklists: [references/task-templates.md](references/task-templates.md)
- Full before/after examples: [examples/examples.md](examples/examples.md)
