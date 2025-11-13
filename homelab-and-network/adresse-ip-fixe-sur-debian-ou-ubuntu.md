# Adresse IP Fixe sur Debian ou Ubuntu

## 1- Avec network/interfaces

On édite le fichier interfaces

```
nano /etc/network/interfaces
```

Il doit ressembler à ça:

```
source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug ens18
iface ens18 inet static
        address 192.168.0.6/24
        address 10.0.0.12/24
        gateway 192.168.0.254
```



## 2- Avec systemd-network

Dans /etc/systemd/network on créé un fichier:

```
nano /etc/systemd/network/lan0.network
```

```
[Match]
Name=enp0s5
MACAddress=<ADRESSE_MAC>

[Network]
DHCP=yes
Address=192.168.0.8/24
Address=10.0.0.15/24
Gateway=192.168.0.254
DNS=192.168.0.254
```

