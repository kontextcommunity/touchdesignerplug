# Слежение за формами (3D, blobtrack, instancing)

**Теги:** 3d, tracking, video, instancing, ortho
**Когда:** «tracking», «интерактивно», «следить за движением».

## Граф
video.mp4 → fit1 → tex3d1 → tima1 (offset −59) → cache1 (32 кадра, index −60)
→ blobtrack1 (thresh 0.46) → datto (u, v, w, h)
→ reorder1 (R=u, G=v) → pos TOP; reorder2 (R=w, G=h) → scale TOP
→ geo1 (box1, instancetop = pos, material constant)
→ render1 (ortho cam, origin bottomleft) → comp (render + bg) → output.

## Ключевое
- Ortho: sy = resx/resy.
- Если боксы в одной точке, проверить имена колонок в blobtrack_blobs DAT.

Источник: Obsidian Проекты/Слежение за Формами.md (порт 9989, mid cue).
