# Дерево создания проекта

Алгоритм для агента:
1. Определить тип: 2D noise (A) / 3D geo (B) / FBX-particles (C) / media (D).
2. Выбрать ближайший шаблон из projects/index.md и рецепт.
3. Взять базовый паттерн из patterns.md.
4. Аудио: kick → яркость/reset/вспышка; low → медленные параметры (displace, hue); mid → gamma, sat, scale; high → детализация (edge, blur size, gain); rythm → дисторшн/UV.
5. Пост по стилю: свечение → bloom (cross1 = kick); цвет → GLSL HSV или ramp + lookup; аберрация → rgbaDelay; симметрия → mirror; мягкость → lumablur.
6. Финал: displace/render → [glsl/lookup] → level(brightness = kick) → null1 → syphon / moviefileout.

Ветка A (2D, есть аудио):
- один noise → жидкий нойз (liquid-noise)
- несколько noise layers → ACID (acid-noise)
- noise + edge cascade → каскад (edge-cascade)
- без аудио: + LUT → свечение (glow-lut)

Ветка B (3D): SOP-геометрия → instancing; источники: audio (lines-instancing), video/blobtrack (blobtrack-instancing), particles (chromatic-particles). FBX → fbx-particles.
Ветка D: text/footage → text-tunnel.

Рабочие комбинации (проверено):
- simplex4d + 2D texture + audio→exp/offset + bloom.cross1 + GLSL HSV = liquid-noise
- simplex3d + 2D texture + 5×edge + feedbackTOP + kick + ramp LUT = glow-lut
- perlin2d + line SOP + audio→gain/amp + bloom + rgbkey = lines-instancing
- video + POP copy + topto→points + opacity feedback + lumablur = flower-ocean
- particle SOP + audio transform + comp feedback + rgbaDelay + bloom = chromatic-particles
- 4× simplex + 2D add/multiply + kick→edge.blacklevel + feedbackTOP = acid-noise
- FBX + sprinkle → GPU particles + bloom×2 + rgbkey + add = fbx-particles

Источник: Obsidian TouchDesigner/Архитектура/Дерево Создания.md.
