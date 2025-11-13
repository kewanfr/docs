# Configurer Machine

## 0 - Création de la machine

Créer un VM avec au moins 50 Gb de Stockage. Sous Ubuntu ou Debian.
Lui configurer une adresse IP statique (DHCP)

## 1 - Mise à jour et config SSH

La première étape est de mettre à jour notre machine:
```bash
apt update && apt full-upgrade
```

On peut ensuite activer le ssh, en modifiant ce fichier:
```bash
nano /etc/ssh/sshd_config
```


On y décommente la ligne `PermitRootLogin prohibit-password`.
Si on souhaite authoriser la connexion en root avec le mot de passe on change `prohibit-password` par `yes`. Sinon on passe à l'étape suivante.


Sur notre client, on récupère la clé ssh:
```bash
cat .ssh/id_ed25519.pub
```

et on l'ajoute au fichier `.ssh/authorized_keys` sur notre machine.

## 2 - Installation des outils et confs utiles

J'installe toujours quelques outils bien pratiques:

```bash
apt update -y && apt install -y curl sudo micro btop
```


Ensuite, je rajoute quelques alias dans mon bashrc:
```bash
nano .bashrc
```

```.bashrc
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'
alias hsc="history | grep"

alias wstop='systemctl stop wg-quick@wg0'
alias wstart='systemctl start wg-quick@wg0'
alias wstatus='systemctl status wg-quick@wg0'
alias wedit='micro /etc/wireguard/wg0.conf'
alias wshow='wg show wg0'


alias mm='micro'
alias start='systemctl start'
alias stop='systemctl stop'
alias status='systemctl status'
alias restart='systemctl restart'
alias reload='systemctl daemon-reload'
```