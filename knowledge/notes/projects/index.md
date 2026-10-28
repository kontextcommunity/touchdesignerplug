# Проекты TouchDesigner (индекс)

| Проект | Группа | Техника | Аудио | Порт (старый MCP) | Рецепт |
|---|---|---|---|---|---|
| Жидкий Нойз | A 2D noise | simplex4d ×2 → edge → displace → bloom + GLSL HSV | 5 каналов | 9980/9981 | liquid-noise |
| Текстовый Тоннель | D media | text → tex3d → tima | нет | 9982 | text-tunnel |
| Каскад Краёв | A 2D noise | noise → 5×edge → dual feedback | kick reset | 9983 | edge-cascade |
| ACID Нойз | A 2D noise | 4× noise → edge(kick) → feedback | 4 канала | 9984 | acid-noise |
| Цветное Свечение | A 2D noise | noise ×2 → 3×edge → feedbackEdge → LUT ramp | нет/keyboard | 9985 | glow-lut |
| Цветочный Океан | C FBX/POP | video → topto → POP copy → lumablur | нет | 9986 | flower-ocean |
| Пылающий Паук | C FBX/POP | FBX ×2 → deform → sprinkle → GPU particles × bloom | нет | 9987 | fbx-particles |
| Линии в Пространстве | B 3D geo | audio → noise → displace → line SOP instanced | 2 канала | 9988 | lines-instancing |
| Слежение за Формами | B 3D geo | video → blobtrack → datto → box SOP instanced | mid cue | 9989 | blobtrack-instancing |
| Хроматические Частицы | B 3D geo | particle SOP → feedback → mirror → rgbaDelay | 5 каналов | 9990 | chromatic-particles |
| Hexagon Instancing | B 3D geo | copy SOP → instancing → render | нет | 9988 | hexagon-instancing |
| 3D Audio-Reactive Noise Blob | 3D audio | sphere SOP + noise SOP + PBR + bloom | rms | — | audio-reactive-blob |
| Exploding Star (частицы на сфере) | B 3D geo / GPU | позиции в TOP-петле (feedback + noise derivative) → нормализация на сферу после петли → point sprites | бас, громкость, верха, удар (аудио-версия) | — | exploding-star-sphere |

Группы:
- A — 2D noise + feedback: noise(s) → edge/blur → composite → feedback → color/level → output.
- B — 3D instanced geometry: source → instancetop → geo SOP → render → post → output.
- C — FBX/POP particles: fbx/video → sprinkle/topto → particlesGpu/copy → bloom → rgbkey → composite.
- D — text/graphics 2D: text/footage → transform → tex3d → tima → ramp/key → output.
