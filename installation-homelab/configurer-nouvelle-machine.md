# Configurer Machine

## 0 - Création de la machine

Créer un VM avec au moins 50 Gb de Stockage. Sous Ubuntu ou Debian.
Lui configurer une adresse IP statique (DHCP)

## 1 - Mise à jour et config SSH

La première étape est de mettre à jour notre machine:
```
apt update && apt full-upgrade
```

On peut ensuite activer le ssh, en modifiant ce fichier:
```
nano /etc/ssh/sshd_config
```


On y décommente la ligne `PermitRootLogin prohibit-password`.
Si on souhaite authoriser la connexion en root avec le mot de passe on change `prohibit-password` par `yes`. Sinon on passe à l'étape suivante.



Sur notre client, on récupère la clé ssh:
```
cat .ssh/id_ed25519.pub
```

et on l'ajoute au fichier `.ssh/authorized_keys` sur notre machine.