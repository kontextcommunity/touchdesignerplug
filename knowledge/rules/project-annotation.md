# Project annotation (handover to a person)

Every finished build is handed over so that a person who opens the project sees at once where everything is and what they may change. This is the last step of every build, after verify (and look) pass, before the final summary. Use the `annotate` tool.

## Language
- All block titles and notes are written in the language the person uses in the chat: Russian chat → Russian labels, English → English, Turkish → Turkish, and so on.
- Operator names stay technical (lowercase English, as in TouchDesigner). Only the human-readable labels follow the chat language.
- The `name` argument of `annotate` is an ASCII id (block_audio, note_color); the visible title can be in any language.

## 1. Group the network into blocks
- Split the network by its logic, not by operator family. Typical blocks: input / source, audio analysis, generation, processing / effects, color, feedback, 3D / render, post, output, controls.
- One `annotate(kind=group)` per block: a network box around all its operators (`ops`). The box encloses the operators, so the person can drag the block as a whole.
- Title = number + name in data-flow order, e.g. `1 · Аудио`, `2 · Шум`, `3 · Feedback`, `4 · Цвет`, `5 · Выход`.
- `role` colours the box: input, audio, generate, process, color, feedback, render, output, controls, note.
- Layout before boxing: blocks left → right in data-flow order, one chain per row, ~200 px between operators, ≥ 120 px between blocks, ~200 px free under each block for its note. Boxes must not overlap.
- Do not box single helper nodes separately; put them into the block they serve.

## 2. Write what can be edited
Under every block — or under a single node when only that node matters — add `annotate(kind=note)` with:
- **What to change:** `op.parameter` — what it does in plain words.
- **Range:** min–max that keeps the result usable, plus the recommended value or the current one.
- **Format / units:** 0–1, px, Hz, seconds, frames, degrees, RGB 0–1, menu values (list them), file path, toggle.
- **Dependencies:** if a parameter is driven by an expression or by audio, say so and point to the place to edit instead.
- **Do not touch:** internal nodes whose change breaks the network (feedback loops, converters, nulls used by expressions).

Keep each note short: 3–8 lines, one parameter per line. One note per block, plus notes for key nodes (palette/ramp, main noise, output resolution, audio gain).

Example (Russian chat):

```
Что менять
• ramp_palette — цвета палитры: клик по ключу, цвет RGB 0–1
• noise_main.period — масштаб узора: 0.2–5 (сейчас 1.5)
• level_out.brightness1 — яркость: 0–2 (1 = без изменений)
• fb_level.opacity — длина шлейфа: 0.80–0.98, больше — длиннее
Не трогать: fb_loop, null_fb — петля обратной связи
```

Example (English chat):

```
What to edit
• audio_gain.gain — reaction strength: 0–10 (now 3)
• audio_filter.width — smoothing, seconds: 0.05–1
Driven by audio: noise_main.amp — edit audio_gain instead
```

## 3. Controls block (when the project has knobs)
If the build exposes custom parameters or a control COMP, put them into a `controls` block and describe each control there with its range and format.

## 4. Final summary in the chat
List the blocks in order and the 3–5 most useful parameters (op path + parameter + range). The summary is in the chat language too.
