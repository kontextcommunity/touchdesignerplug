# ACID нойз (многослойный, аудио)

**Теги:** 2d, noise, layers, edge, feedback
**Когда:** «кислотный», «многослойный», «хаотичный».

## Граф
noise1 (period ← high, t4d ← time), noise2 (seed 2), noise3 (period 0.07, ty ← time*2)
→ comp(add) → noise4 (amp ← low)
→ опционально text (логотип) с comp(multiply)
→ edge1 (blacklevel ← kick) → blur1 → comp(color, noise4)
→ feedbackTOP (top = over) + level(decay) → over
→ noise5 → displace → level → null1.

## Вариации
- Без текста — убрать comp(multiply) с text.
- Без feedback — чётче, меньше размытия.
- В comp пробовать: multiply, color, hardlight.

Источник: Obsidian Проекты/ACID Нойз.md (порт 9984, 4 аудио-канала).
