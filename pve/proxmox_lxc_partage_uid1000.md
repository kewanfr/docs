# proxmox\_lxc\_partage\_uid1000

## Objectif

Mettre en place un partage de dossiers fiable entre :

* l’hôte Proxmox (PVE)
* plusieurs conteneurs LXC
* des services Docker (Syncthing, Paperless, etc.)

en utilisant un utilisateur et un groupe communs nommés `partage`, avec :

* UID = 1000
* GID = 1000

Ce modèle évite :

* les `nobody:nogroup`
* les `chmod 777`
* les comportements incohérents entre conteneurs

{% hint style="info" %}
Principe général :

* Tous les fichiers partagés appartiennent à `1000:1000`
* Chaque machine (PVE, LXC) possède un user + group `partage`
* Les conteneurs LXC sont unprivileged
* Les conteneurs LXC utilisent un idmap cohérent
* Docker utilise `PUID=1000` et `PGID=1000`
{% endhint %}

{% stepper %}
{% step %}
### Configuration des conteneurs LXC (idmap)

Sur l’hôte Proxmox, dans `/etc/pve/lxc/<CTID>.conf` :

{% code title="/etc/pve/lxc/<CTID>.conf" %}
```ini
unprivileged: 1
lxc.idmap: u 0 100000 1000
lxc.idmap: g 0 100000 1000
lxc.idmap: u 1000 1000 1
lxc.idmap: g 1000 1000 1
lxc.idmap: u 1001 101001 64535
lxc.idmap: g 1001 101001 64535
```
{% endcode %}

Redémarrer le conteneur :

{% code title="Commande" %}
```bash
pct restart <CTID>
```
{% endcode %}
{% endstep %}

{% step %}
### Création de l’utilisateur et du groupe partage — sur l’hôte Proxmox (PVE)

{% code title="Créer user/group sur PVE" %}
```bash
groupadd -g 1000 partage
useradd  -u 1000 -g 1000 -m -d /home/partage -s /bin/bash partage
```
{% endcode %}

Vérification :

{% code title="Vérifier" %}
```bash
id partage
```
{% endcode %}
{% endstep %}

{% step %}
### Création de l’utilisateur et du groupe partage — dans chaque conteneur LXC

{% code title="Créer/valider user/group dans le conteneur" %}
```bash
getent passwd partage || useradd -u 1000 -g 1000 partage
getent group  partage || groupadd -g 1000 partage
usermod -g 1000 partage
```
{% endcode %}
{% endstep %}

{% step %}
### Création d’un dossier partagé sur l’hôte

{% code title="Créer le dossier et définir propriétaires/permissions" %}
```bash
mkdir -p /mnt/data/papers
chown -R 1000:1000 /mnt/data/papers
chmod -R 2775 /mnt/data/papers
```
{% endcode %}
{% endstep %}

{% step %}
### Bind mount du dossier dans un conteneur LXC

Sur l’hôte Proxmox :

{% code title="Ajouter bind mount au conteneur" %}
```bash
pct set <CTID> -mp0 /mnt/data/papers,mp=/data/papers
pct restart <CTID>
```
{% endcode %}

Dans le conteneur, vérifier :

{% code title="Vérifier dans le conteneur" %}
```bash
ls -ln /data/papers
```
{% endcode %}
{% endstep %}

{% step %}
### Vérification des droits

Tester en tant qu’utilisateur `partage` dans le conteneur :

{% code title="Test toucher/suppression" %}
```bash
su - partage -c 'touch /data/papers/.test && rm /data/papers/.test'
```
{% endcode %}
{% endstep %}

{% step %}
### Utilisation avec Docker

Exemple d’extrait Docker Compose :

{% code title="docker-compose.yaml — extrait" %}
```yaml
environment:
  - PUID=1000
  - PGID=1000
volumes:
  - /data/papers:/data
```
{% endcode %}

Règles :

* Toujours utiliser des chemins absolus
* Les dossiers montés doivent exister avant `docker compose up`
{% endstep %}

{% step %}
### Partage avec plusieurs LXC

Même dossier hôte, même UID/GID, même idmap.

Ajouter le bind mount sur un autre conteneur :

{% code title="Ajouter bind mount pour un autre conteneur" %}
```bash
pct set <CTID2> -mp0 /mnt/data/papers,mp=/data/papers
```
{% endcode %}
{% endstep %}

{% step %}
### Dépannage

<details>

<summary>Dossier vu en nobody:nogroup</summary>

* Vérifier `lxc.idmap`
* Vérifier que l’utilisateur/groupe `partage` a bien UID/GID 1000 sur l’hôte et dans le conteneur

</details>

<details>

<summary>Impossible de chown dans le conteneur</summary>

Corriger depuis l’hôte :

{% code title="Corriger ownership depuis l" %}
```bash
pct mount <CTID>
chown -R 1000:1000 /var/lib/lxc/<CTID>/rootfs/data/papers
pct unmount <CTID>
```
{% endcode %}

</details>
{% endstep %}

{% step %}
### Bonnes pratiques

* Un seul UID technique (1000)
* Pas de conteneur privileged
* Pas de `chmod 777`
* Toujours tester avec l’utilisateur `partage`
{% endstep %}

{% step %}
### Résumé

Configuration stable, lisible et maintenable pour un homelab Proxmox multi-conteneurs en s’appuyant sur :

* un utilisateur/groupe `partage` en 1000:1000 partout
* des conteneurs LXC unprivileged avec idmap cohérent
* Docker configuré avec PUID/PGID = 1000
{% endstep %}
{% endstepper %}
