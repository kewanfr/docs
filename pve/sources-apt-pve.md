# Sources APT PVE

Par défaut, les sources sont les sources entreprise, qui sont payantes. Il faut donc modifier les repo de APT pour utiliser les sources gratuites.



{% code title="/etc/apt/sources.list.d/ceph.sources" %}
```
Types: deb
URIs: http://download.proxmox.com/debian/ceph-squid
Suites: trixie
Components: no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
```
{% endcode %}

```
Types: deb
URIs: http://download.proxmox.com/debian/pve
Suites: trixie
Components: pve-no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
```
