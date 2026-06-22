# Máquina de Cortes

Pipeline de automação para transformar **lives e vídeos longos** em múltiplos formatos de conteúdo: vídeo completo, cortes médios (10–20 min), Shorts/Reels e assets de publicação.

Projeto do ecossistema **Woragis / LitCode** — canal de LeetCode e DSA em português.

---

## Visão

Uma sessão de estudo ou live de 1–2 horas vira:

| Saída | Exemplo |
|-------|---------|
| 1 vídeo completo | Live inteira ou questão resolvida |
| N cortes longos | Blocos de 10–20 min por tópico |
| N Shorts/Reels | Trechos de 15–60 s com hook forte |
| Metadados | Títulos, descrições, capítulos, thumbnails |

O objetivo não é só economizar tempo de edição — é **reutilizar cada gravação** de forma sistemática.

---

## O que já existe (embrião)

No repositório [`canal/end-screen/`](../canal/end-screen/) há o primeiro estágio do pipeline:

```
Vídeo gravado
  → fade-out de voz (últimos 2s)
  → concatena 12s de outro estático (outro.png)
  → música (trecho específico de outro-song.mp3)
  → fade-in na imagem e na música
  → export (FFmpeg via append-outro_v1.py)
```

Isso prova o padrão: **Python orquestra, FFmpeg renderiza**. A máquina de cortes estende essa ideia para seleção de trechos, múltiplos formatos e metadados.

---

## Fluxos alvo

### 1. Vídeo longo (pós-live ou gravação)

```
Live / gravação
  → transcrição
  → detecção de capítulos (questões, tópicos)
  → timestamps + títulos
  → outro automático
  → thumbnail
  → descrição
  → upload YouTube
```

Exemplo de capítulos (live LeetCode 2h):

```txt
00:00 Introdução
05:23 Two Sum
21:10 Binary Search
47:30 DFS
01:15:42 Dynamic Programming
```

### 2. Cortes longos (10–20 min)

```
Live
  → IA / regras identificam blocos por assunto
  → JSON com start/end + título
  → FFmpeg extrai cada bloco
  → outro + metadados por corte
  → N vídeos publicáveis
```

### 3. Shorts / Reels / TikTok

```
Vídeo ou live
  → transcrição
  → detecção de momentos (hook, dica, erro comum)
  → corte 15–60s
  → legenda (burn-in ou SRT)
  → crop 9:16
  → export
```

---

## Arquitetura proposta

```text
Entrada (live.mp4 | video.mp4)
        ↓
    FFmpeg (probe, normalize)
        ↓
    Whisper (transcrição local ou API)
        ↓
    Análise de cortes
      ├─ IA do YouTube (timestamps sugeridos pós-upload)
      ├─ LLM + LangChain (seleção e títulos)
      └─ Regras/heurísticas (LeetCode)
        ↓
    JSON de cortes
        ↓
    Gerador de vídeos (FFmpeg)
      ├─ longos
      ├─ shorts (9:16)
      └─ outro / música / transições
        ↓
    Saída organizada por pasta
```

### Ferramentas

| Camada | Ferramenta | Papel |
|--------|------------|-------|
| Edição | **FFmpeg** | Cortes, resize, fade, concat, encode |
| Transcrição | **Whisper** | Texto + timestamps por frase |
| Orquestração | **Python** | Scripts, LangChain, CLI |
| Seleção de cortes | **LLM** (OpenAI / local) | Hooks, títulos, agrupamento |
| Input externo | **IA do YouTube** | Timestamps sugeridos após upload |
| Legendas | FFmpeg / Whisper / ASS | Burn-in para Shorts |

> **Nota sobre linguagem:** para pipelines baseados em FFmpeg, Python, Go e Rust têm desempenho equivalente na renderização — o gargalo é sempre o encode. Python é suficiente para orquestração; Go/Rust só valem se precisar de binário único ou serviço de alta concorrência.

---

## Integração com IA do YouTube

Fluxo híbrido (manual + automático):

```text
Live gravada
  → upload (público ou não listado)
  → transcrição do YouTube
  → pergunta à IA do YouTube (melhores momentos para Shorts / cortes)
  → JSON de timestamps
  → LangChain / script Python
  → FFmpeg gera os arquivos
```

Exemplo de prompt para a IA do YouTube:

