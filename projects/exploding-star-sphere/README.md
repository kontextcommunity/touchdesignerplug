# Exploding Star — исходные файлы сборки

Инструкция: `knowledge/recipes/exploding-star-sphere.md`. Полный ход сборки и разбор ошибок: `knowledge/notes/projects/exploding-star-sphere.md`.

| Файл | Что это |
|---|---|
| `stage_work_export.json` | Текущая рабочая сеть `/project1/stage_work` (export_network, глубина 3, с текстом DAT). Сфера вынесена из петли, значения из видео 720p. |
| `history/prompt_1_build.md` | Промт 1: сборка с нуля (разбор по копии 640×360, сфера ошибочно внутри петли). |
| `history/01_plugin_first_build.json` | Что собрал плагин по промту 1: аудио-реактивный GLSL, 200×200 — не туториал. |
| `history/prompt_2_rebuild.md` | Промт 2: переделка сцены под туториал. |
| `history/02_plugin_rebuild_prompt.json` | Результат промта 2: структура есть, 6 ошибок (угол в петле, Monochrome, точки-диски и т.д.). |
| `history/prompt_3_fix.md` | Промт 3: правка 6 пунктов. |
| `history/03_plugin_fix_prompt.json` | Результат промта 3: пункты выполнены, но шар залит (увеличение Point Sprite). |
| `frames/02_rebuild_dots_equator_band.jpg` | Кадр после промта 2: крупные точки, полоса по экватору. |
| `frames/03_fix_solid_sphere.jpg` | Кадр после промта 3: сплошная заливка. |
| `frames/04_tuned_but_sphere_in_loop.jpg` | Подстройка рендера: нити есть, но сфера в петле — равномерно. |
| `frames/05_free_column_sheets.jpg` | Свободная симуляция без сферы: чёткие полотна, как у автора. |
| `frames/06_sphere_after_loop.jpg` | Сфера после петли: итоговый вид. |

Кадры — наши рендеры. Кадров из видео автора здесь нет.
