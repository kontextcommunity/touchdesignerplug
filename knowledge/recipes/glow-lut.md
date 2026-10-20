# Цветное свечение (2D, генеративный, LUT)

**Теги:** 2d, generative, lut, ramp, keyboard
**Когда:** «генеративный», «алгоритмический», «интерактивный».

## Граф
noise1 (4096×4096, seed = счётчик клавиш) + noise2 (1920×1080) → comp1 (hardlight)
→ edge1 → base1 → edge2 → base2 → feedbackEdge (TOX: scale 1.003, gain 0.985)
→ edge3 → lookup1 (ramp1 анимированный LUT) → level1 (brightness 2.0) → bloom
→ comp2 (add: bloom + level1) → output.

## Ключевое
- Ramp: phase = absTime.seconds * 0.1, extendleft = repeat или mirror.
- Lookup: method = secondinput, channel = independent.
- Клавиатура: keyboardin1 меняет seed.

Источник: Obsidian Проекты/Цветное Свечение.md (порт 9985).
