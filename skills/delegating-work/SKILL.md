---
name: delegating-work
description: Use when the user asks to delegate, parallelize, fan out, or split up a large or ambiguous task across subagents, or asks which model tier should handle which part. Covers deciding what is worth delegating, routing each part to the cheapest agent that can do it well, and what evidence to require back.
---

# Delegating work across agents

Delegation buys two things: **parallelism**, and **keeping bulky tool output out of this context**. It does not buy cheaper reasoning — every spawn starts cold and re-derives context, so a task that costs less than its own briefing should just be done directly here.

## Delegating and switching models are different tools

There is one conversation transcript. `/model` changes which model reads it; it does not partition it. Everything done in this session stays in this window on every later turn, whichever model is active.

- To change **what a turn costs** → the user switches the session model. Only they can do that: recommend it, never assume it happened.
- To change **what accumulates in this window** → delegate. A subagent's tool calls happen in its own context; only its report comes back.

So when the whole remaining phase is execution and this window does not need protecting, recommend `/model sonnet` rather than spawning `pug-work-delegation:executor`. Reach for that agent when this context is expensive and must stay clean, or when independent parts should run at once.

## Decide whether to delegate at all

Delegation is worth it in proportion to **(how expensive this context is) × (how bulky the output would be)**. Both terms describe the window you are in right now — not what ran in an earlier phase. The same search that is not worth delegating from a cheap session is clearly worth it from an expensive one.

Delegate when at least one is true:

- the work fans out across many files or directories and only the conclusion matters here
- it will produce a lot of output (logs, large files, full test runs) that would otherwise fill this context
- several independent parts can run at once
- it is long, mechanical, and the result is checkable from evidence

Do it directly when:

- it is a single known file, a small edit, or one search you could run in a couple of commands
- briefing the agent would take about as long as doing the work
- it needs judgment about intent, scope, or tradeoffs — that never delegates well

## Route to the cheapest tier that can do it well

| Work | Agent | Model |
|---|---|---|
| Find files, map a code path, summarize a file or log, verify a checklist item | `pug-work-delegation:scout` | haiku |
| Scoped implementation, tests, local refactors, fixing a diagnosed failure | `pug-work-delegation:executor` | sonnet |
| Complex implementation, deep debugging, cross-module or security reasoning, reviewing cheaper agents' work | `pug-work-delegation:deep-work` | opus |

```
Agent(subagent_type: "pug-work-delegation:scout", description: "...", prompt: "...")
```

Model comes from each agent's definition in this plugin's `agents/` directory, so it does not need restating per call. Pass `model:` only to deliberately override a tier for one call.

These three names are always prefixed `pug-work-delegation:` when calling `Agent`. Below they are shortened to `scout`, `executor`, and `deep-work` for readability.

Independent parts go in a single message so they run in parallel. Parts that feed each other must be sequential — do not spawn a dependent agent before its input exists.

## Brief them properly

Each agent starts cold: it has none of this conversation. A vague prompt is the main reason delegation produces junk that costs more to check than to have done yourself.

Give every agent:

- the concrete goal, and how you will know it succeeded
- the paths, symbols, or commands it should start from
- what is explicitly out of scope
- the form of the answer you want back (file:line citations, a diff, a test result)

## Require evidence, then check it

Take no delegated claim on trust. "Fixed" needs the command and its output; "found it" needs `file:line`. If an agent reports success without evidence, ask for the evidence — do not relay it onward.

Cheaper agents fail in a specific way: they report plausible-sounding conclusions they did not verify. Spot-check anything that matters before it becomes the basis of a decision.

`executor` and `deep-work` end their reports with a **DECISIONS** list: every choice the brief did not specify, one line each, plus `ASSUMED:` and `REVIEW:` lines from `deep-work`. Read it — it is the audit surface, and it is deliberately a pointer rather than an explanation. `DECISIONS: none.` on work that plainly involved discretionary calls is itself a flag.

For `executor`, the diff is ground truth: read `git diff` rather than a narration of it.

Do not ask agents for full rationale up front. Narrative is bulk — it re-imports what delegation was meant to exclude — and post-hoc explanation reads as more reliable than it is. Agent context stays live after the report, so when one DECISIONS line looks wrong, `SendMessage` that agent and ask about that decision alone.

## What stays here

Intent, scope, architecture, tradeoffs, risk, resolving disagreement between agents, deciding when the work is done, and the final answer to the user. High-risk areas — auth, permissions, billing, migrations, data loss, shared state, caching, concurrency, public APIs — are decided here, technically handled or reviewed by `deep-work`, and verified against concrete evidence by `scout`.
