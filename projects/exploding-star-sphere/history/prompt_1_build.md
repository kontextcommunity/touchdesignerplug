# Build prompt: "Exploding Star" — a particle system constrained to a sphere (TouchDesigner 2023)

You are the kontext.stage build agent. Build this network in the live project and report exactly what you built. Everything you need is in this message: you cannot open files from disk. Follow the order. Verify after every block.

---

## 0. How to read this prompt

Every statement carries a source tag:
- **[SRT]** — said in the video narration (auto-generated subtitles, so wording can be slightly off).
- **[FRAME]** — seen on screen in the video (640x360 recording, so numbers in parameter fields are NOT readable).
- **[INFERRED]** — derived from [SRT]/[FRAME] and TouchDesigner knowledge. Treat as the best plan, not a fact.
- **[UNKNOWN]** — the video does not give the value. Pick the starting value written here, label it **APPROXIMATE** in your report, and note the range to try.

The video was recorded in **TouchDesigner 2023.11340** [FRAME]. Build in this project's version, and tell me the build number.

## 1. Result to reproduce

A custom GPU particle system, 1,000,000 particles [SRT]. Positions live in a 1000x1000 32-bit float texture that feeds itself through a Feedback TOP [SRT]. Noise moves the positions a tiny step every frame [SRT]. A final step normalizes every position to length 1, so all particles lie on the surface of a sphere [SRT].

What the picture looks like [FRAME]:
- Before the loop: a white vertical column (the thin tube), instanced as points.
- With noise: the column becomes swirling white strands that look like smoke or gas.
- After colouring with a Level TOP: pale mint-green strands on pure black.
- After the sphere step: a thin glowing shell of fine strands shaped like a sphere. Playing with a Math TOP "Post Add" turns it into shapes that look like an exploding star [SRT].
- Bloom on the final image [SRT].

## 2. Rules for you

1. Build inside a new container `/project1/exploding_star`. Do not touch `kontext_stage` or `stage_work`.
2. **Before setting parameters of any operator type you have not used, call `operator_info` for it.** Parameter names below are descriptive (for example "radius", "number of points"). Use the real internal names from `operator_info`. Never guess names.
3. Use full type names with the family: `tubeSOP`, `noiseTOP`, `mathTOP`. Never write `noise` or `null` alone.
4. After each block call `verify`. Fix errors before you continue. Explain any warning.
5. Do not invent the author's "Life" component. See block 6 for the replacement.
6. Label every value you chose yourself **APPROXIMATE**.
7. Do not publish anything and do not write outside the project.
8. At the end, use `annotate` (see section 12) and then give the final report (section 13).

## 3. Block 1 — Point cloud (video 0:23–2:50)

| # | Type | Name | Settings | Source |
|---|---|---|---|---|
| 1 | tubeSOP | `tube1` | radius = 0.1 | [SRT] |
| 2 | sprinkleSOP | `sprinkle1` | input `tube1`; number of points = `op('constant1')['count']**2` | [SRT] |
| 3 | constantCHOP | `constant1` | one channel named `count`, value 1000 | [SRT] |
| 4 | nullSOP | `null_sop1` | input `sprinkle1` | [SRT] |
| 5 | soptoCHOP | `sopto1` | input `null_sop1`; output channels tx, ty, tz, one sample per point | [SRT], [FRAME] |
| 6 | nullCHOP | `null_chop1` | input `sopto1` | [SRT] |
| 7 | choptoTOP | `chopto1` | CHOP = `null_chop1`; channels tx, ty, tz go to red, green, blue; data layout = fit to square; pixel format = 32-bit float RGBA | [SRT] |

Check: `chopto1` is 1000x1000 and 32-bit float. 1000 x 1000 = 1,000,000 points. Sprinkle must produce exactly 1,000,000 points.

## 4. Block 2 — Position loop (video 2:50–3:30)

| # | Type | Name | Settings | Source |
|---|---|---|---|---|
| 8 | nullTOP | `start` | input `chopto1` | [SRT] |
| 9 | feedbackTOP | `feedback1` | input = `start` (the reset image); target TOP = `end` | [SRT] |
| 10 | nullTOP | `pos` | input `feedback1` | [SRT] |
| 11 | nullTOP | `end` | input = `math_shape` (the end of the update chain: `add_noise` → `point_xform` → `life_pos` → `math_len`/`comp_div` → `math_shape`) | [SRT] |

`pos` is the current position texture. `end` is the next frame. The loop closes through the feedback target.

## 5. Block 3 — Rendering (video 3:30–5:00)

