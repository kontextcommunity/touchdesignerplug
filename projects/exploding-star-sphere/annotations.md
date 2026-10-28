# Exploding Star — подписи в сети (аннотации)

`export_network` не сохраняет тексты аннотаций, поэтому они здесь. Те же данные с координатами — в `annotations.json`. Рамки групп (Mode = networkbox) обводят блоки; заметки (Mode = annotate) — «Что менять».

## 1 · Облако точек (труба → sprinkle → текстура 1000×1000)
`annotate10` · networkbox

## 2 · Петля позиций (feedback)
`annotate7` · networkbox

## 3 · Движение: шум-поток, маска, вращение
`annotate3` · networkbox

## 4 · Сфера — ПОСЛЕ петли (только для рендера)
`annotate4` · networkbox

## 5 · Рендер точек (point sprites)
`annotate5` · networkbox

## 6 · Цвет, свечение, выход
`annotate8` · networkbox

## 7 · Аудио: бас, громкость, верха, удар
`annotate1` · networkbox

## Что менять · аудио
`annotate11` · annotate

• audio_in.file — трек (mp3/wav). Сейчас пример из папки TouchDesigner. Звук идёт в audio_out_dev (volume 0–1).
• Диапазоны полос (fromrange2 у *_norm): bass 0.42, level 0.42, high 0.07. Меньше число — сильнее реакция.
• Сглаживание (*_lag lag2, сек): bass 0.35, level 0.6, high 0.2. Больше — плавнее.
• Удар: kick_trig threshup 0.75 (0.6–0.9), decay 0.25 сек.
Куда идёт звук (выражения, менять множители):
• Дыхание шара: math_len.gain = 1/(0.82 + 0.08*bass + 0.05*kick) — 0.82 базовый радиус.
• Вспышка «звезды» на удар: math_len.preoff = −0.035*kick (0 … −0.1; больше — сильнее лучи).
• Скорость потока: math_step.gain = 0.0008 + 0.0035*level.
• Искры по верхам: pointsprite1.alpha = 0.35 + 0.35*high; bloom1.bloomintensity = 0.25 + 0.55*high + 0.4*kick.
• Яркость: level_color.brightness1 = 1.3 + 0.5*level. Вращение шара: geo_ps.ry = absTime.seconds*5.
Не трогать: имена audio_bass / audio_level / audio_high / audio_kick — на них ссылаются выражения.

## Что менять · блок 6
`annotate9` · annotate

• level_color.highr / highb — оттенок: 0.82 / 0.8 (бледно-зелёный); brightness1 — выражение от громкости (1.3–1.8)
• bloom1.preblacklevel — что светится: 0.5–0.9 (0.7); bloomintensity — выражение от верхов и удара
• bg1 — цвет фона (0.02 / 0.02 / 0.035)
Не трогать: over1 (bloom поверх фона), имя out1.

## Что менять · блоки 1–3
`annotate2` · annotate

• constant1.count — частиц = count²: 300–1000 (1000 → 1 000 000, ~60 fps)
• noise1.period — размер вихрей: 0.5–3 (1). ty = absTime.seconds*0.1 — скорость «плавания» потока: множитель 0.05–0.3
• math_step.gain — шаг за кадр: задан выражением от звука (0.0008 + 0.0035*level), без звука 0.0005–0.003
• noise_mask.period — размер областей вращения: 1–4 (2); вход маски = pos (позиции, не менять)
• threshold1.soften — мягкость маски: 0–1 (1)
• point_xform.ry — поворот ЗА КАДР, градусы: 0–1 (0 = чёткие полотна; больше 0.3 смазывает)
• life_pos uP z — жизнь частицы, сек: 6–30 (12); uP w — разброс перерождения: 0–1 (0.15)
• клавиша 1 — пересобрать частицы с нуля
Не трогать: fb_pos, fb_src, pos, end, start, pos_sim_pixel.

## Что менять · блоки 4–5
`annotate6` · annotate

• math_len.gain = 1/радиус — сейчас выражение от звука (база 0.82 → радиус ~1.2)
• math_len.preoff — формы «взрыва звезды»: −0.1…0 (сейчас −0.035*kick)
• pointsprite1.alpha — плотность/яркость: 0.2–0.8 (выражение от верхов); pointsize 0.5–2 (1)
• pointsprite1.attenpscale = 0 и destblend = one — НЕ трогать (иначе точки огромные / нет свечения)
• camera1.tz — размер шара в кадре: 4–8 (5.4)
• geo_ps.ry — вращение шара: absTime.seconds*5 (множитель 0–20)
Не трогать: comp_div (деление на длину), end как вход math_len.
