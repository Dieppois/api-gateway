# NOVA SERVICES : E-Shop API Gateway

Microservices FastAPI (données en mémoire), conteneurisés avec Docker.

| Service | Routes | Port |
|---|---|---|
| `users` | `/health`, `/users`, `/users/{id}` | 8000 |
| `orders` | `/health`, `/orders`, `/orders/user/{id}`, `/orders/{id}` | 8000 |
| `gateway` | `/health`, `/users`, `/orders`, `/profile/{id}` (Bearer requis sauf `/health`) | 3000 (hôte) |

## Lancer

```bash
cp .env.example .env              # puis remplacer la valeur par : openssl rand -hex 32
docker compose up --build -d      # Gateway sur http://localhost:3000
docker compose down               # arrêter
```

Seule la Gateway est exposée. Users et Orders restent sur le réseau Docker interne.
