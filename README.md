# spring-boot-devops

Backend Spring Boot (Java 21, MySQL) déployé automatiquement par un pipeline Jenkins CI/CD.

## Pipeline
Git → Build & Test Maven → Docker Build → Docker Push → Déploiement MySQL → Déploiement backend-app → Vérification

## Conteneurs
| Nom | Image | Port |
|---|---|---|
| mysql | mysql:8.0 | interne (réseau devops-net) |
| backend-app | adembt/backend-app:latest (registre privé) | 8090 → 8080 |

## Vérification
    docker ps
    docker logs backend-app
    curl http://localhost:8090/entreprise/all
