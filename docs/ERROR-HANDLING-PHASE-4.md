# Fase 4 — Correções por step + apperrors enriquecidos

## Gemini `analyze.gemini.url` (HTTP 400)

`gemini-3.5-flash` pode rejeitar `fileData.fileUri` com URL YouTube (`INVALID_ARGUMENT`).

**Opções:**

1. Pipeline `analyze_only` / URL: baixar metadados/transcript leve antes do Gemini (sem `file_uri`).
2. Usar modelo com suporte explícito a vídeo URL (ex. família 2.5/3.x com URL context) — validar na doc Google.
3. Fallback automático: se 400 em URL analyze → enfileirar `transcribe.run` + `analyze.transcript`.

## Go API

- `apperrors.Error` opcional `Details map[string]any` para ops HTTP.
- Service/repo: wrap DB errors com `operation` (ex. `run.repository.GetByID`).
