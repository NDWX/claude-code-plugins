# claude-code-plugins

Claude Code plugins, skills, and agents for Pug. The repository is itself a plugin
marketplace — `pug-claude-plugins` — so every plugin below installs from one source.

## Layout

```
.claude-plugin/marketplace.json     # marketplace manifest; lists every plugin
plugins/<plugin-name>/
  .claude-plugin/plugin.json        # that plugin's own manifest
  skills/<skill-name>/SKILL.md
  agents/<agent-name>.md
  commands/<command-name>.md        # optional; slash commands
  hooks/hooks.json                  # optional
  .mcp.json                         # optional; MCP servers bundled with the plugin
  README.md
```

## Plugins

| Plugin | What it does |
|---|---|
| [`pug-work-delegation`](plugins/pug-work-delegation) | Delegation routing: a `/delegate` command and a task-triggered skill for deciding what to delegate and to which tier, plus three model-pinned agents (`scout`/haiku, `executor`/sonnet, `deep-work`/opus) that return bounded, auditable evidence. |

## Install

```
/plugin marketplace add <owner>/claude-code-plugins
/plugin install <plugin-name>@pug-claude-plugins
```

The marketplace registers under its manifest name, `pug-claude-plugins`, which is what
every install command references and what `/plugin marketplace list` shows. That name is
independent of the repository and the owner, so both can change without breaking an
install command.

## Adding a plugin

1. Create `plugins/<plugin-name>/` with a `.claude-plugin/plugin.json` (`name`, `description`, `version`, `author`, `keywords`).
2. Add `skills/`, `agents/`, and `hooks/` as needed.
3. Add an entry to `.claude-plugin/marketplace.json` with `"source": "./plugins/<plugin-name>"`.
4. Add a `CLAUDE.md` in the plugin folder recording the constraints and rationale that the files themselves do not show.
5. Add a row to the table above.

Keep plugin names kebab-case — lowercase letters, digits, and hyphens. Claude Code
tolerates other forms, but Claude.ai marketplace sync rejects them. The name becomes the
namespace prefix for that plugin's skills and agents (`<plugin-name>:<agent-name>`), so
changing it later breaks every reference in that plugin's own skill files.
