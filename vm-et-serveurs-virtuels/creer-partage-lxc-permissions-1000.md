# Creer partage LXC - Permissions 1000

## Objectif

Mettre en place un partage de dossiers fiable entre :

* l’hôte Proxmox (PVE)
* plusieurs conteneurs LXC
* des services Docker (Syncthing, Paperless, etc.)

en utilisant **un utilisateur et un groupe communs** nommés `partage`, avec :

* **UID = 1000**
* **GID = 1000**

Ce modèle évite :

* les `nobody:nogroup`
* les `chmod 777`
* les comportements incohérents entre conteneurs

***

## Principe général

* Tous les fichiers partagés appartiennent à `1000:1000`
* Chaque machine (PVE, LXC) possède un user + group `partage`
* Les conteneurs LXC sont **unprivileged**
* Les conteneurs LXC utilisent un **idmap cohérent**
* Docker utilise `PUID=1000` et `PGID=1000`

***

## 1. Configuration des conteneurs LXC (idmap)

Sur l’hôte Proxmox, dans `/etc/pve/lxc/<CTID>.conf` :

```ini
unprivileged: 1
lxc.idmap: u 0 100000 1000
lxc.idmap: g 0 100000 1000
lxc.idmap: u 1000 1000 1
lxc.idmap: g 1000 1000 1
lxc.idmap: u 1001 101001 64535
lxc.idmap: g 1001 101001 64535
```

Redémarrer le conteneur :

```bash
pct restart <CTID>
```

***

## 2. Création de l’utilisateur et du groupe `partage`

### 2.1 Sur l’hôte Proxmox (PVE)

```bash
groupadd -g 1000 partage
useradd  -u 1000 -g 1000 -m -d /home/partage -s /bin/bash partage
```

Vérification :

```bash
id partage
```

***

### 2.2 Dans chaque conteneur LXC

```bash
getent passwd partage || useradd -u 1000 -g 1000 partage
getent group  partage || groupadd -g 1000 partage
usermod -g 1000 partage
```

***

## 3. Création d’un dossier partagé sur l’hôte

```bash
mkdir -p /mnt/data/papers
chown -R 1000:1000 /mnt/data/papers
chmod -R 2775 /mnt/data/papers
```

***

## 4. Bind mount du dossier dans un conteneur LXC

```bash
pct set <CTID> -mp0 /mnt/data/papers,mp=/data/papers
pct restart <CTID>
```

Dans le conteneur :

```bash
ls -ln /data/papers
```

***

## 5. Vérification des droits

```bash
su - partage -c 'touch /data/papers/.test && rm /data/papers/.test'
```

***

## 6. Utilisation avec Docker

### Exemple Docker Compose

```yaml
environment:
  - PUID=1000
  - PGID=1000
volumes:
  - /data/papers:/data
```

Règles :

* Toujours utiliser des chemins absolus
* Les dossiers montés doivent exister avant `docker compose up`

***

## 7. Partage avec plusieurs LXC

Même dossier hôte, même UID/GID, même idmap.

```bash
pct set <CTID2> -mp0 /mnt/data/papers,mp=/data/papers
```

***

## 8. Dépannage

### Dossier vu en `nobody:nogroup`

* Vérifier `lxc.idmap`
* Vérifier UID/GID 1000

### Impossible de chown dans le conteneur

Corriger depuis l’hôte :

```bash
pct mount <CTID>
chown -R 1000:1000 /var/lib/lxc/<CTID>/rootfs/data/papers
pct unmount <CTID>
```

***

## 9. Bonnes pratiques

* Un seul UID technique (1000)
* Pas de conteneur privileged
* Pas de `chmod 777`
* Toujours tester avec l’utilisateur `partage`

***

## Résumé

Configuration stable, lisible et maintenable pour un homelab Proxmox multi-conteneurs.
