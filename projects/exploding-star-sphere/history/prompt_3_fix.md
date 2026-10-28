# Промт-правка 3 (после второй сборки)

Отправлен в плагин после сверки сборки 2 (`02_plugin_rebuild_prompt.json`). Плагин выполнил все 6 пунктов, итог — `03_plugin_fix_prompt.json`, кадр `frames/03_fix_solid_sphere.jpg`.

```
Fix the existing Exploding Star build in /project1/stage_work. Change ONLY what is listed. Do not delete anything except where it says so. Use operator_info for any parameter name you are not sure of, call verify after the changes.

1. point_xform (Point Transform TOP): rotation Y must be a small CONSTANT angle per frame, not a growing one. Remove the expression math.fmod(absTime.seconds * 30, 360); set ry = 0.5 (APPROXIMATE, range 0.1–2). rx and rz stay 0.
2. noise1 (Noise TOP): Monochrome = Off (the tutorial uses colour noise so x, y, z move independently). Keep Derivative/gradient output on, offset 0, amplitude 0.5, period 1.
3. constant1: channel count = 1000 (1,000,000 particles). Report fps after the change; if it drops below 30 fps, use 700 and say so.
4. Particles must be tiny glowing points, not discs:
   - Preferred: inside geo_ps replace disc1 with addSOP (one point) -> convertSOP (to particles), and use a new pointspriteMAT named pointsprite1 as the geo material (alpha 0.3, point size small ~0.003–0.01). Point instancing stays on end (r/g/b).
   - If Point Sprite MAT does not render: keep disc1 but set radx = rady = 0.002 and say "REPLACEMENT".
   - Then set mat alpha back to 0.25–0.35 and level_color brightness1 to 1.0–1.5, tuned so strands are visible but the shell is not blown out.
5. camera1: tz = 4.6 so the whole sphere fits in frame with a margin.
6. Delete the empty default annotation named "annotate" (title "Annotate"). Do not recreate the audio chain.

Then: update the "What to edit" annotations for the changed values (ry, count, point size, alpha, brightness, tz).
Check and report:
- render out1 at t=3 s and t=10 s: is it a thin glowing shell of fine strands (no bright white band at the equator)? colour?
- fps and particle count;
- a table: operator | parameter | old -> new;
- export_network of /project1/stage_work (depth 3, with DAT text).
```

Ошибка самого промта: в пункте 4 размер точки задан как «0.003–0.01», но у Point Sprite MAT в TD 2025 включено увеличение вблизи камеры (Attenuate Point Scale, Near Point Scale = 20), поэтому точки стали огромными. Нужно было требовать `attenpscale = 0`.
