# Грабли при работе с kontext.stage (из практики сборки плагина)

- Open Window пустой: в Web Render TOP `source` — меню `urlorfile`, `url` — адрес. Window COMP: `size='custom'`, `winw`/`winh` заданы.
- Web Render: ввод клавиш через `sendString` и `sendKey`, а не через буфер. Иначе текст в окне не уходит в чат.
- Запуск Claude Code из TD: унаследованные переменные CLAUDE_CODE_* помечают процесс как вложенный. Перед запуском их удалять (см. child_env).
- Ключи API не хранить в .tox и не выводить в чат. Только в ~/.kontext-stage/settings.json (chmod 600).
- Порт MCP по умолчанию 9988, gate = порт+1. Если порт занят, задать gate_port в настройках.
- Аргумент `exec()` в Textport: вставлять только строку с точным путём, без плейсхолдеров и лишнего отступа в начале строки (IndentationError).
- Внутренний callback RequestConfirm: ключевое слово называть dkind, иначе конфликт `kind`.
