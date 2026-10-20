# Гексагональный инстансинг (SOP + CHOP + TOP + COMP)

**Теги:** 3d, instancing, copy sop, lookup, fog
**Рендер:** 1000×1000. Порт MCP в старом проекте: 9988.

## Порядок построения
1. SOP: tube1/2/3 → boolean (A−B = кольцо) → facet → attribute → texture → geo
2. Инстансинг: grid SOP → line → copy → delete → transform → null
3. SOP→CHOP: sopto + topto → shuffle/sort/rename/replace/merge → instancing null (позиции SOP + цвет TOP в один CHOP)
4. Текстура: noise → level → lookup с ramp5 (серый шум → цвет) → fit → null
5. Render COMP: render + cam + light + env → постпро → выход

## Ключевое
- Hex grid: tube (cols 6) задаёт форму, copy по сетке точек создаёт поле.
- topto → rename(tz): канал из текстуры управляет высотой каждого гексагона.
- math4 (torange −2..2): нормализует шум в Z-смещение.
- Circular ramp4: радиальный переход цвета.
- Fog в камере: чёрный туман скрывает дальние ряды.

Источник: Obsidian Проекты/Hexagon Instancing.md.
