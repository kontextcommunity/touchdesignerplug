# touchdesignerplug
## База знаний (knowledge/)

- `knowledge/rules/` — правила, всегда в промте агента.
- `knowledge/gotchas/` — грабли, всегда в промте агента.
- `knowledge/recipes/` — рецепты (.md и .json).
- `knowledge/notes/` — заметки, только поиск.

При пуше в `main` workflow `.github/workflows/release.yml` упаковывает `knowledge/` в `knowledge.zip` и публикует релиз. Плагин kontext.stage забирает его при запуске.
