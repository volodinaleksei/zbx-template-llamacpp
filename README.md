# Zabbix template: llama.cpp server (llama API)

Monitoring for [llama.cpp](https://github.com/ggml-org/llama.cpp) HTTP server (`llama-server`) on Zabbix 7.x.

## Endpoints

| Master item | Endpoint | Provides |
|---|---|---|
| `llama.health.raw` | `/health` | `{"status":"ok"}` |
| `llama.props.raw` | `/props` | build_info, total_slots, is_sleeping, model_path |
| `llama.models.raw` | `/models` | model, context size, file size |

The model in llama.cpp is fixed at server startup — there is no loaded/unloaded notion,
`/models` just reflects server configuration.

## Items

- `llama.health.flag` — 1 if `status == ok`
- `llama.sleeping.flag` — sleep mode (`is_sleeping`; note: a JSON boolean reaches a JS step
  as the string `"false"`, so it must be compared explicitly)
- `llama.slots.total` — parallel slots
- `llama.model.count` / `llama.model.list` / `llama.model.path` / `llama.model.size`
- `llama.build.info` — build version
- `net.tcp.service[http,,{$LLAMA_PORT}]` — simple port check

## LLD

`llama.models.discovery` (from `/models`) — prototypes `llama.model.nparams[{#MODELID}]`
and `llama.model.size[{#MODELID}]`.

## Triggers

| Event | Severity | Condition |
|---|---|---|
| llama.cpp: port {$LLAMA_PORT} unreachable | High | TCP port closed |
| llama.cpp: /health responds but status != ok | Warning | endpoint alive, abnormal status |
| llama.cpp: no model in memory | Warning | `model.count=0` for 10m while health is alive |

## Macros

| Macro | Default |
|---|---|
| `{$LLAMA_PORT}` | `8080` |

## Setup

1. Import `template_llamacpp.yaml`.
2. Host with an Agent interface (`{HOST.CONN}`), link the template.
