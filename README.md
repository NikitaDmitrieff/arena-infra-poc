# Arena Infra POC

Orchestration Docker Compose qui démarre l'API FastAPI et le front statique.

## Prérequis
- Docker et Docker Compose installés (Docker Desktop ou Docker Engine via <https://docs.docker.com/get-docker/>).
- Cloner les dépôts `arena-api-poc` et `arena-front-poc` au même niveau que ce dépôt.

```
parent/
├─ arena-api-poc/
├─ arena-front-poc/
└─ arena-infra-poc/
```

## Démarrage local
```bash
cd arena-infra-poc
docker compose up -d --build
```
L'API répond sur <http://localhost:8000/hello> et le front sur <http://localhost:8080>.

## CI/CD et déploiement
Consulte `docs/ci-cd.md` pour le pipeline GitHub Actions qui orchestre build et déploiement sur la VM.
