# 2nd NVME partagé avec les LXC

Je souhaiter utiliser un NVME spécifiquement pour certaines données. Je souhaiter donc le rajouter sur mon PVE en direct, et donner accès à certains dossier pour chacunes de mes LXC.



### Trouver et partitionner le disque

Je cherche les infos sur mon disque:

<pre class="language-bash"><code class="lang-bash"><strong>root@pve:~# lsblk -o NAME,SIZE,MODEL,SERIAL,TYPE,FSTYPE,MOUNTPOINT
</strong>
NAME                             SIZE MODEL                SERIAL       TYPE FSTYPE      MOUNTPOINT
nvme0n1                        465.8G WD Red SN700 500GB   25465W800702 disk             
</code></pre>

Mon disque est bien nvme0n1. Je le partitionne et le formate&#x20;

```bash
wipefs -a /dev/nvme0n1
sgdisk --zap-all /dev/nvme0n1
sgdisk -n 1:0:0 -t 1:8300 -c 1:"nvme-data" /dev/nvme0n1
mkfs.ext4 -L nvme-data /dev/nvme0n1p1
```

### Monter le disque

```bash
mkdir -p /mnt/nvme
```

Trouver l'UUID du disque:

```bash
blkid /dev/nvme0n1p1
```



Ajouter cette ligne (en remplaçant l'UUID) dans \`/etc/fstab\`

```bash
UUID=xxxx-xxxx  /mnt/data  ext4  defaults,noatime  0  2
```

Le monter:

```bash
mount -a
systemctl daemon-reload
df -h /mnt/data
```

## Lier les dossiers aux VMS

```bash
mkdir -p /mnt/data/ct-101
mkdir -p /mnt/data/ct-102
mkdir -p /mnt/data/shared
```



On vérifie que les LXC sont unprivileged

```bash
pct config 101 | grep unprivileged || true
```



```bash
chown -R 100000:100000 /mnt/data/ct-101
chown -R 100000:100000 /mnt/data/ct-102
chmod -R 770 /mnt/data/ct-101
chmod -R 770 /mnt/data/ct-102
```



```bash
chmod 755 /mnt/nvme
```



### Monter les dossiers

```bash
pct set 101 -mp0 /mnt/data/ct-101,mp=/data/ct
pct set 101 -mp1 /mnt/data/shared,mp=/data/shared
```



Pour un dossier read only:

```shellscript
pct set 150 -mp0 /mnt/data,mp=/data,ro=1
```



