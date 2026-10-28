# Exploding Star — частицы на сфере: инструкция по туториалу (built)

- **Статус:** draft → **built** → verified → published
- **Источник:** YouTube, «Exploding Star — constraining a particle system to a sphere TOUCHDESIGNER TUTORIAL», 14:46. Значения сверены по исходнику 1280×720 (кадры 1:15–14:30).
- **Версия TouchDesigner в видео:** 2023.11340 · **у нас:** 2025.31550 (macOS), собрано и проверено вживую.
- **Результат:** миллион частиц летят как дым по шумовому потоку, а при рендере проецируются на сферу. Получаются светящиеся бледно-зелёные полотна и нити на поверхности шара с пустыми областями между ними. Pre-Add/Post-Add у Math даёт формы «взрывающейся звезды».
- **Теги:** 3d / particles / gpu / feedback / noise / instancing / point-sprite
- **История сборки и ошибки:** `knowledge/notes/projects/exploding-star-sphere.md`. Исходные JSON всех версий и кадры: `projects/exploding-star-sphere/` в репозитории.

## 1. Обзор

Частицы — пиксели текстуры позиций 1000×1000, 32-bit float (r, g, b = x, y, z). Текстура крутится в петле Feedback TOP. Шум двигает позиции на маленький шаг каждый кадр, маска поворачивает часть частиц, Life возвращает старые частицы в начальную трубу.

**Главное правило:** нормализация на сферу стоит **после** петли и нужна только для рендера. Внутри петли частицы свободно летят в объёме. Если поставить сферу внутрь петли, получится равномерная заливка шара без полотен — на этом сборка ломалась.

```
tube1 → sprinkle1 (count²) → null_sop1 → sopto1 → null_chop1 → chopto1 → start
start → fb_pos (Feedback: target fb_src, reset = клавиша 1) → pos
pos → noise1 (Perlin 3D, Derivative, поток плывёт по ty) → math_step (×0.001)
pos + math_step → add_noise (Composite Add)
pos → noise_mask → threshold1 (мягкий) → math_range (−1…1) ─┐
add_noise → point_xform (вход 2 = маска) ←───────────────────┘
point_xform → life_pos (GLSL, вход 2 = start) → end → fb_src   ← конец петли
end → math_len (Length, ×2) ;  end / math_len → comp_div → math_shape → geo_ps (instancing)
geo_ps → render1 → level_color → bloom1 → over1 (поверх bg1) → out1
```

## 2. Подготовка
- Ассетов нет. Аудио в базовой версии нет.
- Внешний компонент автора «Life» (платный, Patreon) заменён GLSL TOP `life_pos`, код ниже.
- Нужна сборка 2023+: Noise TOP с выходом Derivative, Point Transform TOP, Bloom TOP.
- Таймлайн должен играть: на паузе петля и агент стоят.

## 3. Пошаговая сборка

### Блок 1 — Облако точек (видео 1:15–2:39)
| Нода | Тип | Параметры | Вход | Видео | Что менять |
|---|---|---|---|---|---|
| tube1 | tubeSOP | `rad1=rad2=0.1`, height 1, orientation Y | — | 4:48 радиус 0.1 | радиус 0.05–0.3: толщина начального столба |
| constant1 | constantCHOP | `const0name=count`, `const0value=1000` | — | 1:39 | 300–1000, частиц = count² |
| sprinkle1 | sprinkleSOP | method Surface, `numpoints = op('constant1')['count']**2` | tube1 | 1:30 | — |
| null_sop1 | nullSOP | — | sprinkle1 | | не трогать |
| sopto1 | soptoCHOP | SOP = null_sop1, Position XYZ | — | 2:15 | — |
| null_chop1 | nullCHOP | — | sopto1 | | не трогать |
| chopto1 | choptoTOP | CHOP = null_chop1, `dataformat=rgb`, layout square, 32-bit float RGBA | — | 2:30 | — |
| start | nullTOP | — | chopto1 | | не трогать: на него ссылаются fb_pos и life_pos |

### Блок 2 — Петля позиций (видео 2:50–5:21)
| Нода | Тип | Параметры | Вход | Что менять |
|---|---|---|---|---|
| fb_pos | feedbackTOP | Target TOP = `fb_src`, `reset = op('key_reset')['k1']`, 32-bit float, разрешение = start | start | не трогать |
| key_reset | keyboardinCHOP | клавиша `1` | — | другая клавиша — в Keys |
| pos | nullTOP | — | fb_pos | не трогать: вход шума, маски и сумм |
| end | nullTOP | — | **life_pos** (не сфера!) | не трогать |
| fb_src | nullTOP | — | end | не трогать |

