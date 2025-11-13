# Mise en Place et configuration VPN Wireguard - SERVEUR Linux / Réseau HOMELAB

Inspiré du guide de it-connect.fr: [https://www.it-connect.fr/mise-en-place-de-wireguard-vpn-sur-debian-11/](https://www.it-connect.fr/mise-en-place-de-wireguard-vpn-sur-debian-11/)



## 0 - Le réseau souhaité

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Je souhaiter lier mon réseau local avec mes serveurs. Mes VPS sont sur 2 sous réseaux: 150.107.201.0/24 et 46.247.109.0/24.

Sur mon réseau local, le sous réseau est : 192.168.0.0/24. Celui du VPN de la box est 192.168.27.0/24.

Je souhaiter donner à tous mes serveurs une IP en 10.0.0.0/24. Je souhaite que tout le VPN puisse avoir accès à la fois au 192.168.0.0/24 (sous réseau de ma box), au sous réseau du VPN de la box: 192.168.27.64/27 et au sous réseau 10.0/24

## 1 - Préparation du serveur

### 1.1 - Installation de wireguard

```
sudo apt update -y && sudo apt install -y wireguard
```

### 1.2 - Génération de la clé / des clés

`wg genkey | tee privatekey | wg pubkey > publickey`

On garde ces 2 clés pour les étapes 1.3 et 2

### 1.3 - Création de la config

```
sudo nano /etc/wireguard/wg0.conf
```

Mettre ceci dans le fichier (en remplaçant la clé privée et clé publique)

```
[Interface]
Address = 10.0.0.15/32
SaveConfig = true
ListenPort = 51820
PrivateKey = <PRIVATE_KEY_SERVER>
```


## 2 - Génération du client

Sur le client, ouvrir wireguard puis créer une config vide. ![](<../.gitbook/assets/Capture d’écran 2025-11-07 à 14.02.32.jpg>)

On récupère la clé publique générée par celui-ci pour l'étape 3.1

La configuration doit ressembler à ça:

```
[Interface]
PrivateKey = <PRIVATE_KEY_CLIENT>
```

On your ajoute l'ip choisie pour le client (ou les 2 IP dans notre cas)

```
[Interface]
PrivateKey = <PRIVATE_KEY_CLIENT>
Address = 10.0.0.100/24
MTU = 1360

[Peer]
PublicKey = <PUBLIC_KEY_SERVER>
AllowedIPs = 192.168.0.0/24, 10.0.0.0/24, 192.168.27.0/24, 150.107.201.0/24
Endpoint = <SRV_IP>:51820

```

## 3 - Fin de la configuration coté serveur

### 3.1 Ajout du client

(Avant tout modification dans la config, bien couper l'interface avec sudo wg-quick down wg0)

On retourne dans la config:&#x20;

```
sudo nano /etc/wireguard/wg0.conf
```

On ajoute à la fin du fichier une entrée (ou plusieurs) \[Peer]

```
[Interface]
Address = 10.0.0.15/32
SaveConfig = true
ListenPort = 51820
PrivateKey = <PRIVATE_KEY_SERVER>

[Peer]
PublicKey = PUBLIC_KEY_CLIENT1
AllowedIPs = 10.0.0.100/32
```

### 3.2 Sécurité et ouverture de ports

#### 3.2.1 Firewall
On active le routage avec:&#x20;

```
sysctl -w net.ipv4.ip_forward=1
```


Puis on installe ufw:

```
sudo apt install ufw
```

On autorise le ssh pour pas être déconnecté:

```
sudo ufw allow 22/tcp
```

Ainsi que le port du VPN:

```
sudo ufw allow 51820/udp
```



Ensuite, on édite la configuration:

```
sudo nano /etc/ufw/before.rules
```

A la tout fin du fichier on rajoute:

```bash
# NAT - IP masquerade
*nat
:POSTROUTING ACCEPT [0:0]
-A POSTROUTING -o enp0s5 -j MASQUERADE

# End each table with the 'COMMIT' line or these rules won't be processed
COMMIT
```

Et on rajoute (après la section "_# ok icmp code for FORWARD_"):
{% code title="ufw/before.rules" lineNumbers="true" %}
```bash
# autoriser le forwarding pour le réseau distant de confiance (+ le réseau du VPN)
-A ufw-before-forward -s 192.168.0.0/24 -j ACCEPT
-A ufw-before-forward -d 192.168.0.0/24 -j ACCEPT

-A ufw-before-forward -s 192.168.27.64/27 -j ACCEPT
-A ufw-before-forward -d 192.168.27.64/27 -j ACCEPT

# Le réseau SRV de la maison
-A ufw-before-forward -s 10.0.0.0/24 -j ACCEPT
-A ufw-before-forward -d 10.0.0.0/24 -j ACCEPT

# Les serveurs chez HostHatch
-A ufw-before-forward -s 150.107.201.0/24 -j ACCEPT
-A ufw-before-forward -d 150.107.201.0/24 -j ACCEPT

# Les serveurs chez Datalix
-A ufw-before-forward -s 46.247.109.0/24 -j ACCEPT
-A ufw-before-forward -d 46.247.109.0/24 -j ACCEPT

# Optionnel: Tout le trafic est autorisé
-A ufw-before-forward -s 0.0.0.0/0 -j ACCEPT
-A ufw-before-forward -d 0.0.0.0/0 -j ACCEPT
```


On active et redémarre le firewall:

```
sudo ufw enable && sudo systemctl restart ufw
```


#### 3.2.2 Ouverture de ports

Si la machine est derrière un NAT, il faut ouvrir le port de celui-ci.

Faire pointer le port 51820 vers l'ip de notre machine

### 3.3 - Activation du service VPN

On peut enfin activer le vpn

```
sudo systemctl enable wg-quick@wg0.service
```

et on le lance

```
sudo systemctl start wg-quick@wg0.service
```



On peut regarder les logs avec:

```
sudo systemctl status wg-quick@wg0.service
```
