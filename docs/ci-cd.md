# CI/CD (GitHub Actions)

Ce pipeline part d'un dépôt d'infra contenant `docker-compose.yml`. L'objectif :
1. Synchroniser le fichier de composition sur la VM.
2. Lancer `docker compose up -d` pour redéployer les services après publication des images par les dépôts API et Front.

## Secrets requis
| Secret | Description |
| ------ | ----------- |
| `VM_HOST` | IP / DNS de la VM |
| `VM_USER` | Utilisateur SSH avec accès Docker |
| `VM_SSH_KEY` | Clé privée SSH |
| `VM_COMPOSE_DIR` | Dossier sur la VM où stocker `docker-compose.yml`, ex. `/opt/arena/compose` |

Optionnel : `VM_ENV_FILE` si tu veux copier un `.env`.

## Exemple de workflow `.github/workflows/deploy.yml`
```yaml
name: Deploy Compose

on:
  workflow_dispatch:
  push:
    branches: ["main"]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Copier le docker-compose sur la VM
        uses: appleboy/scp-action@v0.1.7
        with:
          host: ${{ secrets.VM_HOST }}
          username: ${{ secrets.VM_USER }}
          key: ${{ secrets.VM_SSH_KEY }}
          source: "docker-compose.yml"
          target: "${{ secrets.VM_COMPOSE_DIR }}"

      - name: Relancer les services
        uses: appleboy/ssh-action@v0.1.10
        with:
          host: ${{ secrets.VM_HOST }}
          username: ${{ secrets.VM_USER }}
          key: ${{ secrets.VM_SSH_KEY }}
          script: |
            cd ${{ secrets.VM_COMPOSE_DIR }}
            docker compose pull || true
            docker compose up -d --remove-orphans
```

## Déploiement initial
1. Sur la VM :
   ```bash
   mkdir -p $VM_COMPOSE_DIR
   cd $VM_COMPOSE_DIR
   # Copier manuellement le docker-compose.yml la première fois (scp ou via l'action ci-dessus)
   docker compose up -d
   ```
2. Vérifie que les images `arena-api-poc` et `arena-front-poc` existent (poussées depuis les autres dépôts).

## Déclenchement coordonné
- Configure les workflows des dépôts API et Front pour publier des images versionnées (`latest` + `SHA`).
- Dans ce dépôt, tu peux déclencher manuellement (`workflow_dispatch`) après publication, ou ajouter un webhook / détection d'image (ex. via `repository_dispatch`).

## Aller plus loin
- Ajouter un job qui attend la complétion des pipelines API/Front (via `repository_dispatch`).
- Gérer `.env`/secrets via `appleboy/scp-action` ou `scp` manuel.
- Intégrer Traefik/HTTPS en ajoutant les services au `docker-compose.yml`.
