# NOVA SERVICES : E-Shop API Gateway

Microservices FastAPI (données en mémoire), conteneurisés avec Docker.

| Service | Routes | Port |
|---|---|---|
| `users` | `/health`, `/users`, `/users/{id}` | 8000 |
| `orders` | `/health`, `/orders`, `/orders/user/{id}`, `/orders/{id}` | 8000 |

## Lancer

```bash
docker build -t eshop-users:local ./users
docker build -t eshop-orders:local ./orders
docker run --rm -p 8001:8000 eshop-users:local
docker run --rm -p 8002:8000 eshop-orders:local
```
