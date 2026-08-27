# pug-work-delegation — development context

Delegation routing for Claude Code: one command, one task-triggered skill, three model-pinned
agents.
This file records the constraints that are **not** recoverable from reading the files, and the
mistakes that produced them. Read it before editing anything here.

## What is here

```
.claude-plugin/plugin.json          name/version/description; name is the namespace prefix
commands/delegate.md                explicit entry point — defers to the skill, never restates it
skills/delegating-work/SKILL.md     routing rules — loaded when the user asks to delegate
agents/scout.md                     haiku · read-only evidence
agents/executor.md                  sonnet · scoped implementation
agents/deep-work.md                 opus  · hard technical work + review
README.md                           user-facing: what it does, install, design summary
```

## The core distinction this plugin exists to encode

Two levers get conflated constantly. Keep them separate in every edit:

- **`/model` controls what a turn costs.** It does **not** partition context — there is one
  conversation transcript, and switching only changes which model reads it. Everything done in a
  session stays in that window for every later turn.
- **Delegation controls what accumulates.** A subagent's tool calls happen in its own context;
  only its report comes back. This is the *only* mechanism that keeps work out of the transcript.

Corollary, and the plugin's central rule: delegation is worth it in proportion to
*(how expensive the current context is)* × *(how bulky the output would be)*. Both terms describe
the window you are in right now — not what ran in an earlier phase, and not which model is active.

**Delegation is not a cost control.** A subagent starts cold and re-derives context, so for work
the parent could do in a couple of tool calls, delegating costs more. If a change to this plugin
is motivated by "save money on cheap tasks", it is aimed at the wrong lever — that is `/model`,
which is a user action this plugin cannot perform and must never pretend to.

## Invariants — breaking these is how the previous version failed

This plugin replaces a skill (`fable-chief-agent`) that did not work. Each rule below fixes a
specific defect in it:

1. **The skill's `description` must be task-shaped, never model-shaped.** Skills are selected by
   matching the description against the *task*. The predecessor said "Use when the active agent is
   Fable 5" — nothing in a user prompt ever matches that, so it could not fire reliably.

2. **Never assert model identity in a skill.** The predecessor opened with "You are Fable 5, the
   senior decision-maker." A skill cannot detect which model loaded it, so that text lies to
   whatever model reads it. Agent frontmatter (`model:`) is the only honest place to pin a model.

3. **Nothing here may instruct proactive delegation.** The harness carries a standing default not
   to spawn agents unless the user asks. A skill telling the model to fan out on its own puts it in
   direct conflict with that. The skill only fires *after* the user asks to delegate — it supplies
   routing for a decision already made, and must stay that way. `commands/delegate.md` sets
   `disable-model-invocation: true` for the same reason: it makes the command reachable only by the
   user typing it, so the model cannot route itself into a fan-out. Do not remove that field.

   The command carries no routing rules of its own — it invokes the skill and defers to it. Rules
   duplicated across the two would drift, exactly as the skill and the agents' DECISIONS fields
   already did once.

4. **The skill carries conclusions, not reasoning.** Anything that does not change a choice at the
   moment it is read belongs in this file or the README, not in `SKILL.md`. The test for including
   a line: does it alter a decision at a decision point? A routing bias ("recommend `/model sonnet`
   rather than spawning `executor` when the window does not need protecting") earns its place; the
   argument behind it does not.

5. **`scout` stays read-only.** Tools are `Read, Grep, Glob, Bash` plus the read-only ReSharper MCP
   tools, with an explicit read-only constraint in the body. It is the cheapest tier and reports
   facts only. Do not give it write tools, and do not ask it to make or explain decisions —
   introspection is the weakest capability of the cheapest model, and produces fluent,
   low-information text. Every MCP tool granted to any tier here is a navigation or diagnostic one;
   `rename_symbol`, `apply_quick_fix`, `fix_usings`, `format_file`, and `generate_members` belong in
   no `tools:` list in this plugin, least of all `scout`'s.

6. **DECISIONS stays a bounded list, never a narrative.** `executor` and `deep-work` end reports
   with one line per unspecified choice plus the rejected alternative — no justification.
   Rationale is deliberately excluded for two reasons: narrative is bulk, which re-imports exactly
   what delegation was meant to keep out; and post-hoc explanation from a model reads as more
   reliable than it is, raising a reader's confidence without raising accuracy. Reasoning is
   available on demand — agent context stays live, so `SendMessage` retrieves it for the one line
   that looks wrong. Do not "improve" this by asking agents to explain themselves up front.

## Namespacing — the sharp edge

The plugin name in `plugin.json` becomes the prefix for every component it ships:
`pug-work-delegation:scout`, `pug-work-delegation:delegating-work`, and so on. The command is
reached as `/delegate` while no other installed plugin claims that name, and as
`/pug-work-delegation:delegate` when one does — so never document the bare form as guaranteed.

**Renaming the plugin breaks every agent reference inside `SKILL.md`.** That has already happened
twice in this repo's history. If the name changes, grep the whole plugin folder for the old name
and update the routing table, the `Agent(subagent_type: ...)` call shape, and the README.

Inside `SKILL.md`, names are fully qualified in the routing table and call shape (where the exact
string matters) and shortened in prose, with one line establishing the mapping. Preserve that
split — fully qualifying every prose mention made the file unreadable.

Keep the name kebab-case. Claude Code tolerates other forms but warns; Claude.ai marketplace sync
rejects them outright.

## Changing things

- **Version lives in two places**: `plugin.json` and the repo's `.claude-plugin/marketplace.json`
  entry. Bump both together or they drift.
- **Agent frontmatter fields**: `name`, `description`, `tools`, `model`. Omit `tools` to inherit
  everything (`deep-work` does). The `description` is what the parent matches against when routing,
  so it must describe the *work*, not the agent's rank.
- **After editing, reinstall to test.** Plugin content is read from the marketplace cache, not
  from this working copy, so edits here are not live until reinstalled. Verify by checking that
  the agent roster lists `pug-work-delegation:*` with the intended tools.
- **The three agents and the skill must agree.** If a DECISIONS field changes in an agent, update
  the "Require evidence" section of `SKILL.md` to match — they drifted once already.

## Non-goals

- Making the model choose its own tier. It cannot; `/model` is a user action.
- Automatic or proactive delegation. See invariant 3.
- Additional tiers. Three is enough, and `deep-work` is already the narrowest — its honest uses are
  isolating a hard sub-problem from an expensive window, and getting an independent model's read on
  risky code.
