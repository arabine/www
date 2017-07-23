---
layout: page
title:  "Portfolio"
permalink: "/portfolio/"
---

En tant que développeur freelance, je peux vous aider à concrétiser votre projet. Vous trouverez ci-dessous quelques exemples
de mes travaux.

Dans tous les cas, n'hésitez pas à me contacter si vous cherchez quelqu'un pour:

**Volet développement logiciel**
  * Développer du logiciel embarqué sur cible microcontrôleur (ARM, AVR...) avec ou sans RTOS
  * Concevoir un outil PC, par exemple un outil de test automatisé (j'utilise généralement du Qt/C++ avec ou sans Lua/Javascript)

**Volet communautaire/rédactionnel**
  * Participer à la mise en place d'une campagne de financement participatif
  * Gérer une communauté (mailing, forum, blog)
  * Écrire des articles techniques ou un manuel utilisateur

## CV

  * [Curriculum Vitae complet (FR) - PDF](http://rabine.fr/cv/CV.pdf)
  * [Curriculum Vitae simple (EN) - PDF](http://rabine.fr/cv/resume.pdf)
  * [Curriculum Vitae simple (FR) - HTML](http://rabine.fr/cv)

## GitHub

Le code source de tous mes logiciels est disponible sur [Github](http://github.com/arabine).

## Projets industriels

Je possède une expérience de 15 ans dans l'industrie, j'y ai participé à la conception de divers produits essentiellement dans le domaine du comptage et du sous-comptage divisionnaire électrique. L'illustration présente quelques produits, cela va de la centrale de mesure au compteur tarifaire industriel en passant par des transducteurs numériques.

Ces projets m'ont fait découvrir les domaines suivants:

  * Bootloader, mise à jour sur terrain via tête optique/RJ45/Ethernet
  * Lecteur de mémoire flash USB en FAT32 sur STM32
  * Sécurité (AES128 GCM) et compression d'images firmware
  * Intégration de TCP/IP et driver MAC+PHY Ethernet (Microchip ENC28J60)
  * Protocoles de communication (DLMS/Cosem, Modbus/RTU, Modbus/TCP, CPL G1 G3, TCP/IP)
  * Affichage LCD: drivers bas niveaux et séquences d'affichages, graphismes LCD Bitmap
  * Beaucoup de pilotes bas niveau (drivers, gestion de la mémoire flash NOR, UART, SPI, I2C, RTC)
  * Beaucoup d'architecture firmware statique (définition des composants et des interfaces interfaces) et dynamique (séquences d'initialisation/arrêt, top seconde, threads, calculs synchrones)
  * Beaucoup de programmation système (librairies, séquenceurs mono-thread, linkage multi exécutables)
  * RTOS: écriture de wrappers, réglages et customisations
  * Applicatif comptage (calendrier tarifaire, registres, entrées/sorties impulsions et TOR)
  * Banc de production
  * Scripting Lua et création d'utilitaires de tests automatisés
  * Architectures dsPIC, AVR ATmega, DSP Texas C5200, ARM9, ARM Cortex-M4 (STM32, SAM4)

![portfolio]({{ site.url }}/assets/portfolio/mozaic.png)

Produits Enerdis (Groupe Chauvin-Arnoux) : Transducteur numérique, centrales de mesures, concentrateur d'impulsion
Produits Itron : différents compteurs tarifaires pour l'international, l'Espagne, l'Arabie Aoudite, la France (Linky)

## Projets d'écriture technique

Durant mes études j'ai été pigiste au sein du défunt magazine "PC Team". J'y ai écrit pendant près de deux ans une rubrique mensuelle consacrée à l'électronique. J'ai essayé de
proposer des sujets variés autour de l'ordinateur et de la robotique en gardant une ligne "débutant" et en tenant d'expliquer simplement les technologies.

Vous trouverez sur ce site un certain nombre de ces articles, la plupart étant techniquement obsolètes.

## Projets personnels

Étant passionné par la programmation, j'ai conçu quelques programmes, utilitaires et jeux pour mon plaisir personnel ou pour combler un manque dans l'offre existante.

Je tente de toujours mettre mes contributions sous licence GPL ou BSD ( Mon OpenSource), histoire de diffuser plus largement possible mon travail. Voir mon compte [Github](http://github.com/arabine) pour plus de détails.

### TarotClub

Je maintiens notamment TarotClub, un jeu de Tarot. Les technologies utilisées sont :

  * Framework Qt
  * Framework en pur C++ créé pour l'occasion, appelé ICL (Independant C++ Library)
  * Du JSON, du socket réseau TCP/IP multiplateformes

Plus d'information sur le site web de TarotClub [http://www.tarotclub.fr](http://www.tarotclub.fr)

![portfolio]({{ site.url }}/assets/portfolio/windows-game.png)


### Tanca

J'ai également développé Tanca, un logiciel de gestion de club et de concours de Pétanque. Il permet de faciliter grandement l'oganisation des concours de Pétanque pour déterminer les rencontres en fonction du classement (développement d'un algorthme d'appariement rapide pour système de type Suisse). Plus d'information sur le site web de Tanca [http://www.tanca.fr](http://www.tanca.fr).

![portfolio]({{ site.url }}/assets/portfolio/tanca.png)
