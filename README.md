 # 🚀 Infrastructure de Développement Locale (_infra)
 
 Ce dépôt contient la configuration centralisée du **Reverse Proxy (Traefik)** et des outils partagés pour le développement local sous Docker / WSL2.
 
 ## 🛠️ Architecture
 
 L'infrastructure repose sur un réseau Docker externe nommé `gateway`. Tous les projets souhaitant être accessibles via une URL personnalisée (ex: `*.localhost`) doivent être connectés à ce réseau.
 
 * **Proxy :** Traefik v3.6 (Dashboard sur port `8080`)
 * **Outils :** phpMyAdmin (accessible sur `pma.localhost`)
 * **Réseau :** `gateway` (bridge externe)
 * **OS cible :** Linux Ubuntu sous **WSL2** (Windows Subsystem for Linux)
 
 ---
 
 ## 🚦 Pré-requis
 
 1. **Docker & Docker Compose** installés à l'intérieur de ton instance Ubuntu WSL2.
 2. **Réseau Gateway :** Doit être créé manuellement une première fois :
    ```bash
    docker network create gateway
    ```
 3. **Fichier Hosts (Windows) :** Ajoutez cette ligne dans `C:\Windows\System32\drivers\etc\hosts` pour que Windows route l'URL vers WSL :
    ```text
    127.0.0.1 pma.localhost
    ```
 
 ---
 
 ## 🐧 Automatisation sous WSL2 Ubuntu (Productivité)
 
 Pour t'éviter de naviguer manuellement dans les dossiers de l'infra à chaque démarrage de ton PC, tu peux installer des raccourcis globaux (fonctions et alias) dans ton terminal Ubuntu.
 
 ### Installation :
 1. Ouvre le fichier de configuration de ton terminal (généralement `~/.zshrc` si tu utilises Zsh, ou `~/.bashrc` si tu es sous Bash) :
    ```bash
    nano ~/.zshrc
    ```
 2. Colle le bloc de scripts suivant tout en bas du fichier :
 
 ```bash
 # 🛰️ Gestion de l'infrastructure globale de dev
 infra-up() {
     cd ~/Projets/_infra/proxy || return
     docker compose up -d
 
     cd ~/Projets/_infra/tools/phpmyadmin || return
     docker compose up -d
 
     cd ~/Projets || return
     echo "🚀 Infrastructure (Proxy + PMA) démarrée avec succès !"
 }
 
 infra-down() {
     cd ~/Projets/_infra/tools/phpmyadmin || return
     docker compose down
 
     cd ~/Projets/_infra/proxy || return
     docker compose down
 
     cd ~/Projets || return
     echo "🛑 Infrastructure arrêtée."
 }
 
 infra-restart() {
     infra-down
     infra-up
 }
 
 # 🐳 Utilitaires Docker globaux
 alias dps="docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'"
 
 docker-stop-all() {
     local containers=$(docker ps -q)
     if [ -n "$containers" ]; then
         docker stop $containers
     else
         echo "💤 Aucun conteneur en cours d'exécution."
     fi
 }
 
 docker-clean() {
     docker system prune -f
     echo "🧹 Ménage effectué dans Docker."
 }
 ```
 3. Sauvegarde (`Ctrl+O`, `Entrée`, puis `Ctrl+X`) et recharge ton terminal :
    ```bash
    source ~/.zshrc
    ```
 
 ---
 
 ## 💡 À quoi ça sert ? Cas d'utilisation au quotidien
 
 | Commande | Cas d'utilisation concret | Ce que ça fait en arrière-plan |
 | :--- | :--- | :--- |
 | `infra-up` | **Le matin en commençant à bosser.** Tu lances cette commande depuis n'importe où pour démarrer le proxy Traefik et phpMyAdmin d'un coup. | Va dans `_infra`, démarre les conteneurs en tâche de fond, puis te remet dans ton dossier `~/Projets`. |
 | `infra-down` | **Le soir en coupant ton PC.** Permet de libérer les ports `80` et `8080` de ta machine proprement. | Éteint proprement Traefik et phpMyAdmin sans supprimer tes données de BDD. |
 | `infra-restart` | **En cas de bug réseau ou de coupure de courant.** Si le proxy Traefik ne répond plus ou que phpMyAdmin a perdu sa connexion. | Enchaîne un `infra-down` puis un `infra-up` pour réinitialiser le réseau de l'infra. |
 | `dps` | **Vérifier ce qui tourne actuellement.** Au lieu du tableau illisible de `docker ps`, tu as une liste ultra-propre. | Affiche uniquement les Noms, le Statut et les Ports ouverts des conteneurs actifs. |
 | `docker-stop-all` | **Changement de projet radical ou PC qui rame.** Tu veux éteindre absolument tous tes projets en cours pour repartir à zéro. | Détecte tous les conteneurs actifs sur ton Docker (même hors infra) et les coupe instantanément. |
 | `docker-clean` | **Gagner de l'espace disque (Espace saturé sous WSL2).** Docker accumule les caches de build et les conteneurs orphelins. | Purge définitivement les conteneurs arrêtés et les réseaux inutilisés pour libérer des Go. |
 
 ---
 
 ## 📥 Installation classique (Sans les alias)
 
 Si tu préfères l'approche manuelle :
 ```bash
 git clone git@github.com:Florimond-Jouffroy/_infra.git
 cd _infra/proxy && docker compose up -d
 cd ../tools/phpmyadmin && docker compose up -d
 ```
 
 ---
 
 ## 🔗 Accès aux services
 
 | Service | URL | Description |
 | :--- | :--- | :--- |
 | **Traefik Dashboard** | http://localhost:8080 | Tour de contrôle de tes routes |
 | **phpMyAdmin** | http://pma.localhost | Gestion de tes bases de données |
 
 ---
 
 ## 📝 Ajouter un projet (Exemple de configuration)
 
 Pour qu'un projet soit routé automatiquement par Traefik, utilise cette structure dans son `compose.yml` :
 
 ```yaml
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
 ```
 
 ---
 
 ## ⚠️ Troubleshooting (WSL2)
 
 **Problème de droits sur le socket Docker :**
 ```bash
 sudo chmod 666 /var/run/docker.sock
 ```
 
 **DNS interne Docker bloqué (Gateway Timeout) :**
 ```bash
 docker network prune -f
 docker network create gateway
 ```