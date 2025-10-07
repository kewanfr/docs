# Augmenter la taille du disque VM PVE

1 - Augmenter la taille du disque sur PVE



Puis, utiliser toute la taille du disque pour la partition:



Trouver le nom de la partition:

```
lsblk -f
```

On trouve /dev/sda3

donc:&#x20;

```
sudo apt-get update && sudo apt-get install -y cloud-guest-utils lvm2
sudo growpart /dev/sda 3
sudo partprobe
sudo udevadm settle

# 2) Tell LVM the PV got bigger
sudo pvresize /dev/sda3

# 3) Extend the root LV to use all free space and resize ext4 online
sudo lvextend -r -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv

# 4) Verify
lsblk -f
df -h /

```

On vérifie avec:

```
df -h
```

