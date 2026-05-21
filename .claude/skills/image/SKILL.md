---
name: image
description: Генерация изображений через Codex CLI (gpt-image-2). Принимает текстовое описание, возвращает PNG. Triggers: "generate image", "create picture", "нарисуй", "сгенерируй картинку", "draw", "picture of", "сделай картинку".
disable-model-invocation: true
user-invocable: true
metadata:
  version: "1.0"
  author: "anton"
  category: "media"
---

# Генерация изображений через Codex (для Claude)

## When to use

Генерация изображений по текстовому описанию. Codex CLI вызывает встроенный image_generation tool (gpt-image-2), GPT-5.5 управляет процессом.

**Триггеры:** "нарисуй", "сгенерируй картинку", "generate image", "draw", "создай изображение", "picture of", "сделай картинку".

## Что делать

### 0. Определи `$SKILL_ROOT`

Найди расположение этого SKILL.md → поднимись на 3 уровня (`../../..`) → это `$SKILL_ROOT`. Все пути ниже — относительно `$SKILL_ROOT`.

Канон: `$SKILL_ROOT/.agents/skills/image/REFERENCE.md`.

### 1. Определи описание

Из `$ARGUMENTS` извлеки что пользователь хочет нарисовать. Если `$ARGUMENTS` пуст — спроси пользователя.

### 2. Enhance prompt

Разверни краткое описание в детальный image prompt на английском:
- Стиль (photorealistic, illustration, oil painting, etc.)
- Композиция (close-up, wide shot, aerial view, etc.)
- Освещение и настроение (warm, dramatic, soft, etc.)
- Детали (текстуры, материалы, фон)

Из «кот» → "A fluffy orange tabby cat sitting on a windowsill, warm afternoon sunlight streaming through the window, photorealistic, detailed fur texture, cozy indoor atmosphere". Из «логотип для кофейни» → "A minimalist logo for a coffee shop, a steaming coffee cup silhouette, clean vector lines, warm brown and cream color palette, modern design".

Не переусердствуй — 1-3 предложения достаточно.

### 3. Определи size

- **1024x1024** (default) — абстрактные сцены, логотипы, квадратные форматы
- **1024x1536** (portrait) — люди в полный рост, здания, постеры, вертикальные сцены
- **1536x1024** (landscape) — пейзажи, панорамы, баннеры, горизонтальные сцены

### 4. Подготовь промпт-файл

Прочитай шаблон Read tool'ом: `$SKILL_ROOT/.agents/skills/image/prompts/generate.md`.

Подставь плейсхолдеры:
- `{{PROMPT}}` — enhanced prompt из шага 2
- `{{OUTPUT_PATH}}` — `$SKILL_ROOT/.temp/image/<slug>.png` (slug из описания, латиницей, kebab-case)
- `{{SIZE}}` — из шага 3

Запиши Write tool'ом в `$SKILL_ROOT/.temp/image/prompt-<TEMP_ID>.md`.

`$TEMP_ID` — через `date +%s`.

### 5. Запусти Codex

```
bash $SKILL_ROOT/.agents/scripts/codex-exec.sh $SKILL_ROOT \
     $SKILL_ROOT/.temp/image/prompt-<TEMP_ID>.md \
     $SKILL_ROOT/.temp/image/output-<TEMP_ID>.txt \
     --sandbox workspace-write --timeout 120
```

### 6. Обработка результата

| Exit code | Действие |
|-----------|----------|
| 0 | Read output-txt для пути к PNG. Read PNG через Read tool — покажи пользователю. Затем `open <путь к PNG>` через Bash — откроет в Preview.app. Сообщи путь к файлу. |
| 124 | «Генерация заняла больше 2 минут — timeout. Попробуй снова.» |
| иное | Read `$SKILL_ROOT/.temp/image/output-<TEMP_ID>.txt.stderr` для диагностики. Сообщи ошибку. |

## Канон

Полное описание — в `.agents/skills/image/REFERENCE.md`. Здесь только Claude-специфика.
