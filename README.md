# Immich

Déploiement auto-hébergé de [Immich](https://immich.app) (photos et vidéos) avec Docker Compose, piloté par un seul script : `./immich`.

Le `docker-compose.yml` est celui publié par Immich, sans modification. Tout ce qui est propre à une machine (mots de passe, chemins, dossiers montés) vit dans des fichiers non versionnés.

## Prérequis

- Docker avec le plugin `docker compose` (v2)
- `curl` (ou `wget`)
- Le port `2283` libre

## Installation

```bash
git clone <ce dépôt> immich && cd immich
./immich install
```

La commande `install` :

1. télécharge le `.env` de la dernière release s'il n'existe pas encore, avec un `DB_PASSWORD` aléatoire et le fuseau horaire de la machine ;
2. crée les dossiers de données ;
3. tire les images et démarre la pile.

Ouvre ensuite `http://<machine>:2283` et crée le compte administrateur.

## Mettre à jour Immich

```bash
./immich update check     # compare sans rien modifier
./immich update           # met à jour (demande confirmation)
```

Une release Immich, ce sont des images **et** un `docker-compose.yml` (digests de Postgres et de Valkey, nouveaux services…). `update` aligne les deux :

1. il cherche la dernière release stable sur GitHub et la compare à la version déployée ;
2. il télécharge le `docker-compose.yml` de cette release et affiche les différences ;
3. il demande confirmation ;
4. il sauvegarde la base dans `backups/`, ainsi que l'ancien `docker-compose.yml` ;
5. il tire les images, recrée les conteneurs et vérifie la version déployée.

Variantes :

| Commande | Effet |
|---|---|
| `./immich update` | dernière release de la version majeure suivie par `IMMICH_VERSION` (`v3` → dernière 3.x) |
| `./immich update v3.3.1` | version précise, écrite dans `IMMICH_VERSION` |
| `./immich update --yes` | sans confirmation : pour ssh non interactif ou cron |

`update` ne passe jamais à une nouvelle version majeure de lui-même. Quand une v4 sort, il s'arrête et demande de la donner explicitement (`./immich update v4.0.0`), après lecture des [notes de version](https://github.com/immich-app/immich/releases).

**En cas d'échec :** remets l'ancien `docker-compose.yml` (dans `backups/`) et l'ancienne valeur d'`IMMICH_VERSION` dans `.env`, puis lance `./immich restore`.

## Commandes

| Commande | Effet |
|---|---|
| `./immich start` / `stop` / `restart [service]` | cycle de vie de la pile |
| `./immich status` | état des conteneurs, réponse de l'API, version |
| `./immich logs [service]` | suit les journaux |
| `./immich backup [N]` | sauvegarde la base dans `backups/` et garde les N dernières |
| `./immich restore [fichier]` | restaure une sauvegarde (la plus récente par défaut) |
| `./immich disk` | taille des médias, de la base et des sauvegardes |
| `./immich library add <dossier> [--rw]` | monte un dossier existant comme bibliothèque externe |
| `./immich library remove <dossier>` / `list` | retire un dossier monté / liste les dossiers montés |
| `./immich shell [service]` / `psql` | shell dans un conteneur / console SQL |
| `./immich admin <commande>` | `immich-admin` (`list-users`, `grant-admin`…) |
| `./immich reset-password` | réinitialise le mot de passe administrateur |
| `./immich remove [--data\|--images\|--all]` | supprime les conteneurs, et sur demande les données |

`./immich help` donne la liste complète.

## Sauvegardes

`./immich backup` sauvegarde **uniquement la base**. Les photos sont dans `library/` (ou `UPLOAD_LOCATION`) et doivent être copiées à part.

Pour une sauvegarde quotidienne qui garde les 7 dernières :

```cron
0 3 * * * /chemin/vers/immich/immich backup 7 >> /var/log/immich-backup.log 2>&1
```

## Bibliothèques externes

Pour indexer un dossier existant sans le copier :

```bash
./immich library add /media/disque/Photos
```

Le dossier est monté au même chemin dans le serveur, en lecture seule, via `docker-compose.override.yml`. Ce fichier est généré par le script : ne le modifie pas à la main. Il reste ensuite à déclarer la bibliothèque dans l'interface web (Administration → Bibliothèques externes). Le script affiche les étapes.

## Fichiers

| Fichier | Versionné | Rôle |
|---|---|---|
| `immich` | oui | script de pilotage |
| `docker-compose.yml` | oui | compose officiel de la release déployée |
| `.env` | **non** | configuration et mot de passe de la base |
| `.external-libraries`, `docker-compose.override.yml` | non | dossiers montés, propres à la machine |
| `library/`, `postgres/`, `backups/` | non | médias, base, sauvegardes |
