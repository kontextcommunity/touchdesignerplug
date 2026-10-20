# 3D аудио-реактивный блоб (sphere, noise, PBR, bloom)

**Теги:** 3d, audio, pbr, noise sop, bloom, lights
**Визуал:** тихо — тёмно-синий блоб с мягким гало; на ударе — bloom-вспышка, рваная форма, яркий key light.

## Аудио
audiodevicein → analyze(rmspower) → filter(width 0.05) → math(fromrange 0..0.25 → torange 0..1) → null `amp`.

## Геометрия
sphereSOP (type mesh, 50×50, radius 1) → noiseSOP (brownian, period 2.5, harmon 2)
- amp = 0.15 + amp * 0.5
- tx = math.sin(absTime.seconds*0.05)*2; ty = math.cos(absTime.seconds*0.04)*1.5; tz = math.sin(absTime.seconds*0.06)*2
- display = True, render = True на noiseSOP.
- geo.ry = math.fmod(absTime.seconds*20, 360)

## Материал PBR
- basecolor b = 0.18, metallic 0.95, roughness 0.06 (чёткие блики для bloom)
- emit b = 0.18 + amp*0.7, emit g = 0.02 + amp*0.35, emit r = amp*0.4

## Свет и камера
- lt_key (голубой, tx −2, ty 3, tz 5): dimmer = 3 + amp*5
- lt_fill (розово-фиолетовый, tx 3, ty −1, tz 3): dimmer 2
- cam: tz 11, rx −5

## Пост
render → lv_bloom (inlow = max(0.25, 0.4 − amp*0.2), brightness1 = 3.5 + amp*6) → blur (size = 50 + amp*120) → comp(add) → lv_out (brightness 0.82, gamma 0.86, contrast 1.35) → out.

Урок: emission делает плоский силуэт; тёмный материал + яркие лайты лучше.

Источник: Obsidian Claude Builds/3D Audio-Reactive Noise Blob.md (готов, 2026-08-03).
