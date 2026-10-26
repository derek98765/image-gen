---
name: image-gen
description: Use when the user wants to generate or edit a bitmap image (icon, banner, illustration, photo, character, edit of an existing image) through a local AI CLI — the Codex CLI (OpenAI gpt-image-2, the model behind ChatGPT images) or the Grok CLI (xAI Imagine, image_gen / image_edit). Picks the CLI automatically: the one the user names, else codex when both are installed, else whichever one is installed; remembers which CLIs are installed after the first check. Triggers include "generate an image", "make an icon/banner/illustration", "edit this image", "generate with codex/chatgpt/gpt", "generate with grok/imagine", "用 codex/chatgpt/grok 出圖". Not for SVG, HTML/CSS, canvas or other code-native assets.
---

# Image generation (Codex or Grok CLI)

Generates or edits images by driving one of two local CLIs headlessly:

| CLI | Model | How it runs |
|---|---|---|
| `codex` | OpenAI `gpt-image-2` (built-in `$imagegen`) | `codex exec … -i <ref> '<prompt>'` |
| `grok` | xAI Imagine (built-in `image_gen` / `image_edit` tools) | `grok -p '<prompt>' --always-approve` |

`SKILL_DIR` below means this skill's folder, `~/.claude/skills/image-gen`.

## Step 1 — Pick the CLI

1. **The user named one** ("with grok", "via codex", "use ChatGPT") → use it. If it
   isn't installed, say so and offer the install steps below or the other CLI —
   don't silently switch.
2. **Otherwise read the remembered state** in `SKILL_DIR/installed-clis.json`:

   ```json
   { "codex": true, "grok": true, "checked": "2026-10-26" }
   ```

   - **The file is missing (first run)** → check both and write the file:

     ```bash
     codex_ok=$(command -v codex >/dev/null && echo true || echo false)
     grok_ok=$(command -v grok >/dev/null && echo true || echo false)
     printf '{ "codex": %s, "grok": %s, "checked": "%s" }\n' \
       "$codex_ok" "$grok_ok" "$(date +%F)" > ~/.claude/skills/image-gen/installed-clis.json
     ```

     Tell the user in one line what was found and which CLI will be the default.
3. **Default rule:**
   - both installed → **codex**
   - only one installed → that one
   - neither → stop and offer the install steps below (ask before any global install)
4. **Re-check when the state is stale:** if the chosen CLI fails with
   `command not found`, or the user says they installed or removed one, re-run the
   check in step 2 and rewrite the file before continuing.

## Step 2 — Write the prompt

Same for both CLIs. Describe, in order: **subject → action/pose → setting → style →
composition → lighting/mood → key details**, plus background (transparent or opaque),
aspect ratio or exact size, and what must stay unchanged for an edit. Front-load the
subject; 2–5 sentences of natural prose. When fidelity to an existing asset matters,
always pass it as a reference image — words alone drift.

Image models garble exact text, numbers, charts and diagrams. If the result needs
correct copy or data, build it in code (HTML/CSS) instead.

## Step 3a — Generate with Codex

Prerequisites: `codex` ≥ v0.115 (`codex --version`), signed in once with
`codex login` (or `OPENAI_API_KEY` set for API-priced batches).

```bash
codex exec --skip-git-repo-check -s workspace-write \
  'Generate an image: <DESCRIBE>. Save it as <OUTPUT.png> in the current directory.'
```

Edit, or generate from references (`-i` is repeatable):

```bash
codex exec --skip-git-repo-check -s workspace-write -i input.png -i style-ref.png \
  'Modify the attached image to <CHANGE>. Keep <WHAT STAYS>. Save as edited.png in the current directory.'
```

- `-s workspace-write` — **required**; the default read-only sandbox makes the save
  step fail silently.
- `--skip-git-repo-check` — needed outside a git repo; harmless inside one.
- Codex always keeps the raw output under `~/.codex/generated_images/<session-id>/ig_*.png`
  — recover from there with `find ~/.codex/generated_images -type f -mmin -5`.
- For many images, `export OPENAI_API_KEY=…` first so generation is billed at API
  pricing instead of consuming Codex usage limits.

## Step 3b — Generate with Grok

Prerequisites: `grok` installed (`grok --version`), signed in with `grok login`
(`grok models` prints "You are not authenticated." when the login has expired), and
image generation not disabled (`features.image_gen` in `~/.grok/config.toml`).

```bash
grok -p 'Use the image_gen tool to generate: <DESCRIBE>. Aspect ratio <1:1|16:9|9:16|3:2|2:3>.
  Save the final image as <ABSOLUTE/PATH/OUTPUT.png> and print the saved path.' \
  --always-approve
```

Edit, or generate from references — pass **absolute paths** in the prompt; grok's
`image_edit` tool takes file paths, not attachments:

```bash
grok -p 'Use the image_edit tool with the reference image /abs/path/input.png to
  <CHANGE>. Keep <WHAT STAYS>. Save the result as /abs/path/edited.png and print the
  saved path.' --always-approve
```

- `--always-approve` lets the headless run call the image tool and write the file
  without a permission prompt. Run it from the folder the output should land in.
- `image_gen` only takes `aspect_ratio` (`1:1`, `16:9`, `9:16`, `3:2`, `2:3`, `auto`),
  not pixel sizes — resize afterwards for an exact size.
- References are downscaled to roughly 768px on the long side before they reach the
  model, so fine detail and small text in a reference won't survive.
- No `n`/`count` option — ask for variations as separate runs.
- A moderation block is final: tell the user, don't paraphrase the prompt to get
  around it.

## Step 4 — Verify (both CLIs)

1. Confirm the file exists: `ls -la <OUTPUT.png>`. If it's missing, recover it
   (codex: `~/.codex/generated_images/`; grok: re-run asking it to print the path)
   before reporting anything.
2. Check size and transparency when they matter:

   ```bash
   python3 -c "from PIL import Image; im=Image.open('<file>'); print(im.size, im.mode, im.split()[-1].getextrema() if im.mode=='RGBA' else 'no alpha')"
   ```

   Neither model reliably hits a custom pixel size — resize with
   `sips -z <H> <W> <file>` when the aspect ratio already matches, otherwise regenerate.
   If a transparent background was required and the file has no alpha (or alpha is
   `(255, 255)`), regenerate with the transparency requirement emphasized.
3. Look at the image: subject, anatomy, edges, cropping, unwanted text, and that
   anything meant to stay unchanged did.
4. Report the path, which CLI made it, and anything that's off.

## Common mistakes

- Picking grok when both CLIs are installed and the user didn't ask for it — the
  default is codex.
- Skipping the first-run check and assuming a CLI is installed, or never refreshing
  `installed-clis.json` after a `command not found`.
- Codex without `-s workspace-write` → the image is generated but never saved to the
  cwd.
- Grok with a relative reference path, or without `--always-approve` → the edit or
  save step stalls or fails in headless mode.
- Asking either model for an exact pixel size and not verifying it.
- Reporting success without checking the file exists.

## Installing a missing CLI

Ask before any global install; never fall back to a placeholder image.

- **Codex:** needs npm (`npm --version`; else `brew install node`), then
  `npm install -g @openai/codex`, then `codex login`.
- **Grok:** install the Grok Build CLI per xAI's instructions, then `grok login`.

After installing, re-run the Step 1 check so `installed-clis.json` is current.
