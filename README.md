# Zabbix шаблон: llama.cpp server (llama API)

Мониторинг [llama.cpp](https://github.com/ggml-org/llama.cpp) HTTP-сервера (`llama-server`) через Zabbix 7.x.

## Эндпоинты

| Master | Эндпоинт | Что даёт |
|---|---|---|
| `llama.health.raw` | `/health` | `{"status":"ok"}` |
| `llama.props.raw` | `/props` | build_info, total_slots, is_sleeping, model_path |
| `llama.models.raw` | `/models` | модель, контекст, размер файла |

Модель в llama.cpp фиксируется при старте сервера — понятия loaded/unloaded нет,
`/models` это конфигурация сервера.

## ITEM'ы

- `llama.health.flag` — 1 если `status == ok`
- `llama.sleeping.flag` — режим сна (`is_sleeping`; JSON-boolean в JS приходит строкой `"false"` — сравнивается явно)
- `llama.slots.total` — параллельные слоты
- `llama.model.count` / `llama.model.list` / `llama.model.path` / `llama.model.size`
- `llama.build.info` — версия сборки
- `net.tcp.service[http,,{$LLAMA_PORT}]` — simple check порта

## LLD

Правило `llama.models.discovery` по `/models` — прототипы `llama.model.nparams[{#MODELID}]`
и `llama.model.size[{#MODELID}]`.

## Триггеры

| Событие | Sev | Условие |
|---|---|---|
| llama.cpp: порт {$LLAMA_PORT} недоступен | High | TCP закрыт |
| llama.cpp: /health отвечает, но status != ok | Warning | живой эндпоинт, ненормальный статус |
| llama.cpp: в памяти нет ни одной модели | Warning | `model.count=0` 10m при живом health |

## Макросы

| Макрос | По умолчанию |
|---|---|
| `{$LLAMA_PORT}` | `8080` |

## Установка

1. Import `template_llamacpp.yaml`.
2. Хост с Agent-interface (`{HOST.CONN}`), привязать шаблон.
