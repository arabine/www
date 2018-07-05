---
layout: post
title: "Créer facilement un hotspot 4G sur Raspberry PI"
date: 2018-07-05 13:55:00 -0120
comments: false
published: true
category:
- raspberry
- linux
---

> Certes, il est plus facile d'acheter un hotspot tout fait, mais j'en ai vraiment marre des produits dont le support finit toujours par
terminer sans plus aucune mise à jour de la part du fabricant. Je préfère donc opter pour un système plus souble et OpenSource. Mon usage de ce hotspot 4G : ligne internet pro de secours, hotspot multimédia pendant le week-end.

# Matériel

Pour ce projet, il faut rassembler :

  * Un Raspberry PI 2 (avec clé WiFi) ou 3, avec un boîtier quelconque
  * Cable ethernet (pour la mise en service)
  * cable liaison série/USB FTDI ou clavier/écran ou en SSH (paramétrage initial)
  * Dongle 4G Huawei e3372

![hotspot]({{ site.url }}/assets/articles/hotspot-rpi/ftdi.png)

J'aime bien utiliser un câble FTDI pour me brancher sur le PI, ce qui permet d'avoir la console Linux même sans réseau. Et c'est plus pratique que de jongler avec un clavier/écran supplémentaire.

Pour le branchement du câble série FTDI, il faut brancher le Tx du Pi sur le Rx du module USB, et le Rx du Pi sur le Tx du module, puis brancher la masse. Le brochage est inscrit au dessus du connecteur du module, en tout petit !

![hotspot]({{ site.url }}/assets/articles/hotspot-rpi/raspberry_pi_pinout.png)

Concernant le dongle, le Huawei est très bien et a l'avantage de fonctionner sans aucune manipulation sous Linux. J'ai acheté une version directement sur AliExpress avec une antenne amplificatrice intérieure.

![hotspot]({{ site.url }}/assets/articles/hotspot-rpi/Huawei-E3372.png)

# Logiciel

Pour flasher une image Raspian, j'ai utilisé l'utilitaire Etcher, disponible pour Linux, Windows et MacOS (https://etcher.io/).
Concernant la distribution, préférez une Raspbian Lite, sans interface graphique (https://www.raspberrypi.org/downloads/raspbian/).

![hotspot]({{ site.url }}/assets/articles/hotspot-rpi/etcher.png)

Sous Windows, TeraTerm est très bien, aussi bien pour la liaison série que le SSH. Ne mettez pas l'écho car la console Linux le fait.

Concernant la connexion par SSH, récupérez l'adresse IP du Raspberry PI assignée par votre box/routeur. Attention par défaut le SSH est désactivé sur la raspian lite. Il faut créer un fichier (vide) se nommant juste 'ssh', sans extension, à la racine de la carte SD (partition boot) pour l'activer. Question de sécurité !

Voici ma configuration de TeraTerm pour que la console soit plus lisible :

![hotspot]({{ site.url }}/assets/articles/hotspot-rpi/teraterm_font.png)
![hotspot]({{ site.url }}/assets/articles/hotspot-rpi/teraterm_serial.png)
![hotspot]({{ site.url }}/assets/articles/hotspot-rpi/teraterm_terminal.png)

# Mise en oeuvre

## Préparation

Branchez d'abord votre clé Huawei sur un PC Windows (je n'ai pas testé sur un Linux), on a ainsi accès à une page web intégrée
permettant de paramétrer le code PIN et le réseau préféré. Dans le cas de Free, imposez du LTE (4G) uniquement pour bénéficier
du quota 4G maximal et éviter le hors forfait en 3G dont le quota est bien plus limité (3Go !).

J'ai également coché l'option permettant de ne pas demander le code PIN.

## Branchez les périphériques

Branchez la clé Huawei et, si vous avez un Raspberry PI 2, un dongle Wifi compatible.

## Point d'accès Wifi

Connectez-vous comme vous voulez, voici les paramètres de connexion :

 * login : pi
 * pass : raspberry

Tout d'abord, il faut installer le paquet suivant, le démon en tâche de fond réalisant la passerelle
proprement dite :

```shell
sudo apt-get install hostapd
```

Nous allons ensuite installer une interface graphique de paramétrage (https://github.com/billz/raspap-webgui) :

```shell
wget -q https://git.io/voEUQ -O /tmp/raspap && bash /tmp/raspap
sudo apt-get install hostapd
```

Par défaut, nous avons les paramètres suivants :

```
IP address: 10.3.141.1
	Username: admin
	Password: secret
DHCP range: 10.3.141.50 to 10.3.141.255
SSID: raspi-webgui
Password: ChangeMe
```

Après redémarrage, un réseau wifi apparaît :

![hotspot]({{ site.url }}/assets/articles/hotspot-rpi/wifi_name.png)

Connectez-vous à l'aide des paramètres ci-dessus et changez ce qu'il vous plaît (le SSID Wifi, le mot de passe ...)

![hotspot]({{ site.url }}/assets/articles/hotspot-rpi/raspap.png)
