# Runtime Service (Elixir/OTP)

**Thin WebSocket coordinator** для roguelike игры.

## Ответственность

✅ **ЧТО ДЕЛАЕТ:**
- WebSocket server (Phoenix Channels)
- Action routing через YAML конфигурацию
- Session state management (ETS)
- State broadcast to clients
- Event Bus subscriber для NPC actions

❌ **ЧТО НЕ ДЕЛАЕТ:**
- AI логика
- Spawn конфигурация
- Level generation
- Бизнес-правила валидации
- Любая game logic

## Технологии

- **Язык:** Elixir 1.15+
- **Framework:** Phoenix Framework (WebSocket)
- **State:** ETS (in-memory)
- **Event Bus:** NATS JetStream

## Архитектура

```
Runtime
  ├── WebSocket Handler
  │   └── Handles player connections
  ├── Action Router
  │   ├── Reads config/action_routes.yaml
  │   └── Routes to gRPC services
  ├── Session Manager (GenServer)
  │   ├── Manages game sessions
  │   └── Stores state in ETS
  └── Event Subscriber
      └── Listens to NATS for NPC actions
```

## Размер кода

**Target:** ~2000 LOC (никогда не растет!)

## Запуск локально

```bash
# Install dependencies
mix deps.get

# Start Phoenix server
mix phx.server

# Health check
curl http://localhost:4001/health
```

## Docker

```bash
# Build
docker build -t runtime:dev -f Dockerfile.dev .

# Run
docker run -p 4000:4000 -p 4001:4001 runtime:dev
```

## Environment Variables

```bash
PORT=4000
SECRET_KEY_BASE=<generate with mix phx.gen.secret>
JWT_SECRET=<your-jwt-secret>
NATS_URL=nats://localhost:4222
REDIS_URL=redis://localhost:6379
ACTION_ROUTES_CONFIG=/config/action_routes.yaml
```

## Endpoints

| Endpoint | Description |
|----------|-------------|
| `ws://localhost:4000/socket` | WebSocket для игроков |
| `GET /health` | Health check (port 4001) |
| `GET /metrics` | Prometheus metrics (port 4001) |

## Testing

```bash
mix test
```

## Принцип работы

1. **Player connects** → WebSocket established
2. **Player sends action** → `{ action: "move", to: {x, y} }`
3. **Runtime reads YAML** → `move` → `movement-service:50051`
4. **Runtime calls gRPC** → `MovementService.ValidateMove()`
5. **Service responds** → `{ status: "success", new_position: {...} }`
6. **Runtime updates ETS** → Session state updated
7. **Runtime broadcasts** → All players in session receive update

## Не меняется при добавлении механик!

Добавление новой механики (например, заклинаний):
- ✅ Создается новый `spell-service`
- ✅ Добавляется 3 строки в `action_routes.yaml`
- ❌ Runtime НЕ трогается!

---

**Статус:** Skeleton (требует имплементации)
**Следующие шаги:** Реализовать WebSocket handler, Action Router, Session Manager