| # | Type | Name | Settings | Source |
|---|---|---|---|---|
| 12 | addSOP | `add1` | one point at the origin | [SRT] |
| 13 | convertSOP | `convert1` | input `add1`; convert to particles, one per point | [SRT] (details [INFERRED]) |
| 14 | geoCOMP | `geo1` | holds the SOP chain. Instancing **On**. Instance OP = `pos`. Instance Translate X/Y/Z = r / g / b. Material = `pointsprite1` | [FRAME] at 4:30 |
| 15 | cameraCOMP | `cam1` | looks at the origin; distance about 4 | [UNKNOWN] — APPROXIMATE |
| 16 | pointspriteMAT | `pointsprite1` | alpha lowered a little (start 0.3, try 0.1–0.6); point size small (start 0.01) | [SRT] alpha; size [UNKNOWN] |
| 17 | renderTOP | `render1` | camera `cam1`, geometry `geo1`; resolution 1920x1080 (1280x720 if the licence limits it); pixel format 16-bit float RGBA; black background | [SRT] pixel format and resolution |
| 18 | constantTOP | `bg1` | black, same resolution | [SRT] "transform to create a background" |

Wire `render1` into the colour and post chain (block 8).

Check: with the loop idle, the render shows a thin white vertical column [FRAME].

## 6. Block 4 — Noise motion (video 4:00–6:30)

| # | Type | Name | Settings | Source |
|---|---|---|---|---|
| 19 | noiseTOP | `noise1` | input `pos` (connect `pos` to every input the noise has, as in the video); offset 0; amplitude 1; monochrome **Off**; Output page: **Derivative** (needs the 2023 build) | [SRT] |
| 20 | mathTOP | `math_step` | input `noise1`; multiply = 0.001 | [SRT] |
| 21 | compositeTOP | `add_noise` | operation Add; input 0 = `pos`, input 1 = `math_step`; 32-bit float | [INFERRED] from "add this to the loop" |

The noise period and harmonics are [UNKNOWN]. Start with the defaults (period 1). Try 0.3–3 and report your choice.

Check: with the loop running, the white column turns into swirling strands [FRAME].

## 7. Block 5 — Rotation with a mask (video 7:10–9:30)

Goal: a simple rotating motion of the particles, only for some pixels, in both directions.

| # | Type | Name | Settings | Source |
|---|---|---|---|---|
| 22 | noiseTOP | `noise_mask` | monochrome On; animate slowly with `absTime.seconds * 0.1` | [INFERRED] |
| 23 | thresholdTOP | `threshold1` | input `noise_mask`; threshold 0.5 | [SRT] threshold; [FRAME] 8:30; value [UNKNOWN] |
| 24 | mathTOP | `math_range` | input `threshold1`; **To Range = -1 to 1** | [SRT] "-1:1 for contrary motion" |
| 25 | Point Transform | `point_xform` | input 0 = `add_noise`; input 1 (second input) = `math_range` as the mask; rotation = a small angle around Y (start 0.5 degrees per frame) | [SRT], [FRAME] |

Use `operator_types` (family TOP, query "point transform") to find the real operator. **If no Point Transform TOP exists in this build**, replace `point_xform` with a GLSL TOP: rotate the RGB position vector around the Y axis by `uAngle * mask`, keep alpha. Label it **REPLACEMENT**.

The motion "can be a little intense" [SRT]. Lower the noise amplitude (start `noise1` amplitude at 0.5).

## 8. Block 6 — Life (the replacement)

The author uses his own "Life" component (Patreon). It is external and not available. Build this stateless replacement and label it **REPLACEMENT** in your report.

- `life_pos` (glslTOP): input 0 = the moved positions, input 1 = `start`. Resolution 1000x1000, 32-bit float RGBA.
- Uniforms (Vectors page; read `operator_info` for the glslTOP names):
  - `uTime` = `absTime.seconds`
  - `uDT` = `1/project.cookRate`
  - `uLife` = 8 (seconds a particle lives; try 3–20)
- GLSL:

```glsl
uniform float uTime;
uniform float uDT;
uniform float uLife;
out vec4 fragColor;
float hash(vec2 p){ return fract(sin(dot(p, vec2(127.1, 311.7))) * 43758.5453); }
void main(){
    vec2 uv = vUV.st;
    vec4 cur = texture(sTD2DInputs[0], uv);
    vec4 st  = texture(sTD2DInputs[1], uv);
    float h  = hash(uv);
    float g1 = floor(uTime / uLife + h);
    float g0 = floor((uTime - uDT) / uLife + h);
    bool respawn = (g1 != g0);
    fragColor = vec4(respawn ? st.rgb : cur.rgb, 1.0);
}
```

If you cannot bind the uniforms, skip the respawn, build without Life, and write that in the deviations.

Reset key [SRT]: a Keyboard In DAT, key `1`, with a callback that pulses the Feedback TOP reset parameter (find its name with `operator_info`). Reset re-seeds from `start`.

## 9. Block 7 — Sphere constraint (video 10:00–12:00) — the key step

