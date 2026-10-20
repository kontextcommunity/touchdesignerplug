# Каскад краёв (2D, edge chain, аудио)

**Теги:** 2d, edge, feedback, kick-reset, 4k
**Когда:** «линии», «контуры», «нервный», «дёрганый».

## Граф
noise1 (simplex3d, period 10, harmon 2, spread 1.6, gain ← high, 3840×2160)
→ edge1 → edge2 → edge3 → edge4 → edge5
→ feedback1 (resetpulse = kick) → level1 (brightness 0.83, gamma ← low)
→ comp1 (overlay) с edge5 → limit1 (posstep ← mid) → displace1 (uvweight ← mid)
Параллельно: edge6 → feedback2 → transform1 (sx=sy=0.99, px=py=0.6) → level2 → comp2 (add)
→ rgbkey1 → level3 → null1 → moviefileout1.

## Ключевое
- Цепочка edge: каждый следующий edge обрабатывает предыдущий, рост абстракции.
- Два независимых feedback-потока дают эхо.
- Kick-reset: feedback1.resetpulse = op('select1')['kick'].
- Масштаб 0.99 в feedback-петле: медленный зум внутрь (тоннель).

Источник: Obsidian Проекты/Каскад Краёв.md (порт 9983, аудио aleo).
