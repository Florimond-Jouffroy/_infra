# 🚀 Infrastructure de Développement Locale (_infra)

Ce dépôt contient la configuration centralisée du **Reverse Proxy** et des outils partagés pour le développement local sous Docker / WSL2.

## 🛠️ Architecture

L'infrastructure repose sur un réseau Docker externe nommé `gateway`. Tous les projets souhaitant être accessibles via une URL personnalisée (ex: `*.localhost`) doivent être connectés à ce réseau.

* **Proxy :** Traefik v3.6 (Dashboard sur port `8080`)
* **Outils :** phpMyAdmin (accessible sur `pma.localhost`)
* **Réseau :** `gateway` (bridge externe)
* **OS cible :** Linux / WSL2

---

## 🚦 Pré-requis

1. **Docker & Docker Compose** installés.
2. **Réseau Gateway :** Doit être créé manuellement une première fois :
   docker network create gateway
3. **Fichier Hosts (Windows) :** Ajoutez cette ligne dans `C:\Windows\System32\drivers\etc\hosts` :
   127.0.0.1 pma.localhost

---

## 📥 Installation et Lancement

### 1. Cloner le dépôt
git clone git@github.com:Florimond-Jouffroy/_infra.git
cd _infra

### 2. Démarrer le Proxy
cd proxy
docker compose up -d

### 3. Démarrer phpMyAdmin
cd ../tools/phpmyadmin
docker compose up -d

---

## 🔗 Accès aux services

| Service | URL | Description |
| :--- | :--- | :--- |
| **Traefik Dashboard** | http://localhost:8080 | Tour de contrôle |
| **phpMyAdmin** | http://pma.localhost | Gestion BDD |

---

## 📝 Ajouter un projet (Exemple)

Pour qu'un projet soit routé par Traefik, utilisez cette structure dans son `compose.yml` :

services:
  mon-app:
    image: my-image
    networks:
      - gateway
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.mon-app.rule=Host(`mon-app.localhost`)"
      - "traefik.http.routers.mon-app.entrypoints=web"
      - "traefik.http.services.mon-app.loadbalancer.server.port=80"
      - "traefik.docker.network=gateway"

networks:
  gateway:
    external: true

---

## ⚠️ Troubleshooting (WSL2)

**Problème de droits sur le socket Docker :**
sudo chmod 666 /var/run/docker.sock

**DNS interne Docker bloqué (Gateway Timeout) :**
docker network prune -f
docker network create gateway