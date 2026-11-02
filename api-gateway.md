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

## Étape 6 : configuration et secrets

- `.env` : `GATEWAY_TOKEN` généré avec `openssl rand -hex 32`, jamais commité.
- `.gitignore` et `.dockerignore` : excluent `.env`, caches Python et `.venv`.

## Étape 7 : Docker Compose

- Seule la Gateway publie un port : `127.0.0.1:3000 -> 8000`.
- Réseau `backend` interne (Users, Orders, Gateway) et réseau `frontend` (Gateway seule, pour le port publié).
- Durcissement commun : `read_only`, `cap_drop: ALL`, `no-new-privileges`.
- Healthchecks sur `/health` (Python, pas de curl dans l'image slim). La Gateway attend que Users et Orders soient healthy.
- `GATEWAY_TOKEN` lu depuis `.env`, le démarrage échoue s'il manque.

## Étape 8 : démarrage de l'architecture

```bash
docker compose up --build -d
docker compose ps
docker compose logs gateway
```

Résultat : les 3 services passent `healthy`, seule la Gateway expose `127.0.0.1:3000->8000/tcp`.

![Compose up](captures/etape8-01-compose-up.png)
![Services healthy et logs Gateway](captures/etape8-02-compose-healthy.png)

## Étape 9 : tests des endpoints

Jeton chargé avec `set -a; source .env; set +a`, puis `curl -i` sur `localhost:3000`.

| Requête | Attendu | Obtenu |
|---|---|---|
| `/users` sans jeton | 401 | 401 `Missing or invalid authentication token` |
| `/users` | 200 | 200 Alice, Bob |
| `/orders` | 200 | 200 commandes 101, 102, 103 |
| `/profile/1` | 200 | 200 Alice + commandes 101, 103 |
| `/profile/999` | 404 | 404 `User not found` |
| `/health` sans jeton | 200 | 200 `{"status":"ok"}` |

![source .env](captures/etape9-01-source-env.png)
![Sans jeton](captures/etape9-02-users-sans-jeton.png)
![Users](captures/etape9-03-users.png)
![Orders](captures/etape9-04-orders.png)
![Profile 1](captures/etape9-05-profile-1.png)
![Profile 999](captures/etape9-06-profile-999.png)
![Health](captures/etape9-07-health.png)

## Étape 10 : tests pytest

`tests/test_gateway.py` : 5 tests sur la Gateway isolée (appels backend simulés via `monkeypatch` de `fetch_json`).

- `/health` public, `/users` refusé sans jeton ou avec un mauvais jeton (401).
- `/profile/1` agrège Alice + commandes 101 et 103, `/profile/999` renvoie 404.

```bash
python -m pip install -r gateway/requirements.txt pytest
export GATEWAY_TOKEN=ci-test-token
python -m pytest -q
```

Résultat : `5 passed`.

![pytest](captures/etape10-01-pytest.png)
