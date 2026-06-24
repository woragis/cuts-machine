# Plano — tratamento de erros e debugging (cuts-machine)

## Problema

Falhas de pipeline aparecem como texto plano (`gemini http 400 model=...`) sem indicar **qual job**, **handler**, **step** ou **cut** falhou. API e worker usam formatos diferentes; o frontend só mostra `run.errorMessage`.

## Formato alvo (`error_context`)

```json
{
  "code": "GEMINI_INVALID_REQUEST",
  "message": "Request contains an invalid argument.",
  "layer": "worker",
  "jobType": "analyze.gemini.url",
  "handler": "handle_analyze",
  "step": "analyze",
  "operation": "gemini.generate_json",
  "cutId": null,
  "cutType": null,
  "runId": "7b63da77-...",
  "jobId": "abc-...",
  "detail": "gemini http 400 model=gemini-3.5-flash: ..."
}
```

`error_message` (legado) continua com resumo legível para logs e buscas:

```text
[analyze.gemini.url] step=analyze handler=handle_analyze op=gemini.generate_json | gemini http 400 ...
```

## Fases

| Fase | Escopo | Status |
|------|--------|--------|
| [F1](./ERROR-HANDLING-PHASE-1.md) | Worker dispatch + DB `error_context` + API + UI básica | **Implementada** |
| [F2](./ERROR-HANDLING-PHASE-2.md) | `PipelineError` nos módulos de domínio (ingest, render, gemini…) | Planejada |
| [F3](./ERROR-HANDLING-PHASE-3.md) | `jobs_log` no worker ao enfileirar; retry inteligente na API | Planejada |
| [F4](./ERROR-HANDLING-PHASE-4.md) | Go apperrors com `cause`; correções por step (ex. Gemini YouTube URL) | Planejada |
| [F5](./ERROR-HANDLING-PHASE-5.md) | Observabilidade: correlação run/job nos logs; polling de jobs no UI | Planejada |

## Mapa job → step

| `job_type` | Handler | Step | Operation |
|------------|---------|------|-----------|
| `ingest.youtube.download` | `handle_ingest_youtube` | ingest | yt_dlp.download |
| `analyze.gemini.url` | `handle_analyze` | analyze | gemini.generate_json |
| `analyze.transcript` | `handle_analyze_transcript` | analyze | gemini.generate_json |
| `transcribe.run` | `handle_transcribe` | transcribe | whisper.transcribe |
| `metadata.generate` | `handle_metadata` | metadata | gemini.generate_json |
| `thumbnail.generate` | `handle_thumbnail` | thumbnail | pillow.render |
| `subtitle.generate` | `handle_subtitle` | subtitle | whisper_words.burn |
| `render.short` / `render.long` | `handle_render` | render | ffmpeg.cut |
| `outro.append` | `handle_outro_append` | outro | ffmpeg.append |
| `publish.youtube` | `handle_publish_youtube` | publish | youtube.upload |

## Referência rápida — onde olhar

| Camada | Arquivo |
|--------|---------|
| Normalização worker | `backend/worker/cuts_worker/pipeline_errors.py` |
| Persistência | `backend/worker/cuts_worker/db.py`, migration `007_error_context.sql` |
| Dispatch | `backend/worker/cuts_worker/handlers.py` → `dispatch()` |
| API models | `backend/server/internal/models/run.go` |
| API HTTP | `backend/server/internal/httpserver/{runs.go,ops.go}` |
| UI | `frontend/src/lib/pipelineError.ts`, `RunDetailView`, `RunJobsPanel` |
