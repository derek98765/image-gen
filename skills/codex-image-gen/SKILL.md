---
name: codex-image-gen
description: Use when the user wants to generate or edit an image via OpenAI / ChatGPT / Codex (gpt-image-2). Triggers include "generate an image with codex/chatgpt/gpt", "用 codex/chatgpt 出圖", "make an icon/banner/illustration via codex". Not for Gemini Gems (use gemini-gem-image) or other providers (wavespeed-ai, poe-api).
---

# Codex Image Generation

Generate or edit images with OpenAI's `gpt-image-2` by shelling out to the
Codex CLI. Only use this for OpenAI-generated images — route Gemini Gems to
`gemini-gem-image`, and other providers to `wavespeed-ai` or `poe-api`.

## Workflow

### 1. Check prerequisites

- `which codex` — if missing, stop and tell the user to install the Codex
  CLI (needs ≥ v0.115 for built-in image generation).
- Auth is either a prior `codex login` or `OPENAI_API_KEY` in the
  environment. Don't check this up front — a missing login surfaces as a
  failed run in step 3.

### 2. Build the command

```bash
codex exec --skip-git-repo-check -s workspace-write \
  'Generate an image: <description>. Save it as <output>.png in the current directory.'
```

- `-s workspace-write` is required. The default sandbox is read-only, so
  codex generates the image but the copy into cwd silently fails.
- `--skip-git-repo-check` — needed only when cwd is not a git repo; harmless
  otherwise, so keep it.
- To edit an existing image, attach it with `-i` and describe the change:

  ```bash
  codex exec --skip-git-repo-check -s workspace-write -i input.png \
    'Modify the attached image to <change>. Save as edited.png in the current directory.'
  ```

  `-i` is repeatable for multiple reference images.
- For batches (many images), have the user export `OPENAI_API_KEY` first:
  codex then routes through the Images API at standard API pricing instead
  of consuming Codex usage limits.

### 3. Run it, then verify the file exists

Never report success from codex's output alone. After the command:

```bash
ls -la <output>.png
```

If the file is missing, codex almost certainly still produced it — every
generation is also saved under `~/.codex/generated_images/<session-id>/`.
Recover it:

```bash
find ~/.codex/generated_images -type f -mmin -5
cp <found-file> <output>.png
```

### 4. Resize if the user asked for specific dimensions

`gpt-image-2` only returns its native sizes (typically 1024×1024 or
1254×1254) regardless of what the prompt asks. If the user wants exact
dimensions, resize after:

```bash
sips -z <height> <width> <output>.png
```

## Troubleshooting

- **Ran but no file anywhere, or auth errors** → run `codex login`, and
  confirm the CLI is ≥ v0.115 (`codex --version`).
- **`Operation not permitted` in codex's output** → you forgot
  `-s workspace-write`. Recover the file from `~/.codex/generated_images/`
  (step 3) rather than re-running — the image already exists.
- Don't use `--dangerously-bypass-approvals-and-sandbox` — `-s
  workspace-write` is sufficient.
