---
mode: agent
---
Prépare pour un Raspberry Pi 5 (Raspberry Pi OS 64 bits) une configuration pour auto-héberger et déployer automatiquement des apps Docker depuis GitHub.

Objectif :
- À chaque push sur main : build image Docker, push dans un registry privé sur le Pi, redéployer via Portainer.

Contraintes :
- Portainer + Portainer Agent pour gérer les containers
- Registry privé basé sur registry:2
- Déploiement via webhook GitHub Actions
- Support Docker Compose

Livrables :
1. setup pour installer Docker, Docker Compose, Portainer, registry privé, config pare-feu
2. docker-compose.yml pour Portainer + registry
3. .github/workflows/deploy.yml : build pi-grafana, push vers registry privé, déclenchement webhook Portainer
4. Instructions pour HTTPS (Let's Encrypt via Caddy ou Traefik), config DNS/IP, auth basique sur registry