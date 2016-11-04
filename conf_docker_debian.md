# Configurer Docker sur Debian8

| Nom     | Version | Modification                                |
| ------- | ------- | ------------------------------------------- |
| L. Arod | 0.01.00 | Création du document.                       |
| L. Arod | 0.01.01 | Mise à jour après une première présetation. |

## Introduction

Docker est un logiciel permettant de créer et de manager des container.
Les containers sont des processus lancer dans un environnement isolé du système
d'exploitation, mais utilisant le même kernel. Pour plus d'information sur
l'impacte d'un container sur les performances d'une application :

- recap des perfs : http://stackoverflow.com/a/26149994
- live coding d'un container : https://www.youtube.com/watch?v=HPuvDm8IC-4
- article par IBM : http://domino.research.ibm.com/library/cyberdig.nsf/papers/0929052195DD819C85257D2300681E7B/$File/rc25482.pdf

Nécessaire : Debian 8 (Linux 3.10), Windows 10 ou macOS Yosmite

## Configurer CNTLM

1. Télécharger CNTLM depuis internet : https://packages.debian.org/jessie/net/cntlm
2. Copier le paquet sur la VM et executer : `$ sudo dpkg -i cntlm_0.92.3-1_amd64.deb `
3. Editer le fichier de configuration :  `$ sudo vim /etc/cntlm.conf` , ci-dessous une liste des champs à modifier :

```
Username	arod
Domain		POSTAL
...
Proxy		proxybgx:8080
...
NoProxy		localhost, 127.0.0.*, 10.*, 192.168.*, 172.*
...
Gateway		yes
...
Allow		127.0.0.1
Allow		10.0.0.1
```

Il est ensuite nécessaire d'encrypter le mot de passe utilisateur.
Exécuter la commande suivante : `$ sudo cntlm -H`  pour obtenir un mot de passe
à recopier dans le fichier `/etc/cntlm.conf`.

Il est possible d'ajouter dans `/etc/cntlm.conf` la ligne suivante pour
s'approcher au plus de la config. proxy de Solystic :

```
NoProxy		intranet, vserv*, gserv*, mib.solystic.com, vkpro1, 172.16*, 159.217*, vpn2.solystic.com, ftp.solystic.com, vsus01, internet, intersite, *.postal.local, servicedesk.solystic.com, vtomprd, ftp.electrobeauce.com, venera, elearning.cegos.com, files.elearning.com, www3.gotomeeting.com, easy, easy.solystic.com, 192.168.7.13, partners.solystic.com, 10.*, ftp.exerion.net, gpac.postal.local, gpac.solystic.com, 172.17.*, 172.18.*, sourceforge.net, 46.21.191.34
```

## Configurer APT

La commande suivante configure `apt` pour utiliser le proxy `cntlm` :

```bash
$ sudo cat > /etc/apt/apt.conf.d/proxy <<EOT
Acquire::http::Proxy "http://127.0.0.1:3128";
Acquire::https::Proxy "http://127.0.0.1:3128";
EOT
```

## Installer open-vm-tools

La ligne suivante installe les outils nécessaires à VMWare pour redimensionner
l'écran :

```bash
$ sudo apt install open-vm-tools open-vm-tools-desktop
```

## Installer Docker

Les commandes suivantes sont extraites de https://docs.docker.com/engine/installation/linux/debian/
et permettent d'installer Docker sur une machine Debian 8 :

```bash
# Install docker-engine
$ sudo apt install apt-transport-https ca-certificates
$ sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
$ sudo cat >> /etc/apt/sources.list.d/docker.list <<EOT
deb https://apt.dockerproject.org/repo debian-jessie main
EOT
$ sudo apt update
$ sudo apt install docker-engine
```

Après avoir installé docker, il est nécessaire de donner des droits aux
utilisateurs non-root :

```bash
# Allow user to run docker
$ sudo gpasswd -a etudes docker
$ sudo systemctl restart docker
```

## Configurer le proxy Docker

Les commandes suivantes permettent de configurer `docker` pour utiliser le
proxy `cntlm` :

```bash
$ sudo mkdir /etc/systemd/system/docker.service.d
$ sudo cat > /etc/systemd/system/docker.service.d/http-proxy.conf <<EOT
[Service]
Environment="HTTP_PROXY=http://127.0.0.1:3128/" "HTTPS_PROXY=http://127.0.0.1:3128/"
EOT
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```

Une fois docker configuré, il est possible de tester l'environnement avec la
commande :

```bash
$ docker run hello-world
```

## Configurer Docker pour le réseau Solystic

Solystic utilise un réseau privé en `172.18.*` qui entre en conflit avec le réseau
privé de docker. Pour résoudre le prolème, il faut indiquer à docker le réseau
à utiliser :

```bash
$ sudo cat > /etc/systemd/system/docker.service.d/network.conf <<EOT
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// --bip=10.0.0.1/16
EOT
$ systemctl daemon-reload
$ systemctl restart docker
```

## Librairies supplémentaires

Ci-dessous les librairies à ajouter sur la machine pour compiler la Matching,
le V-Id Transfère, l'Extract, ... :

```bash
$ sudo apt install libcurl4-openssl-dev libxerces-c-dev libfci-dev
```
