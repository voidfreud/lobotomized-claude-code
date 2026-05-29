<!--
name: 'Inline blob: skill settings file locations'
description: 'update-config skill: settings file locations doc.'
inlineBlobAnchor: '[$\w]+=`## Settings File Locations'
inlineBlobKind: 'template'
injectionGate: '/update-config skill'
ccVersion: '2.1.138'
shadows:
  - skill-update-config-settings-file-locations
-->
## Settings File Locations

| File | Scope | Git | Use For |
|---|---|---|---|
| `~/.claude/settings.json` | Global | N/A | Personal prefs across projects |
| `.claude/settings.json` | Project | Commit | Team-wide hooks, permissions, plugins |
| `.claude/settings.local.json` | Project | Gitignore | Personal overrides for this project |

Load order: user → project → local (later overrides earlier).

## Schema

### Permissions
```json
{
  "permissions": {
    "allow": ["Bash(npm *)", "Edit(.claude)", "Read"],
    "deny": ["Bash(rm -rf *)"],
    "ask": ["Write(/etc/*)"],
    "defaultMode": "default" | "plan" | "acceptEdits" | "dontAsk",
    "additionalDirectories": ["/extra/dir"]
  }
}
```

Rule syntax: exact (`Bash(npm run test)`), prefix wildcard (`Bash(git *)`), or tool-only (`Read`).

### Other top-level keys

- `env`: `{"DEBUG": "true", ...}`
- `model` / `agent` / `alwaysThinkingEnabled`
- `attribution`: `{"commit": "...", "pr": "..."}` (empty string hides)
- `enableAllProjectMcpServers`, `enabledMcpjsonServers`, `disabledMcpjsonServers`
- `enabledPlugins`: `{"name@source": true}` — source ∈ `claude-code-marketplace`, `claude-plugins-official`, `builtin`
- `language`, `cleanupPeriodDays` (default 30), `respectGitignore` (default true)
- `spinnerTipsEnabled`, `spinnerVerbs`, `spinnerTipsOverride`, `syntaxHighlightingDisabled`
