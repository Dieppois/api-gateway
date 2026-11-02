# NOVA SERVICES: E-Shop API Gateway

FastAPI microservices (in-memory data), containerized with Docker.

| Service | Routes | Port |
|---|---|---|
| `users` | `/health`, `/users`, `/users/{id}` | 8000 |
| `orders` | `/health`, `/orders`, `/orders/user/{id}`, `/orders/{id}` | 8000 |
| `gateway` | `/health`, `/users`, `/orders`, `/profile/{id}` (Bearer required except `/health`) | 3000 (host) |

## Run

```bash
cp .env.example .env              # then replace the value with: openssl rand -hex 32
docker compose up --build -d      # Gateway on http://localhost:3000
docker compose down               # stop
```

Only the Gateway is exposed. Users and Orders stay on the internal Docker network.

## Tests

```bash
python -m pip install -r gateway/requirements.txt pytest
GATEWAY_TOKEN=ci-test-token python -m pytest -q
```
