# image — vendor-agnostic канон скилла генерации изображений

<!-- v1 · vendor-neutral · 2026-05-21 · anton -->

Канонический слой для скилла `/image` — генерации изображений через Codex CLI (image_generation tool, модель gpt-image-2).

Тонкие обёртки для AI-агентов (размещаются рядом — в корне того же дерева, где лежит `.agents/`):
- Claude Code: `.claude/skills/image/SKILL.md`
- Codex CLI: `.codex/skills/image/SKILL.md`

## Что делает

Принимает текстовое описание → возвращает PNG-файл с изображением.

**Движок:** Codex CLI вызывает встроенный `image_generation` tool (модель `gpt-image-2`). GPT-5.5 выступает reasoning-движком, решает как интерпретировать промпт и какие параметры передать генератору.

## Механизм

1. Хост-LLM готовит промпт-файл по шаблону `prompts/generate.md`, подставляя плейсхолдеры.
2. Хост вызывает exec-скрипт: `codex-exec.sh <workdir> <prompt-file> <output-text-file> --sandbox workspace-write --timeout 120`.
3. Codex CLI генерирует изображение и сохраняет PNG в `<workdir>/.temp/image/<slug>.png`.
4. Хост читает PNG host file-read'ом и показывает пользователю.

Output-text-file содержит текстовое подтверждение от Codex (путь к файлу, размер в байтах). PNG записывается отдельно в workdir.

## Промпт-контракт

Шаблон: `.agents/skills/image/prompts/generate.md`

Плейсхолдеры:
- `{{PROMPT}}` — детализированное описание изображения (хост расширяет краткий запрос пользователя перед подстановкой)
- `{{OUTPUT_PATH}}` — абсолютный путь к PNG
- `{{SIZE}}` — размер (см. ниже)

## Параметры

**Sizes** (хост выбирает по контексту):
- `1024x1024` — квадрат (default, для абстрактных/неочевидных пропорций)
- `1024x1536` — портрет (вертикальный: люди, здания, постеры)
- `1536x1024` — пейзаж (горизонтальный: сцены, панорамы, баннеры)

**Sandbox:** `workspace-write` (обязательно — Codex записывает PNG на диск).

**Timeout:** 120 секунд (генерация одной картинки занимает 10-30 сек, с запасом).

## Exit codes exec-скрипта

| Код | Причина |
|-----|---------|
| 0   | OK — PNG записан, читай output-text-file для пути |
| 124 | Timeout (120 сек) |
| иное | Ошибка Codex (API, content policy, сеть) |

Stderr sidecar `${OUTPUT_TEXT_FILE}.stderr` содержит диагностику при non-zero exit.

## Content policy

OpenAI отклоняет промпты, нарушающие content policy (насилие, NSFW и т.п.). Codex вернёт non-zero exit code. Хост читает `.stderr` sidecar и сообщает пользователю понятным языком. Retry не нужен — политика детерминирована.

## Требования окружения

1. **Codex CLI** (`codex` в PATH), авторизованный через `codex login` (OAuth)
2. **exec-скрипт** — `$SKILL_ROOT/.agents/scripts/codex-exec.sh`

## Точки расширения (не v1)

- `--edit` — редактирование существующего изображения (inpainting)
- `--style <preset>` — пресеты стиля (photorealistic, illustration, pixel-art, watercolor)
- `--n <count>` — несколько вариантов
- `--variation` — вариации существующего изображения

## Файловая структура

```
<root>/
├── .agents/skills/image/
│   ├── REFERENCE.md          # этот файл (канон, vendor-agnostic)
│   └── prompts/
│       └── generate.md       # промпт-шаблон для Codex
├── .claude/skills/image/
│   └── SKILL.md              # тонкая обёртка для Claude Code
└── .codex/skills/image/
    └── SKILL.md              # тонкая обёртка для Codex CLI
```
