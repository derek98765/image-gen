# image-gen

Generate or edit images within Claude Code by shelling out to a local AI CLI:
the **Codex CLI** (OpenAI `gpt-image-2`, the model behind ChatGPT images) or the
**Grok CLI** (xAI Imagine).

- **Skill:** `image-gen` — picks a CLI, generates or edits an image headlessly,
  verifies the output file actually landed on disk (recovering it if needed),
  and checks size and transparency.

## Which CLI it uses

1. The one you name ("generate with grok", "use codex").
2. Otherwise: **codex** if both are installed, or whichever one is installed.

On first use the skill checks which CLIs are installed and remembers the result
in `installed-clis.json` next to `SKILL.md` (git-ignored, per machine). It
re-checks when a CLI goes missing or you say you installed or removed one.

## Prerequisites

At least one of:

- **Codex CLI** — [github.com/openai/codex](https://github.com/openai/codex),
  ≥ v0.115 for built-in image generation (`npm install -g @openai/codex`). Needs an
  OpenAI account, activated with `codex login` or an `OPENAI_API_KEY` environment
  variable (API key routes generation through the Images API at API pricing
  instead of Codex usage limits — useful for larger batches).
- **Grok CLI** (Grok Build) — signed in with `grok login`, with image generation
  enabled (`features.image_gen`, on by default).

## Installation

Claude Code loads skills from `~/.claude/skills/` (global, all projects) or
`.claude/skills/` (project-local). To install:

```bash
cp -r skills/image-gen ~/.claude/skills/
```

Restart Claude Code (or start a new session) so it picks up the new skill.

## Usage

Ask Claude Code to generate or edit an image, e.g. "generate an icon of a
lighthouse", "edit this banner to use a dark background", or "make this with
grok". See `skills/image-gen/SKILL.md` for the full workflow.

> Previously published as `codex-image-gen` (Codex only). GitHub redirects the
> old repo URL here.
