# NoiseSquares — Realtime White Visuals: инструкция по туториалу

- **Статус:** draft → built → verified → published
- **Источник:** https://www.youtube.com/watch?v=8ABSC0gkJrU — Daniel Steenhoff, «Realtime White Visuals - TouchDesigner Tutorial (Beginner Friendly!)», 5:42, загружено 2024-04-05
- **Версия TouchDesigner в видео:** не указана в кадрах, проект называется `project1/NoiseSquares` · **у нас:** уточнить при сборке
- **Результат:** чёрно-белый feedback-визуал из шума, пикселизированный через Limit (Quantize Position = Round), с искажением через Displace. Разрешение 1280×1280, 60 fps, таймлайн 600 кадров (как в видео).
- **Теги:** 2d / feedback / noise / pixelation / realtime

Источник данных: субтитры видео (в репозиторий не включены) и кадры 00:20–01:50, сверенные с экраном. Главы 02:00–05:42 пока разобраны по тексту и требуют проверки по кадрам.

## 1. Обзор
Идея: шум → обрезка и пикселизация → смещение самого себя → два контура feedback с затуханием. Сеть маленькая (около 20–25 нод), всё в одном network.

```
noise1 → null1 → edge-цепочка → feedback → level → composite(add) → limit (Round) → displace → edge → feedback1 → level → composite(add) → RGB key → level → out
```

## 2. Подготовка
- Ассетов нет: всё генерируется шумом и TOP-нодами.
- Внешних компонентов нет.
- Аудио не нужно.
- Клавиша сброса feedback: в видео используется клавиша `1` (Keyboard In DAT → Pulse feedback).

## 3. Пошаговая сборка

### Блок 1 — Шум (видео 00:00–00:50)
| Нода | Тип | Параметры (внутренние имена) | Вход | Комментарий |
|---|---|---|---|---|
| noise1 | noiseTOP | `type=simplex3d` (Simplex 3D GPU), `period=5`, `harmonics=2`, `harmonicspread=2`, `harmonicgain=0.7`, `exponent=2.5` (по тексту видео: два с половиной), `amp=0.5`, `offset=0.5`, `monochrome=1`, `aspectcorrect=1` | — | Screen в 04:13 показывает Exponent = 1 во время правки. Финальное значение в видео — 2.5, проверить по кадру. |
| noise1 (Transform) | — | `transformorder=scale rotate translate` (в видео Scale Rotate Translate), `rotateorder=rxryrz`, `tz = absTime.seconds / 10` | — | Анимация: движение по оси Z шума. Выражение в tz на кадре 04:13 подтверждено. |
| noise1 (Common) | — | `outputresolution=custom`, `resolutionw=1280`, `resolutionh=1280` | — | В видео пока стоит Use Input 256×256 — финал 1280×1280. |
| null1 | nullTOP | — | noise1 | Alt+N (в видео). Выход для дальнейших веток. |

Уточнить по видео (00:25–00:48): в тексте упомянута анимация одной из осей с делителем 100 вместо 10 — какая именно ось (tx или ty) и какой делитель, проверить по кадрам. В кадрах 00:40–00:50 видно tz = `absTime.seconds/10`. Проверить.

### Блок 2 — Edge-цепочка и первый feedback (видео 00:50–01:50 по тексту)
| Нода | Тип | Параметры | Вход | Комментарий |
|---|---|---|---|---|
| edge1–edge4 | edgeTOP | `alpha`: edge1=1, edge2=1/3, edge3=1/4, edge4=1 (как в видео) | null1 (цепочка) | Порядок и значения альфы — по тексту, проверить по кадрам. |
| feedback1 | feedbackTOP | — | edge (последний) | Петля обратной связи (на экране назван feedBackLoop). |
| level1 | levelTOP | `opacity=0.8` (по тексту видео: 0.8), `brightness1` — по умолчанию | feedback1 | Затухание петли. Не трогать без нужды. |
| comp1 | compositeTOP | `operation=add` | level1 + null1 (вход 2) | Складывает шум и петлю. |
| transform1 | transformTOP | `scale ≈ 0.99` (по тексту видео: 0.99) | comp1 | Небольшой зум в петле. |
| key1 (RGB key) | rgbkeyTOP | — | transform1 | Чёрный фон (по тексту видео — RGB Key для фона). |
| level2 | levelTOP | `contrast`, `brightness`, `gamma` — по вкусу | key1 | Серый и контраст, финальная яркость. |

