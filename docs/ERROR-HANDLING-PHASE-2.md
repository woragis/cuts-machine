# Fase 2 — Erros tipados nos módulos de domínio

## Objetivo

Cada módulo worker levanta `PipelineError` (ou subclass) com `code` estável:

| Módulo | Exemplos de `code` |
|--------|-------------------|
| `ingest.py` | `INGEST_YTDLP_FAILED`, `INGEST_VIDEO_MISSING` |
| `gemini.py` | `GEMINI_HTTP_400`, `GEMINI_HTTP_429`, `GEMINI_JSON_INVALID` |
| `analyze.py` | `ANALYZE_NO_CUTS`, `ANALYZE_INVALID_PIPELINE` |
| `transcribe.py` | `TRANSCRIBE_WHISPER_FAILED` |
| `render.py` | `RENDER_FFMPEG_FAILED`, `RENDER_SOURCE_MISSING` |
| `subtitle.py` | `SUBTITLE_BURN_FAILED` |
| `publish.py` | `PUBLISH_YOUTUBE_DENIED` |

Handlers deixam de interpretar `str(e)` — só propagam `PipelineError`.
