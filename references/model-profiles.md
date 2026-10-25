# Model profiles

Each profile tells you how to shape the prompt for a model family. Apply the **Universal**
rules first, then the family-specific ones. Model families evolve quickly; these profiles
capture guidance that has held across recent generations. When the vendor's current
prompting guide disagrees, follow the vendor (links at the bottom).

---

## Universal rules (every model)

- One clear objective, stated early.
- Concrete nouns: file paths, function names, versions, numbers.
- Separate instructions from data with delimiters, so pasted code or logs are never
  mistaken for instructions.
- Say what *to* do; pair every "don't" with the preferred alternative.
- Explain *why* behind unusual constraints.
- Specify the output format explicitly.
- Define "done" in checkable terms.
- No filler, no all-caps shouting, no threats or rewards.

---

## Claude (Anthropic) — Opus, Sonnet, Haiku

**Structure**
- Use **XML tags** to separate parts: `<context>`, `<task>`, `<constraints>`,
  `<files>`, `<output_format>`, `<success_criteria>`. Tag names are free-form; be
  consistent and refer to them by name ("using the code in `<files>`").
- Put **long documents, code and logs at the top**, and the task/question after them.
- Wrap multiple documents as `<document><source>path</source><content>…</content></document>`.

**Instructions**
- Recent Claude models follow instructions precisely and literally. Ask explicitly for
  everything you want ("include edge-case tests", "go beyond the minimum"); don't
  expect it to infer extras.