### Блок 3 — Поток шума (видео 5:36–7:42)
| Нода | Тип | Параметры | Вход | Видео | Что менять |
|---|---|---|---|---|---|
| noise1 | noiseTOP | `type=perlin3d`, period 1, harmonics 2, spread 2, gain 0.7, exponent 1, `amp=1`, `offset=0`, **`mono=False`**, aspect correct On, Output: **Derivative (`gradient=True`)**, `ty = absTime.seconds*0.1`, 32-bit float | pos, pos | 6:27, 6:39, 7:42 | period 0.5–3: размер вихрей; множитель в ty 0.05–0.3: скорость «плавания» потока |
| math_step | mathTOP | `gain=0.001` | noise1 | 5:51 | 0.0005–0.003: шаг за кадр (больше — быстрее и хаотичнее) |
| add_noise | compositeTOP | `operand=add`, 32-bit float | pos + math_step | 6:00 (у автора Add TOP) | — |

### Блок 4 — Вращение по маске (видео 8:18–9:12)
| Нода | Тип | Параметры | Вход | Видео | Что менять |
|---|---|---|---|---|---|
| noise_mask | noiseTOP | `type=perlin3d`, period 2, harmonics 10, spread 2, gain 0.7, `amp=1`, `offset=0`, `mono=False`, `ty = absTime.seconds*0.1` | **pos** (позиции, не индекс пикселя) | 9:09, 13:45 | period 1–4: размер областей вращения |
| threshold1 | thresholdTOP | comparator **Less**, RGB Luminance, `threshold=0`, **`soften=1`** | noise_mask | 8:40–8:50 | soften 0–1: резкость границ |
| math_range | mathTOP | To Range −1…1 | threshold1 | 8:54 | — |
| point_xform | pointtransformTOP | Input Type Position, `ry=0` (у автора 0.9) | add_noise, math_range | 8:20–8:35 | `ry` 0–1 **за кадр**: 0 — чёткие полотна, больше 0.3 — смазывание |

### Блок 5 — Life (замена компонента автора)
`life_pos` — glslTOP, входы: point_xform и start; 32-bit float, разрешение = start. Uniform `uP` (vec0): x = `absTime.seconds`, y = `1/root.time.rate`, z = 12, w = 0.15.

```glsl
uniform vec4 uP;   // x = time, y = dt, z = life seconds, w = spread of respawn times
out vec4 fragColor;
float hash(vec2 p){ return fract(sin(dot(p, vec2(127.1, 311.7))) * 43758.5453); }
void main(){
    vec2 uv = vUV.st;
    vec4 cur = texture(sTD2DInputs[0], uv);
    vec4 st  = texture(sTD2DInputs[1], uv);
    float h  = hash(uv) * uP.w;
    float g1 = floor(uP.x / uP.z + h);
    float g0 = floor((uP.x - uP.y) / uP.z + h);
    fragColor = vec4(g1 != g0 ? st.rgb : cur.rgb, 1.0);
}
```
Что менять: `uP.z` 6–30 (у автора lifetime 12, позже 9 — 4:57, 11:48). `uP.w` 0–1: 0 — все частицы перерождаются разом (пульс), 1 — непрерывный поток. Поток даёт туман, поэтому держать 0.1–0.3.

### Блок 6 — Сфера (видео 10:33–14:21) — после петли
| Нода | Тип | Параметры | Вход | Видео | Что менять |
|---|---|---|---|---|---|
| math_len | mathTOP | Combine Channels = **Length** (`chanop=len`), input mask RGB, mono 32-bit float, `gain=2` | end | 10:33, 10:45, 12:03 | `gain` 1–3 = 1/радиус (2 → радиус 0.5); **`preoff` / `postoff` −0.1…0.1 — формы «взрыва»** (у автора Pre-Add −0.08, Post-Add −0.07, Multiply 2–3) |
| comp_div | compositeTOP | `operand=divide`, fixed layer = input 1, 32-bit float | end, math_len | 10:51–11:00 | не трогать |
| math_shape | mathTOP | по умолчанию | comp_div | | запасной множитель радиуса |

### Блок 7 — Рендер точек (видео 3:33–4:39, 7:27, 13:36)
| Нода | Тип | Параметры | Что менять |
|---|---|---|---|
| geo_ps | geometryCOMP | instancing On, Translate OP = **math_shape**, r/g/b, count = `op('pos').width * op('pos').height`. Внутри `add_pt` (addSOP, одна точка) → `convert_pt` (convertSOP → particles) | — |
| pointsprite1 | pointspriteMAT | **`attenpscale=0`** (у автора Off), `pointsize=1`, `alpha=0.5`, blending: `srcblend=sa`, `destblend=one` (сложение) | alpha 0.2–0.8: плотность и яркость; pointsize 0.5–2 |
| camera1 | cameraCOMP | `tz=5` | 3–8: размер шара в кадре |
| render1 | renderTOP | camera1, geo_ps, 1920×1080, rgba16float | — |

### Блок 8 — Цвет и свечение (видео 7:51–11:27)
| Нода | Тип | Параметры | Что менять |
|---|---|---|---|
| level_color | levelTOP | `brightness1=1.5`, RGBA: `highr=0.82`, `highg=1`, `highb=0.8` | high r/g/b: оттенок (бледно-зелёный у автора) |
| bloom1 | bloomTOP | `preblacklevel=0.7`, `bloomintensity=0.3`, min radius 0.5, max radius 1, output Input + Bloom | intensity 0.2–1; pre-black 0.5–0.9: что светится |
| bg1 + over1 | constantTOP + overTOP | фон 0.02/0.02/0.035, bloom1 поверх bg1 | цвет фона |
| out1 | nullTOP | — | не переименовывать |

