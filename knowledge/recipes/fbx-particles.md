# Пылающий паук (FBX + GPU-частицы)

**Теги:** 3d, fbx, particles, bloom
**Когда:** «персонаж», «живая анимация», «объём».

## Граф
FBX.fbx → importselect (geometry path) → merge → deform (skelrootpath = абсолютный путь к FBX COMP)
→ transform (scale 0.01) → sprinkle (30000 pts, surface, neardist 0.01)
→ particlesGpu (birth 30000, external force, turb = time*speed)
→ bloom (iterations 6–8, intensity 1–3) → rgbkey → comp(add с другим потоком) → output.

## Ключевое
- FBX в сантиметрах: scale 0.01 обязательно.
- Частицы в одной точке: sprinkle не подключён к deform, или нет scale.

Источник: Obsidian Проекты/Пылающий Паук.md (порт 9987).