```text
Analise esta live de programação.

Encontre:
1. Os 10 melhores momentos para Shorts (15–60s).
2. Os 5 melhores momentos para cortes de 5–15 minutos.
3. Trechos com dicas de entrevista, erros comuns e explicações de algoritmos.

Retorne timestamps exatos (start, end, title).
```

Exemplo de JSON consumido pelo pipeline:

```json
{
  "shorts": [
    {
      "start": "00:12:15",
      "end": "00:12:55",
      "title": "Erro que reprova candidatos"
    },
    {
      "start": "00:35:02",
      "end": "00:35:48",
      "title": "Binary Search em 40 segundos"
    }
  ],
  "long_cuts": [
    {
      "start": "00:05:23",
      "end": "00:21:10",
      "title": "Two Sum — solução completa"
    }
  ]
}
```

---

## Gatilhos para LeetCode / DSA

Além da IA genérica, regras específicas do nicho aumentam a qualidade dos cortes:

- "cai em entrevista"
- "Google / Amazon pergunta isso"
- "truque importante" / "pegadinha"
- "complexidade O(n)"
- "erro comum"
- mudança de questão (detectada por título na tela ou fala)
- momento "Accepted" no LeetCode

Esses gatilhos podem ser combinados com a transcrição Whisper + keywords + LLM.

---

## Base de dados própria (fase avançada)

Armazenar por vídeo/live:

- transcrição completa
- timestamps de cortes gerados
- título, descrição, tags
- métricas pós-publicação (views, CTR, retenção)

Com volume (100+ lives), dá para otimizar: quais hooks viram Short, quais questões performam melhor, duração ideal, etc.

---

## Roadmap sugerido

Ordem de implementação com maior retorno para o canal:

| # | Etapa | Status |
|---|--------|--------|
| 1 | Outro automático (imagem + música + fade) | ✅ [`canal/end-screen/`](../canal/end-screen/) |
| 2 | Thumbnails automáticas | ✅ [`canal/generate-thumbnails.py`](../canal/generate-thumbnails.py) |
| 3 | Descrições e capítulos automáticos | 🔲 |
| 4 | Transcrição + JSON de cortes (Whisper + LLM) | 🔲 |
| 5 | Gerador de Shorts (9:16 + legenda) | 🔲 |
| 6 | Gerador de cortes longos | 🔲 |
| 7 | Integração IA YouTube → LangChain | 🔲 |
| 8 | Postagem / fila de publicação | 🔲 |

---

## Estrutura de pastas (planejada)

```text
maquina-de-cortes/
├── README.md                 # este arquivo
├── config/
│   └── pipeline.json         # durações, paths, volumes, gatilhos
├── input/                    # vídeos brutos
├── output/
│   ├── full/                 # vídeo completo com outro
│   ├── long/                 # cortes 10–20 min
│   └── shorts/               # 9:16
├── transcripts/              # Whisper .json / .srt
├── cuts/                     # JSON de cortes aprovados
└── scripts/                  # Python + chamadas FFmpeg
```

---

## Princípios de design

1. **FFmpeg faz o pesado** — scripts só montam filter graphs e paths.
2. **JSON como contrato** — cortes, metadados e configs em arquivos versionáveis.
3. **Human-in-the-loop** — revisar cortes antes de publicar (especialmente Shorts).
4. **yuv420p + A/V sync** — evitar pixel formats exóticos e dessincronia (lição do outro).
5. **Reutilizar assets do canal** — `outro.png`, `character/clean.png`, thumbnails, overlay.

---

## Referências no monorepo

| Path | Descrição |
|------|-----------|
| [`canal/end-screen/`](../canal/end-screen/) | Outro 12s, música, fades |
| [`canal/generate-thumbnails.py`](../canal/generate-thumbnails.py) | Thumbnails LeetCode |
| [`canal/overlay/`](../canal/overlay/) | Overlays OBS (config.js) |
| [`canal/character/`](../canal/character/) | Personagem aprovado para artes |

---

## Próximo passo

Definir o **formato do JSON de cortes** e o primeiro script: `extract_clip.py` — recebe `video.mp4` + `{ start, end, title }` e exporta um corte com fade opcional.

Depois: batch a partir de um `cuts.json` gerado por Whisper + LLM ou pela IA do YouTube.
