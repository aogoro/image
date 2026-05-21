<!-- v1 · vendor-neutral · 2026-05-21 · anton -->
<!-- Промпт для Codex CLI: генерация изображения через image_generation tool. -->
<!-- Плейсхолдеры: {{PROMPT}}, {{OUTPUT_PATH}}, {{SIZE}} -->

Generate an image using the image generation tool based on the description below.
Save the result as a PNG file to the specified output path.
If the parent directory doesn't exist, create it first.

## Image description

{{PROMPT}}

## Output

- Path: {{OUTPUT_PATH}}
- Size: {{SIZE}}

After saving, verify the file exists and report its absolute path and byte size.
Do not output the image content as text. Only save it to disk.
