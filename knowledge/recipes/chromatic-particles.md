# Хроматические частицы (3D, particle SOP, feedback)

**Теги:** 3d, particles, feedback, chromatic aberration, audio
**Когда:** «частицы», «облако», «поток», «хаос».

## Граф
audio → select1 → math1 (torange 0.1..0.2) → particle1.turbx
lfo1 (sin, 0.3 Hz) → particle1.turby
line1 → transform1 (параметры от audio) → particle1 (birth 10000, life 3.0)
→ null1 → geo1 → render1 (1920×1080, aa4)
→ null2 → comp1 ← transform2 (rotate, tile) → level1 (opacity ← high)
→ feedback1 → mirror1 (rotate 90) → rgbaDelay (R −6, G −12, B −2)
→ bloom → rgbkey → rgbaDelay2 → output.

## Ключевое
- rgbaDelay: RGB сдвинуты на разное число кадров → хроматическая аберрация. Применять после bloom.
- Частицы: select1_export даёт каналы напрямую в таблицу для instancing.

Источник: Obsidian Проекты/Хроматические Частицы.md (порт 9990, 5 аудио-каналов).
