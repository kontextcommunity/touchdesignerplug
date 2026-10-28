# Rebuild prompt: turn the current `stage_work` visual into the "Exploding Star" tutorial build

**How to use:** paste the `export_network` JSON of `/project1/stage_work` first, then this whole message. The JSON above is the CURRENT state. You cannot open files; everything you need is in the JSON and in this message.

You are the kontext.stage build agent. The current network is an audio-reactive GLSL particle sphere. It is NOT the tutorial design. Change it in place, in `/project1/stage_work`, into the "Exploding Star — constraining a particle system to a sphere" build (TouchDesigner 2023 tutorial). Do not start a new container. Do not delete the annotation boxes: you will rewrite them at the end.

Source tags: **[SRT]** said in the video, **[FRAME]** seen on screen (640x360, so numbers in fields are unreadable), **[INFERRED]** derived, **[UNKNOWN]** the video does not say. Every value you choose yourself is **APPROXIMATE** in your report. A different operator is **REPLACEMENT**.

---

## 0. Target look [FRAME]
White vertical column of points → swirling smoke-like strands (noise) → pale mint-green strands on pure black → a thin glowing shell shaped like a sphere, with bloom. Raising a Math TOP "Post Add" turns the shell into "exploding star" shapes [SRT].

## 1. Rules
1. First call `td_info` and report the real TouchDesigner build (the JSON says `099`, which looks wrong). Then call `checkpoint` so the change can be undone.
2. Before touching an operator type you have not used, call `operator_info`. Use the real parameter names. Never guess names. Full type names with the family: `tubeSOP`, `noiseTOP`.
3. Work block by block (section 3). Call `verify` after every block and fix errors before going on.
4. Do not delete any operator the section 3 table does not mark REMOVE. "Disconnect" means unwire, not delete.
5. Do not push, publish or write outside the project.

## 2. Audit first (find these in the JSON, report in 6 lines max, then continue)
1. The real names of the Feedback TOP parameters for **target TOP** and **reset** (current: `top`, `reset`) — confirm with `operator_info`.
2. Does a **Point Transform TOP** exist in this build? (`operator_types`, family TOP, query "point transform"). If not, use a GLSL TOP rotation = REPLACEMENT.
3. Does the **Noise TOP** have an Output = **Derivative** option and how many inputs does it have? If there is no Derivative, say so and use `noise` output with a Math TOP (subtract 0.5, ×2) = REPLACEMENT.
4. Does the **Bloom TOP** exist (it does in the JSON). Name its intensity parameter.
5. Does the **Point Sprite MAT** exist (`pointspriteMAT`)? Name its alpha/size parameters.
6. Current GPU cost: `pos_sim` cook ms with 40,000 particles. This decides if 1,000,000 particles is allowed (section 3, block A). If the machine drops under 30 fps at 1000×1000, fall back to `count` = 600 and say so.

## 3. What to change

### Block A — Point cloud and count  (REPLACES `seed_pos` and `seed_pos_pixel`)
| Action | Operator | Settings | Source |
|---|---|---|---|
| ADD | `tube1` (tubeSOP) | radius 0.1 | [SRT] |
| ADD | `constant1` (constantCHOP) | one channel `count` = 1000 | [SRT] |
| ADD | `sprinkle1` (sprinkleSOP) | input `tube1`; number of points `op('constant1')['count']**2` | [SRT] |
| ADD | `null_sop1` (nullSOP) | input `sprinkle1` | [SRT] |
| ADD | `sopto1` (soptoCHOP) → `null_chop1` (nullCHOP) | one sample per point, channels tx ty tz | [SRT] |
| ADD | `chopto1` (choptoTOP) | CHOP `null_chop1`; tx→r, ty→g, tz→b; layout "fit to square"; 32-bit float RGBA | [SRT] |
| ADD | `start` (nullTOP) | input `chopto1` | [SRT] |
| DISCONNECT | `seed_pos`, `seed_pos_pixel` | leave in place, unused (do not delete) | — |

Check: `chopto1` is 1000×1000, 32-bit float, 1,000,000 points.

### Block B — Position loop  (EDIT `fb_pos`, `fb_src`, `pos`)
| Operator | Change |
|---|---|
| `fb_pos` | input 0 = `start` (was `seed_pos`). Output resolution 1000×1000 (was 200×200). Keep format `rgba32float`. Keep `reset` = `op('key_reset')['k1']` |
| `fb_src` | input = `end` (new, see Block F). Rename nothing |
| `end` | NEW nullTOP, input = `math_shape` (Block F) |
| `pos` | input = `fb_pos` (was `pos_sim`). Stays the name instancing reads |
| `key_reset` | keep (Keyboard In CHOP, key 1) |

