# image

AI image generation skill for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) and [Codex CLI](https://github.com/openai/codex). Uses OpenAI's `gpt-image-2` model via Codex CLI's built-in `image_generation` tool.

---

Скилл генерации изображений для Claude Code и Codex CLI. Использует модель `gpt-image-2` через встроенный `image_generation` tool в Codex CLI. GPT-5.5 выступает reasoning-движком.

## Возможности

- Генерация PNG по текстовому описанию на любом языке
- Claude / Codex автоматически расширяет краткое описание в детальный prompt
- Три размера: квадрат (1024×1024), портрет (1024×1536), пейзаж (1536×1024)
- Автовыбор ориентации по контексту описания
- Content policy handling (понятные сообщения при отклонении)

## Требования

- [Codex CLI](https://github.com/openai/codex) (`npm i -g @openai/codex`)
- `OPENAI_API_KEY` с доступом к Images API
- `codex-exec.sh` — exec-скрипт (входит в [duo](https://github.com/aogoro/duo) или берётся из `.agents/scripts/`)

## Установка

```bash
git clone https://github.com/aogoro/image.git /tmp/image-skill
```

### Куда положить

Репо содержит три папки: `.agents/`, `.claude/`, `.codex/`. Их можно разместить двумя способами:

**В рабочую папку проекта** — скилл виден только в этом проекте:
```bash
cd ~/my-project
cp -r /tmp/image-skill/.agents/skills/image .agents/skills/image
cp -r /tmp/image-skill/.claude/skills/image .claude/skills/image
cp -r /tmp/image-skill/.codex/skills/image .codex/skills/image
```

**В домашнюю директорию (`~/`)** — скилл глобальный, виден из любого проекта в Claude Code CLI, Cursor, VS Code, desktop-app:
```bash
cp -r /tmp/image-skill/.agents/skills/image ~/.agents/skills/image
cp -r /tmp/image-skill/.claude/skills/image ~/.claude/skills/image
cp -r /tmp/image-skill/.codex/skills/image ~/.codex/skills/image
```

> Если у вас уже есть `.agents/`, `.claude/`, `.codex/` с другими скиллами — копируйте только `skills/image/` в соответствующие папки.

## Использование

### AI-скилл

После установки Claude Code и Codex автоматически подхватывают скилл. Триггеры:

- `/image кот в космосе`
- «нарисуй логотип для кофейни»
- «generate image of a sunset over mountains»
- «сгенерируй картинку ...»
- «draw a ...»

### Как это работает

1. Хост-LLM (Claude или Codex) расширяет краткое описание в детальный image prompt
2. Подготавливает промпт-файл по шаблону `prompts/generate.md`
3. Вызывает `codex-exec.sh` с `--sandbox workspace-write`
4. Codex CLI генерирует изображение через `image_generation` tool (gpt-image-2)
5. PNG сохраняется в `.temp/image/<slug>.png`
6. Хост показывает картинку пользователю

### Размеры

| Размер | Когда |
|--------|-------|
| 1024×1024 | Квадрат (default): логотипы, абстрактные сцены |
| 1024×1536 | Портрет: люди, здания, постеры |
| 1536×1024 | Пейзаж: панорамы, баннеры, горизонтальные сцены |

Хост автоматически выбирает ориентацию по контексту описания.

## Vendor-agnostic архитектура

Скилл построен по 3-tier vendor-agnostic архитектуре:

```
.agents/skills/image/          <- Tier 1: канон (vendor-neutral)
  REFERENCE.md                   source of truth
  prompts/generate.md            промпт-шаблон для Codex

.claude/skills/image/          <- Tier 2: тонкая обёртка Claude Code
  SKILL.md                       ~80 строк: парсинг аргументов + Claude I/O

.codex/skills/image/           <- Tier 3: тонкая обёртка Codex CLI
  SKILL.md                       ~40 строк: парсинг аргументов + Codex I/O
```

**Канон** (`.agents/`) содержит алгоритм, параметры, контракт. Vendor-neutral — не упоминает конкретные tools.

**Обёртки** (`.claude/`, `.codex/`) — тонкие адаптеры. Маппят vendor-neutral операции на инструменты конкретного хоста. Ссылаются на канон, не дублируют.

## Стоимость

Одна генерация через gpt-image-2: ~$0.04–0.19 в зависимости от размера и качества (определяется моделью автоматически).

## Лицензия

MIT.