## 4. Вариант: крупнее и аудио-реактивный (наша доработка, не из видео)
Сеть: `projects/exploding-star-sphere/stage_work_export_audio.json`.

**Крупнее:** `math_len.gain` 1 (радиус 1), `camera1.tz` 5.4 — шар около 80% высоты кадра.

**Аудио-цепочка** (блок «7 · Аудио»). Весь звук читается внутри Base COMP `audio_reader`, наружу выходят только 4 управляющих канала, 1 значение на кадр:
```
audio_reader/ (Base COMP — внутри звук 44.1 кГц)
  audio_in (audiofileinCHOP, sequential, repeat) → audio_out_dev (звук наружу, volume 0.8)
  audio_in → bass_filt (lowpass 150 Гц) → bass_rms (analyze rmspower) → bass_mono (math avg) → bass_lag (0.02 / 0.35 с) → bass_norm (0.05…0.42 → 0…1) → bass_clamp → bass_name
  audio_in → level_filt → level_rms → … → level_lag (0.05 / 0.6) → level_norm (0.05…0.42) → level_name
  audio_in → high_filt (highpass 4000 Гц) → high_rms → … → high_lag (0.01 / 0.2) → high_norm (0.006…0.07) → high_name
  bass_mono → kick_lag (0 / 0.06) → kick_norm (0.18…0.4) → kick_trig (threshup 0.75, down 0.4, attack 0, decay 0.25) → kick_name
  merge_ctrl → out_ctrl (Out CHOP)
audio_reader → audio_ctrl (null) → audio_smooth (Filter CHOP, gauss, width 0.06 с) → audio (null: bass, level, high, kick)
```
Все выражения берут `op('audio')['bass']`, `['level']`, `['high']`, `['kick']`.

**Почему так:** Filter CHOP, поставленный на сырой звук (`audio_in`, 44 100 значений в секунду, 735 на кадр), считает свёртку по всем сэмплам каждый кадр. При ширине фильтра в доли секунды и больше проект подвисает. После `*_rms` и Analyze остаётся одно значение на кадр, и Filter / Lag на нём почти бесплатный (замер: Filter width 1 с на `audio` — 0.34 мс, ~58 fps). Поэтому любые новые Filter / Lag / Math ставить после `audio_ctrl` или `audio`, а не внутри `audio_reader`.

| Куда | Выражение | Что менять |
|---|---|---|
| Дыхание шара | `math_len.gain = 1/(0.82 + 0.08*bass + 0.05*kick)` | 0.82 — базовый радиус; множители 0–0.2 |
| Вспышки «звезды» на удар | `math_len.preoff = -0.035*kick` | 0 … −0.1. Больше −0.06 — лучи вылетают за кадр |
| Скорость потока | `math_step.gain = 0.0008 + 0.0035*level` | множитель 0.001–0.005 |
| Искры по верхам | `pointsprite1.alpha = 0.35 + 0.35*high` | — |
| Свечение | `bloom1.bloomintensity = 0.25 + 0.55*high + 0.4*kick` | — |
| Яркость | `level_color.brightness1 = 1.3 + 0.5*level` | — |
| Вращение | `geo_ps.ry = absTime.seconds*5`, `geo_ps.rx = math.sin(absTime.seconds*0.13)*12` | множители |

Где `bass = op('audio')['bass']` и так далее. Общее сглаживание — `audio_smooth.width` (0.02–0.5 с). Диапазоны `audio_reader/*_norm` подобраны под пример из папки TouchDesigner (`Audio/JeremyCaulfield…mp3`). Для своего трека смотреть пики `audio_reader/*_rms` и ставить `fromrange2` чуть ниже пика. Первая версия с `preoff = -0.10*kick` и радиусом 1.18 выбрасывала частицы далеко за кадр (кадр `08` снят уже после правки).

## 5. Отклонения от видео
- Life автора недоступен, заменён GLSL `life_pos` (блок 5). Как именно его компонент перерождает частицы, по видео не видно.
- `point_xform.ry = 0` вместо 0.9: с вращением полотна у нас размазываются, причина пока не найдена.
- Point Sprite: у нас сложение (`destblend=one`, alpha 0.5, size 1), у автора альфа-смешивание (Constant Point Scale 0.5, Alpha 1, Sorted Draw). Дымки у нас больше, пустот меньше.
- Цветной окантовки по краям, как в начале видео, нет: этот эффект в разобранных кадрах не найден.
- Render 1920×1080 16-bit float вместо 1280×720 8-bit.

## 6. Проверка
- `chopto1`, `pos`, `end` — 1000×1000, 32-bit float; fps 60 (M4 Pro), рендер около 1 мс.
- Кадр через 2–5 секунд после сброса (клавиша 1): шар из полотен и нитей с пустыми областями, без равномерной заливки и без яркой полосы на экваторе.
- `math_len.preoff = -0.08`: оболочка рвётся в лучи «звезды».