Feedback target stays `fb_src`, so the loop is `pos → … → end → fb_src → fb_pos → pos`.

### Block C — Noise motion  (REPLACES the flow code inside `pos_sim_pixel`)
| Action | Operator | Settings | Source |
|---|---|---|---|
| ADD | `noise1` (noiseTOP) | inputs: `pos` on every input it has; offset 0; amplitude 0.5 (APPROXIMATE, range 0.2–1); monochrome Off; Output = Derivative; 32-bit float, 1000×1000 | [SRT] |
| ADD | `math_step` (mathTOP) | input `noise1`; multiply 0.001 | [SRT] |
| ADD | `add_noise` (compositeTOP) | operation Add; input 0 = `pos`, input 1 = `math_step`; 32-bit float RGBA | [INFERRED] |

Noise period and harmonics are [UNKNOWN]: start with defaults (period 1), try 0.3–3, report the choice.

### Block D — Rotation with mask
| Action | Operator | Settings | Source |
|---|---|---|---|
| ADD | `noise_mask` (noiseTOP) | monochrome On; slow motion `absTime.seconds * 0.1` | [INFERRED] |
| ADD | `threshold1` (thresholdTOP) | input `noise_mask`; threshold 0.5 (APPROXIMATE) | [SRT]/[FRAME] |
| ADD | `math_range` (mathTOP) | input `threshold1`; To Range −1 … 1 | [SRT] |
| ADD | `point_xform` (Point Transform TOP, per audit 2) | input 0 = `add_noise`, input 1 = `math_range` (mask); small rotation around Y, start 0.5°/frame (APPROXIMATE) | [SRT]/[FRAME] |

### Block E — Life replacement  (EDIT `pos_sim`)
The tutorial uses the author's external Life component. It is not available. Reuse `pos_sim` as the respawn shader, label it **REPLACEMENT**:
- `pos_sim` inputs: 0 = `point_xform`, 1 = `start`. Resolution 1000×1000, `rgba32float`.
- Rename it `life_pos`. Delete its old uniforms usage; keep the Vectors page and use `uTime` = `absTime.seconds`, `uDT` = `1/project.cookRate`, `uLife` = 8 (range 3–20).
- Replace the text of `pos_sim_pixel` with:

```glsl
uniform vec4 uP;   // x = time, y = dt, z = life seconds
out vec4 fragColor;
float hash(vec2 p){ return fract(sin(dot(p, vec2(127.1, 311.7))) * 43758.5453); }
void main(){
    vec2 uv = vUV.st;
    vec4 cur = texture(sTD2DInputs[0], uv);
    vec4 st  = texture(sTD2DInputs[1], uv);
    float h  = hash(uv);
    float g1 = floor(uP.x / uP.z + h);
    float g0 = floor((uP.x - uP.y) / uP.z + h);
    fragColor = vec4(g1 != g0 ? st.rgb : cur.rgb, 1.0);
}
```
Bind `uP` as `vec0` (name `uP`; x = `absTime.seconds`, y = `1/project.cookRate`, z = 8). If binding fails, bypass the respawn (pass input 0 through) and write it in the deviations.

### Block F — Sphere constraint (the key step)
| Action | Operator | Settings | Source |
|---|---|---|---|
| ADD | `math_len` (mathTOP) | input `life_pos`; ignore alpha; combine channels = **Length**; monochrome; 32-bit float | [SRT] |
| ADD | `comp_div` (compositeTOP) | input 0 = `life_pos`, input 1 = `math_len`; operation **Divide**; fixed layer = input 1 (so the result stays RGB); 32-bit float RGBA | [SRT] |
| ADD | `math_shape` (mathTOP) | input `comp_div`; multiply 1.0; Post Add 0,0,0 | [SRT] |
| ADD | `end` | see Block B | — |

If `math_len` has no Length option, build it as sqrt(r²+g²+b²) with Math TOPs and say so. The soft pull `mix(p, n*target, uSphere.y)` is gone: every point must sit at exactly radius = `math_shape` multiply.

### Block G — Render  (EDIT `geo_ps`, `mat_dot`, `camera1`, `render1`)
| Operator | Change |
|---|---|
| `geo_ps` | instancing On; instance OP `pos`; translate r/g/b; `numinstances` = 1,000,000 or the TOP size (not 40,000). Material = `pointsprite1`. Inside: REMOVE `disc1`; ADD `add1` (addSOP, one point) → `convert1` (convertSOP, to particles) → `out` null |
| `pointsprite1` | NEW pointspriteMAT replacing `mat_dot`. Alpha 0.3 (range 0.1–0.6), point size 0.01 (range 0.003–0.03) — APPROXIMATE. `mat_dot` stays unused (do not delete) |
| `camera1` | keep `tz` 3.4. Replace the sway expressions in `rx`/`ry` with 0 (the tutorial camera is static) |
| `render1` | keep. Pixel format 16-bit float RGBA, black background |

