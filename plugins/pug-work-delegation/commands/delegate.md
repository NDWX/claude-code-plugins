---
description: Delegate a task across subagents — route each part to the cheapest tier that can do it well, then check the evidence that comes back
argument-hint: [what to delegate]
disable-model-invocation: true
---

Invoke the `pug-work-delegation:delegating-work` skill and follow its routing rules. They are
the source of truth for tier selection, briefing, and evidence — do not restate or re-derive them
here.

**What to delegate:** $ARGUMENTS

If that is empty, delegate the work currently under discussion. If no such work is identifiable,
ask what to delegate rather than guessing — one question, then stop.

Typing this command is the user asking to delegate, so route and spawn rather than relitigating
the decision. Two things still apply from the skill:

- A part that fails its "delegate when" criteria — a single known file, one search, work whose
  briefing costs as much as the work — is done inline instead. Say so in one line; do not spawn an
  agent to look busy.
- If the whole remaining phase is execution and this context does not need protecting, say that
  `/model sonnet` is the better lever and let the user choose. Only they can switch the session
  model.

Before spawning, state the split in a few lines: which part goes to which agent, and what each
returns as evidence. Then spawn — independent parts in a single message so they run in parallel,
dependent parts only once their input exists.

When the reports come back, apply the skill's verification rules: evidence over claims, `git diff`
as ground truth for `executor`, and read the **DECISIONS** list as the audit surface. Follow up on
a single questionable line with `SendMessage` to that agent rather than asking for full rationale.
