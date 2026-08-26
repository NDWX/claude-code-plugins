# pug-work-delegation

Delegation routing for Claude Code.

## What it provides

**Skill — `delegating-work`.** Task-triggered: fires when you ask to delegate, parallelize, fan out, or split work across subagents. It carries the routing rules, not a narrative — when delegating beats doing the work inline, when switching the session model beats delegating, how to brief a cold agent, and what evidence to demand back.

**Agents — three tiers, model-pinned in frontmatter.**

| Agent | Model | Role |
|---|---|---|
| `pug-work-delegation:scout` | haiku | Read-only evidence: locate files, map a code path, summarize a file or log, verify checklist items. Cites `file:line`; never edits, never decides direction. |
| `pug-work-delegation:executor` | sonnet | Scoped implementation against a decided plan. Escalates rather than improvising on ambiguity, architecture, or high-risk areas. |
| `pug-work-delegation:deep-work` | opus | Complex implementation, deep debugging, cross-module and security reasoning, reviewing cheaper agents' output. |

`executor` and `deep-work` end every report with a bounded **DECISIONS** list — each unspecified choice on one line with the alternative rejected, no justification. `deep-work` adds `ASSUMED:` and `REVIEW:` lines. The list is an audit surface, deliberately a pointer rather than an explanation: follow up with `SendMessage` on the one line that looks wrong, rather than paying for rationale on everything up front.

## Design notes

Two levers, often conflated:

- **`/model` controls what a turn costs.** It does not partition context — there is one transcript, and switching only changes which model reads it.
- **Delegation controls what accumulates.** A subagent's tool calls stay in its own context; only its report comes back.

So delegation is worth it in proportion to *(how expensive the current context is)* × *(how bulky the output would be)*. Both terms describe the window you are in now, not what ran in an earlier phase — which is why the same search is not worth delegating from a cheap session and clearly worth it from an expensive one.

The plugin does not, and cannot, switch models for you. That stays a user action.

## Install

```
/plugin marketplace add ~/dev/pug/Pug.AI.Generative.Agents.ClaudeCode
/plugin install pug-work-delegation@pug-claude-plugins
```

Once pushed, point the marketplace at the remote instead to sync across machines:

```
/plugin marketplace add <git-remote-url>
```

## Naming

Claude Code namespaces plugin components by plugin name, so the agents resolve as
`pug-work-delegation:scout`, `pug-work-delegation:executor`, and `pug-work-delegation:deep-work`.
Keep the plugin name kebab-case — Claude.ai marketplace sync requires it.