### Block H — Colour, post, audio
| Operator | Change |
|---|---|
| `level_color` | RGBA tab per channel: pale mint green (start high R 0.55, G 1.0, B 0.75; `brightness1` back to 1.0) — APPROXIMATE |
| `bloom1` | `bloomintensity`: replace the expression with 1.0 (APPROXIMATE, range 0.5–2). Keep threshold 0.5 |
| `bg1`, `over1`, `out1` | keep as is |
| audio chain (`audiofilein1`, `audiodevout1`, `audio_level`, `audio_smooth`, `audio_norm`, `audio_limit`, `audio_out`) | DISCONNECT only: replace every expression that reads `audio_out` with a constant (`pos_sim` uniforms are rebuilt in Block E; `mat_dot` is unused; `bloom1` is set above). Pause `audiodevout1` (volume 0). Do not delete |

## 4. Annotations (rewrite all 9 existing boxes)
Use `annotate` to update the existing `annotate1..annotate9` (same kinds). New titles and contents:
1. **1 · Point cloud**: `tube1`, `constant1`, `sprinkle1`, `null_sop1`, `sopto1`, `null_chop1`, `chopto1`, `start`.
2. **2 · Position loop**: `fb_pos`, `fb_src`, `pos`, `end`, `key_reset`.
3. **3 · Motion**: `noise1`, `math_step`, `add_noise`, `noise_mask`, `threshold1`, `math_range`, `point_xform`, `life_pos`, `pos_sim_pixel`.
4. **4 · Sphere**: `math_len`, `comp_div`, `math_shape`.
5. **5 · Render**: `geo_ps`, `camera1`, `pointsprite1`, `render1`.
6. **6 · Colour and output**: `level_color`, `bloom1`, `bg1`, `over1`, `out1`.
7. Note boxes 7, 8, 9 become **"Что менять"** notes for blocks 1–2 + 3, 4 + 5, 6, each as a list "operator.parameter — what it does — range — start" and a last line "Не трогать: …". Use these rows:

| Operator.parameter | Does | Range | Start |
|---|---|---|---|
| `constant1` `count` | particles = count² | 300–1000 | 1000 |
| `math_step` multiply | step of the noise per frame | 0.0002–0.003 | 0.001 |
| `noise1` period / amplitude | scale / strength | 0.3–3 / 0.2–1 | 1 / 0.5 |
| `threshold1` threshold | which pixels rotate | 0.3–0.7 | 0.5 |
| `point_xform` rotation | speed | 0.1–2°/frame | 0.5 |
| `life_pos` `uP.z` | seconds before respawn | 3–20 | 8 |
| `math_shape` multiply | sphere radius | 0.5–2 | 1 |
| `math_shape` Post Add | shape; makes the "exploding star" | −0.5–0.5 | 0 |
| `pointsprite1` alpha / size | visibility | 0.1–0.6 / 0.003–0.03 | 0.3 / 0.01 |
| `level_color` RGBA high | colour | any | mint green |
| `bloom1` intensity | glow | 0.5–2 | 1 |

Do-not-touch line: `fb_pos`, `fb_src`, `pos`, `end`, `start`, `key_reset`, names `out1` and `pos`.

Write only what is true in the network after your changes. If an old annotation claim no longer fits, remove it.

## 5. Acceptance checks (run them, report results)
1. `verify` clean on the whole container.
2. `chopto1`, `fb_pos`, `pos` are 1000×1000 float. Point count = 1,000,000 (say how you checked it).
3. cook ms and fps. If too slow, `count` = 600 and say so.
4. Render at t = 3 s and 10 s; two sentences each: sphere shell? colour? glow?
5. Press key 1: the picture resets and rebuilds.
6. Set `math_shape` Post Add = 0.2, describe the shape change, set it back to 0.
7. `math_len` sampled on a few points returns ≈ `math_shape` multiply (sphere is exact).

## 6. Final report
1. Real TD build number and the audit answers (section 2).
2. Table: operator | type with family | key parameters (real names) | FOLLOWED / APPROXIMATE / REPLACEMENT.
3. Parameter names that differed from this message.
4. Deviations from the video, one line each, with the reason.
5. Acceptance results.
6. `export_network` of `/project1/stage_work` (depth 3, with DAT text).

Be literal. If something does not match this message, say so instead of making it fit.
