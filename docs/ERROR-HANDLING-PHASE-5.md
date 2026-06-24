# Fase 5 — Observabilidade

## Objetivo

1. Logs estruturados JSON no worker (`LOG.info("job_done", extra=error_context)`).
2. `consumer.py` inclui `job_type`, `run_id`, `job_id` em cada linha de erro.
3. Frontend: link "Ver job que falhou" entre banner do run e item no painel Jobs.
4. Opcional: exportar histórico de falhas por pipeline para debugging (`GET /v1/runs/{id}/debug`).
