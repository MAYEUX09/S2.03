# 🛡️ SAE 2.03 - Forge Logicielle Sécurisée & Supervision

Ce projet déploie une infrastructure conteneurisée sécurisée comprenant une forge logicielle (Gitea), sa base de données (PostgreSQL) et un outil de supervision (Uptime Kuma). L'architecture repose sur une séparation stricte des réseaux (frontend/backend) pour isoler la base de données de l'extérieur.

---

## 🏗️ Architecture et Services

L'infrastructure est composée de 3 services principaux gérés par Docker Compose :

| Service | Image | Description | Ports exposés |
| :--- | :--- | :--- | :--- |
| **Gitea** | `gitea/gitea` | Forge logicielle (Git). Connectée à la base de données et accessible par les utilisateurs. | `3000` (HTTP)<br>`2222` (SSH) |
| **PostgreSQL** | `postgres:16` | Base de données pour Gitea. Totalement isolée de l'extérieur. | *Aucun* |
| **Uptime Kuma** | `louislam/uptime-kuma` | Outil de supervision. Surveille l'état de santé de Gitea. | `3001` (HTTP) |

---

## 🌐 Topologie Réseau

Pour des raisons de sécurité, l'environnement utilise deux réseaux Docker distincts :

1. **`net-backend` (192.168.10.0/24)** : Réseau privé.
   * **PostgreSQL** (`192.168.10.10`) : Accessible uniquement sur ce réseau.
   * **Gitea** (`192.168.10.20`) : Communique avec la base de données via ce réseau.

2. **`net-frontend` (192.168.20.0/24)** : Réseau public/supervision.
   * **Gitea** (`192.168.20.10`) : Expose son interface web et SSH.
   * **Uptime Kuma** (`192.168.20.20`) : Surveille Gitea (via `http://gitea:3000`) et expose son propre tableau de bord. Uptime Kuma n'a aucun accès au `net-backend`.

---

## 💾 Persistance des Données

Trois volumes Docker nommés sont configurés pour garantir que les données ne soient pas perdues lors du redémarrage ou de la suppression des conteneurs :
* `postgres_data` : Stockage des bases de données de Gitea.
* `gitea_data` : Stockage des dépôts Git, de la configuration et des clés SSH.
* `uptime_kuma_data` : Stockage de la configuration et de l'historique de supervision.

---

## 🚀 Installation et Utilisation

### Prérequis
* [Docker](https://docs.docker.com/get-docker/) installé.
* [Docker Compose](https://docs.docker.com/compose/install/) installé.

### Démarrer l'infrastructure
Placez-vous dans le répertoire contenant le fichier `docker-compose.yml` et lancez la commande suivante en arrière-plan :
```
docker compose up -d
```
Accès aux interfaces

Une fois les conteneurs démarrés, vous pouvez accéder aux services via votre navigateur :

    🦊 Interface Gitea : http://localhost:3000

    📈 Tableau de bord Uptime Kuma : http://localhost:3001

Note : Lors du premier lancement de Gitea, une page de configuration initiale s'affichera. Les identifiants de la base de données sont déjà préconfigurés via les variables d'environnement.
Commandes utiles

Voir les logs des services :
Bash
```
docker compose logs -f
```
Arrêter les services (sans supprimer les données) :
Bash
```
docker compose down
```
Arrêter les services ET supprimer les données (⚠️ Attention) :
Bash
```
docker compose down -v
```
🔒 Configuration de la supervision (Uptime Kuma)

Pour configurer la surveillance de Gitea dans Uptime Kuma :
``
Allez sur `http://localhost:3001` et créez votre compte administrateur.
``
Ajoutez un nouveau moniteur de type HTTP(s).

Définissez l'URL à surveiller sur http://gitea:3000 (grâce à la résolution DNS interne de Docker sur le réseau net-frontend).
