# Жидкий нойз (2D, аудио)

**Теги:** 2d, noise, audio, bloom, glsl, portrait
**Формат:** 360×900, rgba32float. Выход null1 → Syphon / запись.
**Когда:** «жидкий», «органичный», «переливающийся».

## Граф
audioAnalysis (COMP) → select(low, mid, high, kick, rythm) → filter → math → параметры шума.
- noise2 (simplex4d, реактивный): exp ← low, period ← mid, offset ← high
- noise1 (base, simplex4d): amp ← mid
- edge1 ← noise2, blur1 (size = high*45+6)
- comp2 (difference) noise2 + edge/blur; comp1 (heat) noise1 + comp2
- displace1 (uvweight ← rythm) → bloom (cross1 = 0.965 − kick*0.45, glowinput)
- comp3 (difference) с comp1 → displace2 (uvweight ← low)
- GLSL HSV (hue ← low, sat ← mid, val ← kick) через constantTOP-uniform
- level1 (brightness ← kick*0.4+0.3) → null1

## Ключевое
- noise2.seed: другой seed = другой характер.
- noise2.period: 0.3–2.0 — скорость изменения.
- Feedback без feedbackTOP: bloom.cross1.

Источник: Obsidian Проекты/Жидкий Нойз.md (порт 9980/9981).