All steps apply to the positions **after** the noise, the rotation and the life step, and **before** `end`.

| # | Type | Name | Settings | Source |
|---|---|---|---|---|
| 26 | mathTOP | `math_len` | input `life_pos`; ignore alpha; combine channels = **Length** (distance from zero of r, g, b); output monochrome; 32-bit float | [SRT] |
| 27 | compositeTOP | `comp_div` | input 0 = `life_pos`, input 1 = `math_len`; operation = **Divide**; on the transform page set the fixed layer to **input 1** so the result does not turn monochrome; 32-bit float RGBA | [SRT] |
| 28 | mathTOP | `math_shape` | input `comp_div`; multiply = 1.0 (radius of the sphere); Post Add = 0, 0, 0 | [SRT] |

Wire `math_shape` into `end`.

Why it works [SRT]: a number divided by itself is 1, so every vector has length 1, so every particle sits one unit from zero, which is a sphere. If `math_len` has no "length" combine option, compute it as the square root of (r² + g² + b²) with Math TOPs and tell me.

Check: the render shows a thin glowing shell shaped like a sphere [FRAME 11:00].

## 10. Block 8 — Colour and post (video 5:30–6:00, 7:50–8:10, 9:50–10:10)

| # | Type | Name | Settings | Source |
|---|---|---|---|---|
| 29 | levelTOP | `level_color` | input `render1`; RGBA tab: low/high per channel to give pale mint green (start high R 0.55, G 1.0, B 0.75) | [SRT] "Level TOP to add colour", [FRAME] 10:00; values [UNKNOWN] |
| 30 | bloomTOP | `bloom1` | input `level_color`; default settings first | [SRT] (needs the 2023 build) |
| 31 | compositeTOP | `comp_out` | `bloom1` over `bg1` | [INFERRED] |
| 32 | nullTOP | `out1` | input `comp_out` | rule: the output is always `out1` |

## 11. Tweak knobs — "what to change" (write these in your annotations)

| Operator.parameter | What it does | Try range | Start |
|---|---|---|---|
| `constant1` channel `count` | particle count = count² | 300–1000 | 1000 |
| `math_step` multiply | how far noise moves a particle per frame | 0.0002–0.003 | 0.001 |
| `noise1` period / amplitude | scale and strength of the flow | 0.3–3 / 0.2–1 | 1 / 0.5 |
| `threshold1` threshold | which pixels rotate | 0.3–0.7 | 0.5 |
| `point_xform` rotation | speed of rotation | 0.1–2 deg/frame | 0.5 |
| `math_shape` multiply | sphere radius | 0.5–2 | 1 |
| `math_shape` Post Add | shifts the shape; makes the "exploding star" shapes | -0.5–0.5 | 0 |
| `math_shape` Post Add, blue off | removes the blue channel offset | 0 or 1 | on |
| `pointsprite1` alpha / size | particle visibility | 0.1–0.6 / 0.003–0.03 | 0.3 / 0.01 |
| `life_pos` uLife | seconds before a particle respawns | 3–20 | 8 |
| `level_color` RGBA high | colour of the particles | any | mint green |
| `bloom1` | glow strength | defaults first | default |

Do not touch: `feedback1`, `start`, `pos`, `end` (they form the position loop).

## 12. Annotations

Call `annotate` for these boxes (labels in the language the user writes in; operator names stay as is):
1. Point cloud (block 1) 2. Position loop (2) 3. Rendering (3) 4. Noise motion (4) 5. Rotation (5) 6. Life replacement (6) 7. Sphere constraint (7) 8. Colour and post (8). Under each box add a note: what to change, the range, the format, and what not to touch (section 11).

## 13. Acceptance checks (run them and report the results)

1. `verify` returns no errors for the whole container.
2. `chopto1` and `pos` are 1000x1000, 32-bit float.
3. The point count is 1,000,000 (state how you checked it).
4. Frame time: report cook ms and fps. If the GPU is too slow, reduce `count` to 600 and say so.
5. Render at t = 3 s and t = 10 s. Describe each frame in two sentences: is it a sphere shell? what colour? is there glow?
6. Press key `1`: the picture resets and rebuilds.
7. Change `math_shape` Post Add and describe the shape change.

## 14. Final report format (paste it in the chat)

1. Build number of TouchDesigner.
2. Table: operator name | type with family | key parameters (real names) | status: **FOLLOWED**, **APPROXIMATE**, **REPLACEMENT**.
3. The list of parameter names that differed from this prompt (for example the real name of "radius").
4. Deviations from the video, one per line, with the reason.
5. The acceptance results from section 13.
6. The output of `export_network` for `/project1/exploding_star` (depth 3, include DAT text).

Be literal. If something does not match this prompt, say so instead of making it fit.
