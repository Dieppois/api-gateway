# Journal du TP

## Étapes 1 et 2 : services Users et Orders

- `main.py` : API FastAPI, données en mémoire, 404 si ressource inconnue.
- `requirements.txt` : `fastapi`, `uvicorn`.
- `Dockerfile` : image `python:3.12-slim`, `pip --no-cache-dir`, utilisateur non-root `appuser`, pas de `.pyc`.

## Étape 3 : tests des services seuls

```bash
docker build -t eshop-users:local ./users
docker build -t eshop-orders:local ./orders
docker run --rm -p 8001:8000 eshop-users:local    # puis curl localhost:8001/users et /users/1
docker run --rm -p 8002:8000 eshop-orders:local   # puis curl localhost:8002/orders et /orders/user/1
```

Résultat : toutes les requêtes répondent 200 avec les données attendues.

![Build](captures/etape3-01-build-images.png)
![Logs Users](captures/etape3-02-users-logs.png)
![Réponses Users](captures/etape3-03-users-curl.png)
![Logs Orders](captures/etape3-04-orders-logs.png)
![Réponses Orders](captures/etape3-05-orders-curl.png)

## Étape 4 : responsabilités de la Gateway

| Route | Rôle | Auth |
|---|---|---|
| `GET /users` | Proxy vers Users | Bearer |
| `GET /orders` | Proxy vers Orders | Bearer |
| `GET /profile/{id}` | Agrégation BFF user + commandes | Bearer |
| `GET /health` | État de la Gateway | Public |

Protections : jeton Bearer en variable d'environnement, timeout vers les services, erreurs internes masquées (502), backends sur réseau Docker privé sans port publié.

## Étape 5 : code de la Gateway

- `main.py` : auth Bearer comparée en temps constant (`secrets.compare_digest`), appels `httpx` avec timeout 5 s, erreurs backend converties en 502, `/profile/{id}` appelle Users et Orders en parallèle (`asyncio.gather`) puis filtre.
- `requirements.txt` : `fastapi`, `uvicorn`, `httpx`.
- `Dockerfile` : identique aux services (slim, non-root).
