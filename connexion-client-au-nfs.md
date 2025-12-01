# Connexion CLIENT au NFS

## 1 - Configuration du serveur

### 1.1 - Autorisation de l'ip

Il faut d'abord autoriser l'adresse IP du client sur le serveur. On modifie \`/etc/exports\`

```
micro /etc/exports
```

On ajoute dedans les lignes correspondantes (exemple pour les ip locales):

```
/mnt/silo_secure 10.0.0.0/24(rw,sync,anonuid=65534,anongid=65534,no_subtree_check,no_root_squash)
/mnt/silo_secure 150.107.201.0/24(rw,sync,anonuid=65534,anongid=65534,no_subtree_check,no_root_squash)
```

(On peut purger et stopper les partages NFS avec:)

```
exportfs -ua
```

On applique:

```
exportfs -a
```

## 2 - Configuration du client

### 2.1 - Installation du client

```
apt update && apt install -y nfs-common
```

### 2.2 - Montage du dossier

On créé le dossier de montage:

```
mkdir /mnt/silo_nfs
```

On teste le montage:

```
mount -t nfs4 10.0.0.22:/mnt/silo_secure /mnt/silo_nfs/
```

On le supprime:

```
umount /mnt/silo_nfs/
```

### 2.3 - Montage au démarrage

On édite fstab:

```
micro /etc/fstab
```

Ajoutons la ligne suivante:

```
10.0.0.22:/mnt/silo_secure  /mnt/silo_nfs  nfs  defaults  0 0
```

On recharge la configuration:

```
sudo mount -a
```

```
systemctl daemon-reload
```
