# Mise en Place du SILO (NFS sécurisé)

Je souhaite mettre en place mon serveur nommé "silo", c'est à dire un serveur de stockage, accessible en NFS depuis mes autres serveurs. Ce tuto est la mise en place d'une partition chiffrée ainsi que de l'accès depuis les machines de mon réseau.

## 1 - Partition et volume

### 1 - Création de la partition

On commence par créer la table de partition sur l'entièreté du disque, ainsi qu'une partition unique.

```
# Créer une table GPT et une partition unique
sudo parted /dev/vdb --script \
  mklabel gpt \
  mkpart primary 0% 100%

# Le nouveau device sera /dev/vdb1
```



### 2 - Chiffrement de la partition



On chiffre une partie de la partition. On lui donne une passphrase:

```
sudo cryptsetup luksFormat /dev/vdb1 \
  --type luks2 \
  --cipher aes-xts-plain64 \
  --key-size 512 \
  --hash sha256 \
  --pbkdf argon2id

```

On ouvre la partition chiffrée:

```
sudo cryptsetup open /dev/vdb1 silo_crypt
# Il sera alors demandé la passphrase précisée plus tot
```

### 3 - Formatage du volume en ext4

On peut choisir un autre format mais le ext4 me semble le plus approprié

```
sudo mkfs.ext4 /dev/mapper/silo_crypt
```

### 4 - Montage et test

```
sudo mkdir -p /mnt/silo_secure
sudo mount /dev/mapper/silo_crypt /mnt/silo_secure

# Vérifiez la lecture/écriture :
touch /mnt/silo_secure/test && ls /mnt/silo_secure
```





### 5 - Ajout de la passphrase dans une clé

On créé la clé avec la commande

```
/usr/bin/dd if=/dev/urandom of=/root/silo.key bs=4096 count=1
```

On donne les droits d'accès au fichier:

```
sudo chmod 0400 /root/silo.key
```

Ajout de la clé dans le fichier .key (la passphrase sera demandée):

```
sudo cryptsetup luksAddKey /dev/vdb1 /root/silo.key
```



### 6 - Automatiser le montage au démarrage

```
sudo blkid /dev/vdb1 | grep UUID
```

On récupère ainsi l'UUID du volume

On l'utilise et on ajoute la ligne suivante dans `/etc/fstab`

```
silo_crypt UUID=abcd-1234-... /root/silo.key luks,discard
```

Cela décrypte le volume et le mappe au démarrage



Puis, on monte le volume en ajoutant la ligne suivante dans `/etc/fstab`&#x20;

```
/dev/mapper/silo_crypt  /mnt/silo_secure  ext4  defaults  0 2
```



Enfin, on met à jour `initramfs`

`sudo update-initramfs -u`





## 2 -&#x20;
