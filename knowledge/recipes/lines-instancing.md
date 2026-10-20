# Линии в пространстве (3D, instancing, аудио)

**Теги:** 3d, instancing, line sop, audio, bloom
**Когда:** «линии», «поле», «wave field», аудио-реактив.

## Граф
audio → filter(×4) → math(×4): gain ← high, amp ← mid
ramp1 + ramp2 → reorder1 → noise1 (perlin2d, tx = time/4)
→ displace1 (uvweight 0.6) → geo1 (line1, instancetop = displace1, tx = R, ty = G)
→ render1 (cam1: tz 2.8, fov 45) → bloom + rgbkey → null1.

## Ключевое
- Instancing: позиции из R и G TOP, масштаб из B.
- Аудио-chain: device in → analyze(rms) → filter → math(0..0.25 → 0..1).
- Геометрия: display и render = True у финального SOP.

Источник: Obsidian Проекты/Линии в Пространстве.md (порт 9988, 2 аудио-канала).