Примечание: в видео сначала идёт `comp` с `brightness 0.9` (перед feedback) для предотвращения «переполнения». Это вторая строка блока 1–2, проверить по кадру 01:10–01:20.

### Блок 3 — Limit и Displace (видео 01:50–03:15 по тексту)
| Нода | Тип | Параметры | Вход | Комментарий |
|---|---|---|---|---|
| limit1 | limitTOP | `quantizeposition=round` (Quantize Position = Round), `quantizepositionstep=0.18–0.35` | comp (вход 1) | Кадр 00:20: Position Step 0.181. Кадр 00:30: 0.352. Это главный регулятор визуала: 0.18 — мелкая сетка, 0.35 — крупная. |
| displace1 | displaceTOP | — | limit1 (вход 2) | В тексте назван display stop, по смыслу вероятно displace. Проверить по кадру. |
| edge5 | edgeTOP | `alpha=1` | displace1 | Контуры. |
| feedback2 | feedbackTOP | — | edge5 | Вторая петля (на экране feedBackLoop1). |

### Блок 4 — Вторая петля и финальный Composite (видео 03:15–05:42 по тексту)
| Нода | Тип | Параметры | Вход | Комментарий |
|---|---|---|---|---|
| level3 | levelTOP | `opacity=0.8` | feedback2 | Затухание второй петли. |
| transform2 | transformTOP | `scale ≈ 0.99` | level3 | Зум. |
| comp2 | compositeTOP | `operation=add` | transform2 + limit1 | Сборка. |
| keyboardin1 | keyboardinDAT | `key=1` | — | Pulse для reset feedback1 и feedback2. |

Проверка после блоков: без ошибок в panel verify; во вьюере видна сетка и контуры.

## 4. Выражения и связи
| Куда | Выражение | Зачем |
|---|---|---|
| `noise1.par.tz` | `absTime.seconds / 10` | движение шума по Z. В видео именно так (кадр 04:13). |
| `noise1.par.tx` или `ty` | `absTime.seconds / 100` | (проверить по видео 00:25–00:48) |

Про `absTime.seconds` в видео линейный рост. Наша сборка: при длинной работе значение растёт. См. риски.

## 5. Риск-менеджмент
| Риск | Вероятность | Влияние | Как заметить | Как избежать / исправить |
|---|---|---|---|---|
| Имена параметров в нашей версии TD отличаются | средняя | ошибка при set_params | ошибка set_params | брать имена из operator_info, не из видео |
| Exponent в видео 2.5, на кадре виден 1 | средняя | шум выглядит иначе | сравнить с кадром 04:13 | зафиксировать 2.5 и сравнить с видео |
| Position Step 0.18–0.35 — диапазон, а не точка | высокая | визуал отличается от видео | Limit на вьюере | выбрать значение по картинке, записать в «Отклонениях» |
| Резолюция 1280×1280 при feedback — тяжело для GPU | средняя | fps падает ниже 60 | Performance Monitor | понизить до 960 или 720 |
| Линейный рост `absTime.seconds` в выражениях | средняя | сцена «уезжает» через время | значения растут | `math.sin` / `math.fmod` |
| Feedback без затухания | высокая | белый экран | яркость растёт | level с opacity 0.8 и brightness < 1 в петле |
| Keyboard In не сбрасывает feedback | средняя | петля застревает | визуал не меняется по `1` | проверить Pulse в feedback |
| Ось/делитель в видео неясны (00:25–00:48) | высокая | движение не совпадает | сравнить анимацию по кадрам | сверить по видео и записать в «Отклонениях» |

## 6. Чек-лист приёмки
- [ ] verify без ошибок и предупреждений
- [ ] выход совпадает с видео по композиции (чёрный фон, белые контуры, сетка pixelation)
- [ ] анимация шума есть, по tz (absTime.seconds/10)
- [ ] fps не ниже 60 на нашем железе
- [ ] клавиша `1` сбрасывает feedback
- [ ] ручки для настройки перечислены (путь + параметр): `limit1.quantizepositionstep`, `noise1.period`, `level1.opacity`, `level3.opacity`

## 7. Отклонения от видео
Заполняется после сборки.
- Exponent: видео 2.5, на кадре 1 (во время правки).
- Resolution: видео 1280×1280, на кадре 256×256 (во время правки).
- Остальные пункты — по сборке.

## 8. Журнал
- 2026-10-21 — draft: инструкция по видео (субтитры + кадры 00:20–01:50)
- YYYY-MM-DD — built: собрано в TD
- YYYY-MM-DD — verified: подтверждено
- YYYY-MM-DD — published: в knowledge/recipes/
