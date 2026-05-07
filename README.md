=========================================
STRUCTURE DE DOSSIERS RECOMMANDÉE
=========================================
_infra/
├── proxy/
│   └── compose.yml
└── tools/
    └── phpmyadmin/
        └── compose.yml

=========================================
1. FICHIER : _infra/proxy/compose.yml
=========================================
services:
  reverse-proxy:
    image: traefik:v3.6
    container_name: devel-reverse-proxy
    restart: unless-stopped
    environment:
      - DOCKER_API_VERSION=1.54
    command:
      - --api.insecure=true
      - --api.dashboard=true
      - --providers.docker=true
      - --providers.docker.exposedbydefault=false
      - --entrypoints.web.address=:80
    ports:
      - "80:80"
      - "8080:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    networks:
      - gateway

networks:
  gateway:
    external: true

=========================================
2. FICHIER : _infra/tools/phpmyadmin/compose.yml
=========================================
services:
  phpmyadmin:
    image: phpmyadmin:latest
    container_name: devel-phpmyadmin
    restart: unless-stopped
    environment:
      PMA_ARBITRARY: 1
      UPLOAD_LIMIT: 256M
    networks:
      - gateway
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.pma.rule=Host(`pma.localhost`)"
      - "traefik.http.routers.pma.entrypoints=web"
      - "traefik.http.services.pma.loadbalancer.server.port=80"
      - "traefik.docker.network=gateway"

networks:
  gateway:
    external: true

=========================================
COMMANDES DE LANCEMENT (RAPPEL)
=========================================
1. docker network create gateway
2. cd _infra/proxy && docker compose up -d
3. cd ../tools/phpmyadmin && docker compose up -d