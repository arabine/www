---
layout: post
title: "Développement sur carte STM32F469 Discovery avec Linux"
date: 2018-01-01 18:43:00 -0700
comments: false
published: false
category:
- embedded
---

> Linux n'est pas forcément la plateforme idéale pour le développement embarqué, non pas à cause de Linux lui-même mais surtout du non support au niveau outillage et pilotes de périphériques de la part des fondeurs. Les choses changent cependant peu à peu et nous allons vous présenter pas à pas comment paramétrer votre environnement pour du développement embarqué sur ARM Cortex-M4.

# Introduction

La plateforme matérielle qui sera utilisée dans cet article est la carte STM32F469-Discovery qui dispose d'un superbe écran LCD capacitif. L'avantage des cartes ST est qu'elles embarquent toujours de quoi programmer le microcontrôleur. Pas besoin de s'outiller de sonde JTAG donc.

Comme tout développement embarqué (ou pas), nous allons essayer de travailler dans un ordre logique. Dans un premier temps, nous nous intéresserons au flashage de la carte. Puis nous installerons ce qu'il faut pour compiler un programme d'exemple que nous tenterons de déboguer sur cible.

## Présentation de la carte STM32F469I-DISCO


## Documentation

Il vous faudra :
  * Le manuel de référence du microcontrôleur STM32F469
  * La datasheet, plutôt orientée électronique
  * Enfin, le manuel de la discovery board, présentant la carte et les périphériques disponibles

Les manuels du microcontrôleur sont à consulter lorsque vous avez un problème sur un périphérique en particulier, sinon lisez bien attentivement le manuel de la carte, elle fourmille d'informations utiles.

Dans notre cas, notre premier exemple sera de faire clignoter une LED.

## logiciels

En dehors des logiciels de développement, il vous faudra :
  * Un bon éditeur de texte (Kate, Vim, Atom ...)
  * Git, et savoir s'en servir
  * Cloner le dépôt de notre projet qui servira dans notre article (https://github.com/arabine/stm32f469i-disco-tutorial.git)


# Flashage de la carte

Tout d'abord, pour tester la programmation, il nous faut quelque chose à programmer, c'est-à-dire un binaire (programme déjà construit). Je vous ai préparé quelques binaires prêt à flasher pour vos essais.

Ils sont disponibles dans le répertoire "bin" du projet Github. Voici leur description

  * blinky: le classique "hello world" de l'embarqué, ça fait clignoter la LED verte sur la carte

## STLink (Linux)

STlink (pour Linux !) est un projet assez récent et prometteur visant à reproduire l'utilitaire de programmation fournit par ST, mais sous Linux. Il fonctionne très bien, en tout cas pour notre carte, et permet même de débiguer grâce à un serveur GDB embarqué, à l'instar d'OpenOCD.

https://github.com/texane/stlink

## Segger

Segger est un fabricant Allemand célèbre pour ses merveilleuses sondes JTAG et son écosystème associé. Il fournit un outil sympa permettant de transformer n'importe quel programmeur STLink en sonde JLink de Segger. Malheureusement, l'outil n'est disponible que sous Windows. C'est néanmoins une des meilleures solutions sous Linux car les drivers sont disponibles et les outils de qualité.

https://www.segger.com/downloads/jlink#STLink_Reflash



# Compiler le programme d'exemple





# Conclusion

Nous voici donc dotés d'un sytème simple, reproductible et extensible pour nos projets modulaires.
