# claude-state-template

A template for persisting Claude Code memory, preferences, and settings across Docker container rebuilds and machines. Fork this repo to create your own private `claude-state` — a git-tracked store for everything Claude should remember about you and your project.

Designed for use with the [PersonaRL](https://github.com/persona-ai-inc/persona_rl) Docker environment, but the structure works with any project where you want to bind-mount Claude's config into a container.

---

## Quick Start

1. **Fork this repo** on GitHub (or click **Use this template** to create a fresh copy). Make it **private** — it will contain your personal preferences and project context.

2. **Clone your fork** alongside `persona_rl`:
   ```bash
   cd ~   # or wherever your workspace lives
   git clone git@github.com:YOUR_USERNAME/claude-state.git
   ```

3. **Set `CLAUDE_STATE_PATH`** in `persona_rl/docker/.env`:
   ```
   CLAUDE_STATE_PATH=/home/<your-username>/AJ/claude-state
   ```

4. **Start the container** with the Claude overlay:
   ```bash
   ./docker/container.sh start --files claude.yaml
   ```

5. **Fill in your context** — see the sections below for how to populate `project/CLAUDE.md` and `project/preferences.md` effectively.

---

## How It Works

The `docker/claude.yaml` overlay bind-mounts two directories from your `claude-state` clone into the container:

| Host path | Container path | Purpose |
|-----------|----------------|---------|
| `claude-state/project/` | `/workspace/isaaclab/.claude/` | Project-level config: CLAUDE.md, memory, settings |
| `claude-state/home/` | `/workspace/isaaclab/.claude-home/` | Claude's home dir, symlinked from `/root/.claude` |

The Dockerfile sets up `ln -sfn /workspace/isaaclab/.claude-home /root/.claude` in the base stage, so Claude's home directory persists across container rebuilds via the bind mount. Changes Claude makes to memory or settings appear on the host immediately.

Without `--files claude.yaml`, the container starts normally but Claude has no persistent state.

---

## Building Your CLAUDE.md

`project/CLAUDE.md` is the most important file in this repo. Claude reads it at the start of every session. A good CLAUDE.md dramatically reduces the time Claude spends asking questions or exploring before it can help you.

**What to put in it:**

| Section | What goes here | Anti-patterns |
|---------|---------------|---------------|
| **Overview** | What the project does, tech stack, key entry points | Marketing copy, history |
| **Key Commands** | Build, test, run, lint — with flags and expected output | Commands that change often |
| **Repository Structure** | Directory layout of the parts Claude will touch | Exhaustive file listings |
| **Architecture** | Design patterns, invariants, conventions | Implementation details |
| **Coding Standards** | Lint config, style rules not captured by tooling | Rules already in linter config |

**Tips:**
- Be specific. "We use Ruff with 120-char line length" is better than "we have a linter."
- Include gotchas. If there's something that's easy to get wrong (import order, a required decorator, a file to never modify), say so explicitly.
- Keep it current. An outdated CLAUDE.md is worse than none — Claude will follow stale instructions confidently. Update it when the architecture changes.
- Link to additional context files. For domain-heavy subsystems (calibration pipelines, custom ML frameworks), create a separate `.md` file in `project/` and link to it from the Reference Files table in CLAUDE.md.

---

## Building Your preferences.md

`project/preferences.md` is for rules about how you collaborate — not technical facts about the code. Claude reads it alongside CLAUDE.md.

**What to put in it:**

- **Workflow rules**: Who does what? If you run experiments on a remote machine and Claude should only design them (not run them), say so here. If you review all commits before pushing, say so. These boundaries prevent Claude from doing things you wanted to do yourself.
- **Communication style**: How verbose? Bullet points or prose? Skip trailing summaries? Prefer questions up front vs. just attempting things? Your preferences here save you from having to correct Claude repeatedly.
- **Hard constraints**: Things that are easy to get wrong in your project. "Never edit files in `src/upstream/`." "Always ask before creating new files." These are the rules you've had to repeat in the past.

**The goal**: Claude should be able to operate without you needing to give the same correction twice. If you find yourself saying the same thing across multiple sessions, it belongs in `preferences.md`.

---

## Settings Guide

Claude Code uses a layered settings system. This repo provides two of the four layers:

| Layer | File | Scope | Committed? |
|-------|------|-------|-----------|
| Global user | `home/settings.json` → `/root/.claude/settings.json` | All projects | Yes (in this repo) |
| Global local | `home/settings.local.json` | All projects, this machine only | No (gitignored) |
| Project | `project/settings.json` → `<workspace>/.claude/settings.json` | This project | Yes (in this repo) |
| Project local | `project/settings.local.json` | This project, this machine only | No (gitignored) |

Lower layers override higher ones. Use `project/settings.json` for project-specific permissions (e.g., allowing reads into a large external path). Use `home/settings.json` for your baseline preferences that apply everywhere.

### Default (Conservative) — `home/settings.json`

The template ships with a conservative default: Claude auto-approves read-only tools and asks before making any changes. Good starting point if you're new to Claude Code.

```json
{
  "permissions": {
    "allow": ["Read", "Glob", "Grep"],
    "ask":   ["Edit", "Write", "Bash", "WebFetch", "WebSearch"],
    "deny":  [
      "Bash(git push --force:*)",
      "Bash(git reset --hard:*)",
      "Bash(git checkout -- .:*)",
      "Bash(git clean:*)"
    ]
  }
}
```

**What this means in practice:** Claude will ask for your approval before editing any file or running any shell command. You'll see a lot of prompts, but nothing happens without your say-so.

### Advanced Config (AJ's Setup)

Once you're comfortable, you can grant more permissions so Claude can work more autonomously. Here's a more permissive config that auto-approves file operations and most read-only git commands, but still asks for anything that changes git history:

```json
{
  "permissions": {
    "allow": [
      "Read", "Edit", "Write", "Glob", "Grep", "Bash", "LSP",
      "Bash(git status:*)",
      "Bash(git diff:*)",
      "Bash(git log:*)",
      "Bash(git show:*)",
      "Bash(git branch:*)",
      "Bash(git fetch:*)",
      "Bash(git remote:*)",
      "Bash(git rev-parse:*)"
    ],
    "ask": [
      "Bash(git:*)"
    ],
    "deny": [
      "Bash(git push --force:*)",
      "Bash(git reset --hard:*)",
      "Bash(git checkout -- .:*)",
      "Bash(git clean:*)",
      "Bash(docker:*)",
      "Bash(curl:*)",
      "Bash(wget:*)"
    ]
  },
  "model": "opus[1m]",
  "enabledPlugins": {
    "pyright-lsp@claude-plugins-official": true
  },
  "extraKnownMarketplaces": {
    "claude-plugins-official": {
      "source": {
        "source": "github",
        "repo": "anthropics/claude-plugins-official"
      }
    }
  }
}
```

**Key differences from the conservative config:**
- `Edit`, `Write`, and `Bash` are auto-allowed — Claude can modify files and run commands without prompting
- Read-only git commands (`status`, `diff`, `log`, etc.) are auto-allowed
- Write git commands still ask (commits, checkouts, merges, stashes, pushes)
- Destructive git operations and network tools are denied entirely
- Adds the `pyright-lsp` plugin for Python type-checking (remove if not relevant)
- Sets the model to `opus[1m]` (Claude Opus 4 with extended context)

**Additional directories:** If Claude needs to read files outside the workspace root (e.g., a large external library at a fixed path), add them to `additionalDirectories`:

```json
"additionalDirectories": [
  "/path/to/external/library"
]
```

---

## Memory System

Claude Code automatically builds up memory files in `home/projects/<project-key>/memory/` as it works with you. These persist across container restarts and machines because they're tracked in this repo.

**Four types of memory:**

| Type | What it stores | Examples |
|------|---------------|---------|
| `user` | Your role, expertise, how you like to collaborate | "Senior ML engineer, new to this codebase's frontend" |
| `feedback` | Rules about approach — corrections and validated choices | "Don't mock the DB in tests — prod divergence burned us before" |
| `project` | Ongoing work, decisions, bugs, context not in code/git | "Merge freeze starts 2026-03-15 for mobile release cut" |
| `reference` | Pointers to external systems | "Pipeline bugs tracked in Linear project INGEST" |

Memory is auto-saved when Claude learns something worth keeping. It's indexed in `home/projects/<key>/memory/MEMORY.md`. You don't need to manage it manually — just commit and push when you want to sync to other machines.

---

## What's Tracked vs. Ignored

| Tracked (committed) | Ignored (gitignored) |
|--------------------|---------------------|
| `project/CLAUDE.md`, `preferences.md`, `settings.json` | Credentials (`home/.credentials.json`, OAuth tokens) |
| `home/settings.json` (global Claude settings) | Sessions, file history, shell snapshots |
| `home/projects/*/memory/` (persistent memory files) | Plugins (re-downloaded automatically) |
| | Per-session plans, subagent threads, env snapshots |

---

## Syncing Across Machines

Memory files and settings are written directly to the bind mount, so changes appear on the host immediately. Commit and push to sync to other machines:

```bash
cd ~/AJ/claude-state
git add -A && git commit -m "update memories" && git push
```

On another machine, pull before starting a session:

```bash
cd ~/AJ/claude-state && git pull
```

---

## PersonaRL Integration Reference

The `docker/claude.yaml` overlay in `persona_rl` handles the bind mounts. It defines volume entries for all profiles (base, shared, persona, ihmc, all) so the state is available regardless of which profile you start.

See the **"Claude Code in the Container"** section of the [PersonaRL README](https://github.com/persona-ai-inc/persona_rl/blob/main/README.md) for the full setup walkthrough.
