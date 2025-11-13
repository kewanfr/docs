# Mise en Place et configuration VPN Wireguard - CLIENT Linux

Afin de permettre à mes serveurs distants d'accéder à mon réseau local, je les ai connecté à mon serveur VPN Wireguard chez moi. Voici comment modifier les paramètres et redémarrer le service.



## 1. Installer wireguard

```
sudo apt update && sudo apt install -y wireguard  
```



## 2. Configurer Wireguard

1. **Créer le fichier de configuration Wireguard**\
   Créez le fichier de configuration pour l'interface Wireguard en éditant `/etc/wireguard/wg0.conf`.

Cela configurera votre VPN Wireguard et le service sera activé pour se lancer au démarrage du système. Assurez-vous de remplacer les champs de clé et d'adresse par vos informations spécifiques.

<pre class="language-yaml"><code class="lang-yaml"><strong>/etc/wireguard/wg0.conf
</strong>[Interface]
PrivateKey = .....
Address = 10.0.0.0/32
#DNS = __.__.__.__
#MTU = 1360

[Peer]
PublicKey = ......
Endpoint = _:_
AllowedIPs = 192.168.0.0/24, 10.0.0.0/24, 192.168.27.0/24, 150.107.201.0/24
</code></pre>


## 3. Activer et tester le VPN

On peut lancer le VPN avec la commande:

```
sudo wg-quick up wg0
```

On vérifie ensuite qu'il est lancé avec

```
sudo wg show wg0
```

On devrait obtenir un message avec la clé publique, le port de connexion et les informations de connexions.



## 4. Création et activation d'un service

On créé le service avec la commande enable, puis on le démarre

```
sudo systemctl enable wg-quick@wg0

sudo systemctl start wg-quick@wg0
```

Parfait, le serveur est connecté au réseau local avec wireguard !