- Give the motivation behind rules ("keep responses short because they're read on a
  phone") — Claude generalises well from reasons.
- Say whether to **act** or **suggest**: "make the changes" vs. "propose changes, don't
  edit".
- For complex tasks, invite planning: "Think through the approach before writing code."
- Examples work well; wrap them in `<example>` tags and make them varied.

**Avoid**
- Excessive emphasis (CAPS, "CRITICAL", "MUST") — modern Claude over-applies it.
- Vague negatives without an alternative.

**Skeleton**
```text
<context>
Project: … (language, framework, versions)
Why this matters: …
</context>

<files>
…relevant paths or pasted code…
</files>

<task>
…one clear objective…
</task>

<constraints>
- …
</constraints>

<output_format>
…
</output_format>

<success_criteria>
- …
</success_criteria>
```

---

## OpenAI GPT (GPT-4.1, GPT-4o, GPT-5 and later, non-reasoning use)

**Structure**
- Use **Markdown headings** (`# Role`, `# Task`, `# Context`, `# Constraints`,
  `# Output format`) and fenced blocks or `"""` / XML for data.
- Order that works well: role/objective → instructions → detailed rules → output
  format → examples → context/data. For very long context, repeat the key instruction
  both before and after the context.

**Instructions**
- GPT-4.1 and later follow instructions literally; if behaviour is wrong, a single
  firm clarifying sentence usually fixes it.
- Remove contradictions — conflicting rules waste effort and cause erratic output.
- For agentic use, include three reminders:
  1. *Persistence*: "Keep going until the task is fully resolved before ending your turn."
  2. *Tool use*: "If unsure about file contents or structure, use your tools to read
     them — don't guess."
  3. *Planning*: "Plan before each tool call and reflect on the result."
- Give a short example of the output format when it matters.

**Avoid**
- Ambiguous or competing instructions.
- Overly long role-play preambles.

---

## OpenAI reasoning models (o-series, GPT-5 with high reasoning effort)

**Structure**
- Short and direct. Goal → constraints → output format. Use Markdown or XML delimiters.
- Put the instruction in the developer/system message where available.

**Instructions**
- Do **not** ask it to "think step by step" or explain its reasoning — it reasons
  internally; such prompts can reduce quality.
- Do not prescribe a detailed approach unless the method itself is a requirement; state
  the goal and let it plan.
- Be explicit about success criteria and edge cases — that's where the reasoning goes.
- Try zero-shot first; add examples only if output format is wrong.

**Avoid**
- Long step-by-step procedures, chain-of-thought instructions, redundant context.

---

## Google Gemini (Gemini 2.x, 3.x, Pro and Flash)

**Structure**
- Clear sections with Markdown headings or XML-style tags; be consistent.
- With long context (codebases, documents), place the context **first** and the
  question/instructions **at the end** ("Based on the code above, …").
- Prefix the expected output to steer format ("Output as JSON:", "Table:").

**Instructions**
- Few-shot examples are very effective; show the pattern to follow, not anti-patterns.
- Be explicit about length, tone and format.
- Break complex tasks into ordered sub-tasks or chained prompts.
- For thinking models, state the goal and constraints clearly; step-by-step guidance is
  optional and only needed when the method matters.

**Avoid**
- Mixing examples with different formats.
- Leaving output format implicit.

---

## Open-weight models — Llama, Mistral, Qwen, Gemma, Phi, and other local models

**Structure**
- Short, flat structure: a few Markdown headings or numbered rules. Avoid deep nesting.
- Put the most important instruction **first and again at the end** — smaller models
  lose track of instructions in long prompts.

**Instructions**
- Simple, direct sentences; one instruction per line.
- Always give the exact output format, ideally with one short example.
- Narrow the task: one objective per prompt; split large tasks into a sequence.
- Supply the needed context inline — don't rely on the model's knowledge of niche
  libraries or recent versions.
- State the language and version explicitly (e.g. "Python 3.12, FastAPI 0.115").

**Avoid**
- Long prompts with many interacting constraints.
- Implicit expectations, subtle tone instructions, or references to "the usual way".

---

## DeepSeek reasoning (R1 family) and similar open reasoning models

- Put everything in the **user message**; keep the system prompt empty or minimal.
- Zero-shot; few-shot examples tend to hurt.
- State the goal and the required final-answer format clearly
  (e.g. "Give the final answer in a fenced code block").
- Don't ask for step-by-step reasoning — it already does this.

---

## Coding agents — Claude Code, OpenAI Codex, Cursor, GitHub Copilot, Gemini CLI, Windsurf, Aider

These read files and run commands themselves, so the prompt should steer *work*, not
paste the whole codebase. Combine these rules with the profile of the underlying model.

- **Point, don't paste**: reference files by path (`src/auth/session.ts`), symbols by name.
  Tools that support it can take `@path` mentions.
- **Mode**: say whether to plan first and wait for approval, or implement directly.
  For larger changes: "First explore and propose a plan; don't edit until I approve."
- **Scope fence**: list what must not change (public APIs, migrations, config).
- **Verification**: name the exact commands (`npm test`, `pytest -q`, `go test ./...`,
  linter, type checker) and ask the agent to run them and fix failures.
- **Done when**: tests pass, behaviour X observed, no new warnings.
- **Stop conditions**: when to stop and ask (ambiguous requirements, destructive steps,
  failing tests it can't explain).
- **Conventions**: refer to the project's instructions file (`CLAUDE.md`, `AGENTS.md`,
  `.cursorrules`) instead of restating it.
- **Commits**: say whether to commit, and how (one commit, conventional message), or not
  to commit at all.

---

## Generic (unknown target)

Use Markdown headings with XML-style tags only around pasted data. Order: Context → Task →
Constraints → Output format → Done when. Keep it moderately short, include one example of
the output format when format matters, and avoid model-specific tricks. This works
acceptably on every model above.

---

## Vendor guides

- Anthropic: https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/overview
- OpenAI: https://platform.openai.com/docs/guides/prompt-engineering
- Google Gemini: https://ai.google.dev/gemini-api/docs/prompting-strategies
- Meta Llama: https://www.llama.com/docs/how-to-guides/prompting/
- Mistral: https://docs.mistral.ai/guides/prompting_capabilities/
