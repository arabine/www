---
layout: post
title: "Systemd est magique"
date: 2018-12-31 8:55:00 -0120
comments: false
published: true
category:
- raspberry
---

> J'adore systemd, la transversale boîte à outil système intégrée à de nombreuses distributions de Linux. Les commandes simples et puissantes me rendent la vie facile lors de mes prestations ! Voici quelques fonctions que j'utilise

# Démarrer un processus automatiquement

Pratique pour lancer un serveur Web par exemple, en tâche de fond. S'il plante, systemd va le redémarrer. Des logs sont enregistrés pour permettre une analyse à postériori. N'oubliez pas d'indiquer le 'WorkingDirectory' car souvent les programmes utilisent des chemins relatifs à leur exécutable pour charger des ressources (images, logs, config, etc...).

Dans l'exemple suivant, il s'agit de démarrer un serveur Web en Nodejs. J'ai ajouté des options pour injecter une variable d'environnement et j'utilise un utilisateur spécifique pour ce service.

```shell
[Unit]
Description=tarotclubweb
After=network.target

[Service]
WorkingDirectory=/opt/www/tarotclub/tarotclub/www
ExecStart=/usr/bin/node /opt/www/tarotclub/tarotclub/www/app.js
Restart=alway
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=tarotclubweb
Environment='NODE_ENV=production'
User=d8s

[Install]
WantedBy=multi-user.target
```
Sauvegardez ce script dans un fichier situé dans :  /etc/systemd/system/<nom_du_service>.service

Les commandes principales sont :

```shell
systemctl enable tarotclubweb.service
systemctl start tarotclubweb.service
systemctl stop tarotclubweb.service
systemctl status tarotclubweb.service
```

La première est à exécutée une seule fois. Les autres sont explicites ! Il existe PLEIN d'options, le tout est je trouve très simple d'écriture. Enfin un truc non cryptique sous Linux.

# Montage des disques automatique

Il m'arrive souvent de devoir monter des disques réseaux, soit chez moi pour des sauvegardes ou syncrhonisation, soit chez des clients pour les mêmes raisons. Facile, on utilise une entrée dans le fichier fstab pour que ce disque soit monté automatiquement au démarrage de la machine, par exemple :

```shell
//10.0.0.1/nas /media/nas cifs vers=3.0,user,guest,sec=ntlmssp,auto,rw,uid=pi,gid=pi,iocharset=utf8  0  0
```

Or, il arrive que ces disques soient réguièrement déconnectés (extinction d'oridinateur, disque dur USB débranché, réseau ou Internet qui va et vient ...) ou si l'ordre de démarrage n'est pas respecté (Linux démarre avant le branchement du disque) . Si un script de sauvegarde doit s'exécuter, il va planter. Ajoutez cette option à votre ligne fstab (dans la liste des options séparées par des virgules) :

```shell
x-systemd.automount
```

Magique ! Un simple 'ls' ou tout autre accès sur le répertoire de destination va lancer son montage automatique.
