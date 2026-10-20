# Текстовый тоннель (2D/медиа, tex3d + tima)

**Теги:** text, tunnel, tex3d, tima, rgbkey
**Когда:** «текстовый поток», «тайпографика в 3D», «брендинг».

## Граф
text1 (длинная строка, большой шрифт, res 9000×128)
→ transform1 (tx = time*speed, tile −5..6, extend repeat)
→ tex3d1 (cachesize = N кадров) + noise1
→ tima1 (blackoffset = −N*fps, samplemode depthoffset)
→ ramp1 (phase = time*speed) → rgbkey → output.

Источник: Obsidian Проекты/Текстовый Тоннель.md (порт 9982, без аудио).
