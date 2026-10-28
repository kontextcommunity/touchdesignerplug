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

## 7 · Аудио: чтение звука внутри audio_reader, наружу 4 канала (bass, level, high, kick)
`annotate1` · networkbox

## Что менять · аудио
`annotate11` · annotate

• Трек: audio_reader/audio_in.file (mp3/wav). Громкость: audio_reader/audio_out_dev.volume 0–1.
• Сглаживание всех каналов: audio_smooth.width, сек: 0.02–0.5 (0.06). Filter / Lag CHOP добавлять ТОЛЬКО после audio_ctrl или audio — там 1 значение на кадр, это дёшево.
• НЕ ставить Filter CHOP внутри audio_reader до *_rms: там звук 44 100 значений в секунду, фильтр шириной в секунды подвешивает проект.
• Чувствительность: audio_reader/*_norm fromrange2 — bass 0.42, level 0.42, high 0.07. Меньше число — сильнее реакция. Удар: kick_trig threshup 0.75 (0.6–0.9).
Куда идёт звук (менять множители в выражениях):
• Дыхание шара: math_len.gain = 1/(0.82 + 0.08*bass + 0.05*kick)
• Вспышки «звезды»: math_len.preoff = −0.035*kick (0 … −0.1)
• Скорость потока: math_step.gain = 0.0008 + 0.0035*level
• Искры и свечение по верхам: pointsprite1.alpha, bloom1.bloomintensity; яркость level_color.brightness1
Не трогать: имя audio и каналы bass / level / high / kick — на них ссылаются выражения.

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

## Что менять · блок 6
`annotate9` · annotate

• level_color.highr / highb — оттенок: 0.82 / 0.8 (бледно-зелёный); brightness1 — выражение от громкости (1.3–1.8)
• bloom1.preblacklevel — что светится: 0.5–0.9 (0.7); bloomintensity — выражение от верхов и удара
• bg1 — цвет фона (0.02 / 0.02 / 0.035)
Не трогать: over1 (bloom поверх фона), имя out1.
