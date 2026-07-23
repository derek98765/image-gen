# codex-image-gen

Generate or edit images within Claude Code using OpenAI's `gpt-image-2`
model, by shelling out to the Codex CLI.

- **Skill:** `codex-image-gen` — generates or edits an image via
  `codex exec`, verifies the output file actually landed on disk (recovering
  it from Codex's internal cache if the write was sandboxed away), and
  resizes it if the user asked for specific dimensions.

## Prerequisites

- The [Codex CLI](https://github.com/openai/codex) must be installed
  (`which codex`; needs ≥ v0.115 for built-in image generation).
- You must have an OpenAI subscription/account.
- The account must be activated in the Codex CLI, either by:
  - running `codex login` (browser-based auth), or
  - setting an `OPENAI_API_KEY` environment variable (routes generation
    through the Images API at standard API pricing instead of Codex usage
    limits — useful for larger batches).

## Installation

Claude Code loads skills from `~/.claude/skills/` (global, all projects) or
`.claude/skills/` (project-local). To install:

```bash
cp -r skills/codex-image-gen ~/.claude/skills/
```

Restart Claude Code (or start a new session) so it picks up the new skill.

## Usage

Ask Claude Code to generate or edit an image via Codex/ChatGPT/GPT, e.g.
"generate an icon with codex" or "use codex to make a banner image." The
skill handles invoking the Codex CLI correctly (sandbox flags, reference
images for edits, output verification) — see
`skills/codex-image-gen/SKILL.md` for the full workflow.
