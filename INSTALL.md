# my-harness

Backup of Claude Code global skills/commands plus a project's local commands/agents, snapshotted on 2026-08-28.

## Contents

```
global/skills/     <- ~/.claude/skills      (user-level skills)
global/commands/   <- ~/.claude/commands    (user-level slash commands)
project/commands/  <- a project's .claude/commands  (project-level)
project/agents/    <- a project's .claude/agents    (project-level)
```

## Restore on a new machine

```bash
# global (user-level) skills and commands
cp -r global/skills/.   ~/.claude/skills/
cp -r global/commands/. ~/.claude/commands/

# project-level commands/agents (adjust path to the target repo)
cp -r project/commands/. "<path-to-repo>/.claude/commands/"
cp -r project/agents/.   "<path-to-repo>/.claude/agents/"
```

## Plugins

Marketplace used: `claude-plugins-official` (github: `anthropics/claude-plugins-official`)

```bash
claude plugin marketplace add anthropics/claude-plugins-official

claude plugin install frontend-design@claude-plugins-official   # user scope
claude plugin install skill-creator@claude-plugins-official     # user scope

# project-scoped, only needed inside the HappHabit repo:
# claude plugin install expo@claude-plugins-official
```

## MCP servers (user scope)

```bash
claude mcp add --transport http supabase https://mcp.supabase.com/mcp

claude mcp add --transport http claude-design https://api.anthropic.com/v1/design/mcp

# requires a Stitch API key — replace the placeholder, do not commit the real key
claude mcp add --transport http stitch https://stitch.googleapis.com/mcp \
  --header "X-Goog-Api-Key: <YOUR_STITCH_API_KEY>"

# requires the Spline desktop app installed at the default path below
claude mcp add Spline --env ELECTRON_RUN_AS_NODE=1 -- \
  "C:\Users\223\AppData\Local\Programs\Spline\Spline.exe" \
  "C:\Users\223\AppData\Local\Programs\Spline\resources\spline-mcp.cjs"
```

## Global settings notes (`~/.claude/settings.json`)

Not restored automatically by this repo (kept out on purpose — it's local config, not shareable content). For reference, worth re-applying by hand:

- `env.CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS = "1"`
- `autoUpdatesChannel = "latest"`
- `theme = "dark"`
