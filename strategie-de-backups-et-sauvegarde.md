# Stratégie de backups et sauvegarde

J'ai actuellement plusieurs données importantes à sauvegarder et garder:

1. Mes documents (paperless) -> TRES IMPORTANT (redondant au maximum)
2. Mes photos (immich) -> plutot important
3. Dossier drive: un dossier drive synchronisé avec syncthing entre mes appareils (mac et clients) et mon serveur
4. Dossier depot: un dossier sur mon serveur sur lequel je ne fait que déposer des documents via syncthing avec mon mac. (Les documents sont ensuite absorbés et gardés uniquement sur le serveur)
5. Mes documents de cours : Je prends les notes de mes cours sur Obsidian, je le synchronise avec le cloud de l'université qui nous est offert.



## 1 - Stockage actuell



J'ai actuellement 4 endroits ou je stocke des données

1 - Mon Mac, les données que j'emporte quotidiennement: documents, dossier drive et documents de cours y sont stockées

2 - SILO, un serveur VPS de 1To: pas grand chose pour le moment, un serveur un peu fourre tout avec des données peu importantes et peu sensibles

3 - Mon Proxmox: 500Gb de stockage dit "important". Des dossiers partagés avec mes LXC papers, immich et sync/backup. Y sont stockées mes documents, mes photos et mon drive synchronisé avec syncthing

4 - Backup externe, 700Gb. Une backup chez un hébergeur en suisse. Je souhaite y enregistrer mon stockage important de manière chiffrée à intervalle régulier.



## 2 - Paperless - Mes documents

### 2.1 - Export périodique de paperless - LXC papers

Mes données paperless sont enregistrés dans le dossier /data/papers:

<figure><img src=".gitbook/assets/Capture d’écran 2026-02-04 à 22.56.51.png" alt=""><figcaption></figcaption></figure>

Je créé donc y script pour exporter tous mes documents dans export:

{% code title="/usr/local/bin/backup_paperless.sh" lineNumbers="true" %}
```bash
#!/bin/bash

cd /opt/stacks/paper
docker compose exec -T webserver document_exporter ../export
```
{% endcode %}

Je rends ce fichier exécutable:

```bash
chmox +x /usr/local/bin/backup_paperless.sh
```



Puis, on l'exécute tous les matins à 3h du matin:

```bash
crontab -e
```

Et on y ajoute ceci:

```bash
0 3 * * * /usr/local/bin/backup_paperless.sh >> /var/log/backup_paperless.log 2>&1
```



### 2.2 - Backup des documents - LXC sync

J'utilise borg pour mes backups:

```bash
apt update && apt install -y rsync borgbackup
```



On créé un utilisateur dédié et on lui ajoute la clé ssh de connexion:

```bash
adduser --system --group --home /home/sync --shell /bin/bash sync

mkdir -p /home/sync/.ssh
cp /root/.ssh/id_rsa* /home/sync/.ssh/
chown -R sync:sync /home/sync/.ssh
chmod 600 /home/sync/.ssh/id_rsa

su sync
eval "$(ssh-agent -s)"
ssh-add .ssh/id_rsa
```



On créé le repo de sauvegarde:

```bash
borg init --encryption=repokey --remote-path=borg12 USER@HOST:borg_paperless
```



On configure les permissions sur le dossier de paperless:

```bash
chown -R sync:sync /data/papers/export/
```



On créé le script sur `/home/sync/backup_paperless.sh`

{% code title="/home/sync/backup_paperless.sh" %}
```bash
#!/bin/bash
set -e

# 1. Sauvegarde Borg (chiffrée, versionnée)
borg create --stats --progress --remote-path=borg12 \
        --rsh "ssh -i /home/sync/.ssh/id_rsa" \
    USER@HOST:borg_paperless::paperless-{now} \
    /data/papers/export/

# 2. Nettoyage (7 jours de rétention)
borg prune --keep-daily 7 --remote-path=borg12 \
        --rsh "ssh -i /home/sync/.ssh/id_rsa" \
    USER@HOST:borg_paperless

# 3. Vérification de l'espace
ssh -i /home/sync/.ssh/id_rsa  USER@HOST quota
```
{% endcode %}



On le rend exécutable:

```bash
chown sync:sync /home/sync/backup_paperless.sh
chmod +x /home/sync/backup_paperless.sh
```



Puis on l'ajoute au crontab à 4h tous les matins:

```bash
crontab -u sync -e
```

Et on ajoute la ligne:

```bash
0 4 * * * /home/sync/backup_paperless.sh >> /home/sync/backup.log 2>&1
```



On ajoute de la sécurité en interdisant le ssh de l'utilisateur:

```bash
usermod -s /usr/sbin/nologin sync
```

