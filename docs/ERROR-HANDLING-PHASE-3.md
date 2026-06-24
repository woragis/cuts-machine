# Fase 3 — Audit trail completo de jobs + retry inteligente

## Objetivo

1. Worker chama `insert_job_log()` em `queue_util._enqueue()` para jobs encadeados (metadata, thumb, render…).
2. API `Retry` usa `error_context.step` / último `job_type` failed em vez de default ingest.
3. UI: polling de jobs enquanto run ∈ `{analyzing, rendering, …}`; highlight do job failed.
