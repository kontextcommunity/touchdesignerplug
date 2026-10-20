# Паттерны и техники

- P1 Audio → CHOP chain: audioanalysis → select(channel) → filter → math → destination.par.expr.
  filter.width: 0.25 быстро (high/kick), 0.73–1.0 средне (mid), 1.61–2.0 медленно (low, bass).
  Каналы audioAnalysis: low, mid, high, kick, rythm, snare, centroid.
- P2 Noise → edge → feedback: noise → edge → feedback(top) → level(brightness < 1) → composite → output.
  edge.blacklevel 0.5–0.9; edge.strength 1000–15000; offset 1–31 px. kick → blacklevel или feedback.resetpulse.
- P3 Feedback loop: feedbackTOP (result, level внутри петли), либо bloom.cross1 = 0.965 − kick*0.45, либо transform scale 0.99 в петле.
- P4 Instancing (TOP → геометрия): geo.instancetop = noise/displace; instancetx = r, instancety = g. noise в rgba32float.
- P5 Chromakey + composite: rgbkey (redmin 0.43, rsoftlow 0.62, rgbout multalpha). composite operand: over, add, difference, heat, hardlight, multiply.
- P6 3D render pipeline: cameraCOMP + lightCOMP + geometryCOMP (SOP внутри) → renderTOP (camera, lights, antialias aa4). Авто-вращение: cam.ry.expr = absTime.seconds * 30.
- P7 Chromatic aberration: rgbaDelay (Red −6, Green −12, Blue −2, Drywet 0.989) после bloom.
- P8 Color via LUT: ramp (rampx, extend repeat/mirror, phase = time*0.1) + lookup (secondinput, independent).
- P9 GLSL uniform через constantTOP: 1×1 rgba32float, colorr/g/b через expr; в GLSL texelFetch(sTD2DInputs[1], ivec2(0,0), 0).
- P10 Particle SOP: prtype lines, birth 10000, life 3.0, drag 0.15, mass 0.05; turbx через lfo; transform1.tx от audio.

Источник: Obsidian TouchDesigner/Архитектура/Паттерны и Техники.md.
