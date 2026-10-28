# Грабли TouchDesigner Python

## Выражения и параметры
- Параметры в expressions: `math.sin()`, `math.cos()`, `math.fmod()`. Голый `sin()` даёт ошибку `parameter r`.
- Не использовать линейный рост `absTime.seconds * k` для позиций и углов: поле шума уходит в миллионы, камера улетает. Использовать `math.sin(absTime.seconds * 0.05) * 2` или `math.fmod(absTime.seconds * 20, 360)`. Это для обычной анимации. Внутри петли обратной связи — наоборот, см. раздел «Петли обратной связи».
- Имена параметров — внутренние короткие имена (`resolutionw`, `winop`), не подписи UI. Брать из operator_info текущей сборки.
- Меню принимает имена пунктов (как в operator_info), не подписи.
- CHOP в выражении: `op('name')['chan']` или `op('name')[0]`.
- Аудио в audioSpectrum (режим visual) даёт значения до ~44000, а не 0..1. Для RMS использовать audiodeviceIn → analyze(rmspower) → filter → math(fromrange 0..0.25 → torange 0..1).

## Соединения
- Соединение COMP: `connect(comp.outputConnectors[0])`, а не `connect(comp)`.
- Ошибка «Not enough sources» у feedbackTOP при старте: заменить на встроенный feedback (bloom.cross1 или level с opacity < 1 внутри петли).
- Семейства не смешиваются: TOP→TOP, CHOP→CHOP, SOP→SOP. Между ними — конвертеры (topto, choptop, sopto, topto CHOP).

## Создание и отображение
- `appendCustomPage` работает только на COMP. Для GLSL передавать uniforms через constantTOP 1×1 (rgba32float).
- Поиск GLSL DAT: `p.op('glsl1_pixel')` на уровне /project1, а не внутри glslTOP.
- Geometry COMP, созданный с торусом по умолчанию: новые SOP имеют display=False и render=False. Явно ставить display=True и render=True на финальном SOP.
- Sphere SOP для noise: type = `mesh`, не `poly` (poly даёт только контур, ~42 точки).
- Noise SOP: type `brownian` даёт случайную деформацию. Sphere 60×60 + sparse noise даёт «мандалу» из 4 лепестков.
- Emission делает силуэт плоским. Лучше тёмный материал + яркие лайты: specular идёт в bloom.
- Bloom, заливающий кадр: поднять `inlow` (0.4+), иначе светится вся поверхность.

## Геометрия и данные
- FBX обычно в сантиметрах, TD — в метрах: transform scale 0.01 обязательно.
- Deform SOP: skelrootpath — абсолютный путь к FBX COMP.
- FBX без анимации: имя animation должно точно совпадать.
- Частицы в одной точке: sprinkle не подключён к deform, или нет scale.
- datto: имена колонок в blobtrack_blobs DAT смотреть через viewer. Если боксы в одной точке — переименование в reorder не сработало.
- Ortho camera: sy = resx/resy (из constantCHOP), иначе масштаб не совпадает с aspect.
- instancing TOP: R→tx, G→ty, B→tz или scale, A→rotation/opacity. Источник — noise или displace в rgba32float.

## Петли обратной связи (частицы на текстуре позиций)
- Всё, что стоит внутри петли Feedback TOP (Point Transform TOP, Transform TOP, сдвиг в GLSL, Math add), применяется заново каждый кадр. Значение — это шаг за кадр, а не итоговый угол. Ставить маленькую постоянную величину (поворот 0.1–2° за кадр). Растущий угол `math.fmod(absTime.seconds * 30, 360)` в петле проворачивает частицы на десятки градусов каждый кадр — хаос.
- Noise TOP: Monochrome по умолчанию On. Для смещения позиций (r, g, b = x, y, z) ставить Monochrome Off, иначе все три оси двигаются одинаково и частицы собираются на линию или полосу (яркий «экватор» на сфере).
- Noise TOP с входами (позиции на обоих входах) и Output = Derivative (gradient) даёт направление потока; умножать на маленький шаг (Math ×0.001), не на 1.
- Нормализация на сферу: Math TOP (Combine Channels = Length, только rgb) → Composite TOP Divide (позиции / длина). Каждая точка на радиусе 1; радиус меняется Math multiply после деления.
- Нормализация на сферу (Length → Divide) ставится **после** петли и идёт только в рендер. Внутри петли она прижимает симуляцию к сфере, и шар равномерно заливается вместо полотен и нитей.
- Шум-маска для Point Transform берёт на вход позиции (`pos`/`end`), а не пустую текстуру: иначе соседние в пространстве частицы получают разные повороты, и полотна размазываются.
- Количество частиц из туториала (`count**2`) не снижать «на всякий случай». Сначала замерить fps, снижать только при падении ниже 30 fps и писать об этом в отчёте.

## Рендер частиц
- Point Sprite MAT в TD 2025: Attenuate Point Scale по умолчанию увеличивает точки вблизи камеры (Near Point Scale 20). Для частиц ставить `attenpscale = 0`, для свечения — сложение (`srcblend=sa`, `destblend=one`).
- Для нитей и «дыма» нужны крошечные точки: Point Sprite MAT или геометрия размером ≤ 0.003. Круги 0.01 и больше дают «горошек» вместо нитей.
- Не прятать проблему альфой и яркостью (alpha 0.06 + Level ×6). Если кадр пересвечен или точки крупные — менять размер точки, количество, шум, а не компенсировать.
- Сферу радиуса 1 при камере `tz` 3.4 срезает по краям кадра 16:9. Для целой сферы с полями `tz` ≈ 4.5–5.

## Аудио-CHOP
- Filter CHOP (и Lag, Trail, Math с большим окном) на сыром звуке Audio File In / Audio Device In — это 44 100 сэмплов в секунду, 735 на кадр. Фильтр шириной в доли секунды на таком потоке подвешивает проект. Сначала свести звук к control rate: Audio Filter (полоса) → Analyze (RMS Power) → Math (Combine Channels Average) — получается 1 значение на кадр. Filter / Lag ставить только после этого.
- Удобно прятать всю audio-rate цепочку в Base COMP с Out CHOP и выводить наружу только управляющие каналы (`bass`, `level`, `high`, `kick`). Выражения ссылаются на внешний Null (`op('audio')['bass']`), и добавление фильтров снаружи ничего не ломает.

## Сеть и порты
- Web Server DAT: Callbacks DAT = полный путь к callbacks, иначе 502 Bad Gateway.
- Старые проекты Claude Builds использовали порты 9980–9990, у каждого проекта свой порт. Порт 9988 использовался также в «Линии в пространстве» и «Hexagon Instancing»: при запуске kontext.stage на 9988 проверить, что этот порт свободен.

Источник: Obsidian Vehcadrob/TouchDesigner (Паттерны и Техники, Известные ошибки, Проекты, 3D Audio-Reactive Noise Blob).
