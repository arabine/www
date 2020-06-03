---
layout: post
title: "Premiers pas sur l'architecture RISC-V"
date: 2020-06-15 8:55:00 -0120
comments: false
published: true
category:
- riscv
- arm
- microcontroller
- embedded
---

> Une révolution, tout simplement. Depuis que je travaille dans le domaine embarqué, j'ai connu quelques changements intéressants avec notamment l'arrivée de l'architecture Cortex-M, mais là, c'est un cran au dessus. Une architecture de microcontrôleur Open Source, j'en ai rêvé, maintenant elle existe. 

# Rappel sur les microcontrôleurs

Petite piqûre de rappel : les microcontrôleurs sont de très petits processeurs équipés de périphériques pour contrôler le monde extérieur au composant : des actionneurs ou des capteurs. De plus, ces composants disposent de leur propre mémoire pour y stocker le logiciel (ROM/RAM), cela en font donc de véritables petits ordinateurs prête à l'emploi.

En pratique, on obtient un composant électronique avec plus ou moins de pattes qui sera l'élément central d'une carte électronique.

![image]({{ site.url }}/assets/articles/getting-started-riscv/mcu_example.png)

**_Exemple de micro_**

![image]({{ site.url }}/assets/articles/getting-started-riscv/pic24_board.jpg)

**_Exemple de carte de développement_**

Continuons par un panorama non exhaustif des architectures existantes. Il en existe plein, mais les plus populaires sont :

* La gamme Microchip, PIC12, PIC16, PIC24 sont très populaires, notamment grâce à leur bonne intégration dans l'outil MPLAB du fondeur. Je les trouve régulièrement chez mes clients car les outils (sondes JTAG comprises) sont peu chers. Ces architectures ne sont pas licenciées à des tiers.
* La grande famille Cortex-M3 : tous les fondeurs ont créé leur gamme à partir de cette IP appartenant à ARM qui vie grêces aux licences et royalties sur chaque composant vendu
* Chez TI, on trouve des MSP430 et autres DSP intéressants ; là aussi, c'est du propriétaire
* La gamme AVR, chez Atmel, maintenant Microchip, popularisée par l'Arduino
* Les fondeurs Japonais : Hitachi/Renesas/NEC/Mitsubishi ont créé des architectures plutôt intéressantes, j'ai eu l'occasion de travailler sur des NEC à la mémoire Flash incroyablement performante

Depuis 15 ans maintenant, ce petit monde se côtoyait ; l'architecture ARM est devenue populaire pour avoir rapidement misé sur la faible consommation sans détériorer les performances. C'est vraiment l'architecture qui a fait passer les architectures embarqués vers le tout 32-bits. Elle est devenue au fil des ans un standard de fait et en choisissant un microcontrôleur ARM on s'assurait d'une disponibilité très grande d'applications tierces compatibles (notamment les RTOS), d'un éco-système large et d'un savoir faire largement répendu (livres). Par exemple, on avait ENFIN le choix de ses outils de développements comme la sonde JTAG, le compilateur et l'éditeur/débogueur.

# Historique RISC-V

Revenons au RISC-V.

C'est en l'an 2010 que l'architecture RISC-V est née au sein l'Université de Californie à Berkeley, aux États-Unis. Cette spécification décrit un jeu d'instructions 32, 64 ou 128 bits est libre et dénuée de tout royalties. Une fondation a été créée et a même récemment déménagée en Suisse ; tout est bon pour pérenniser le projet.

![image]({{ site.url }}/assets/articles/getting-started-riscv/riscv.org.png)

**_Le site de la fondation, bon point de départ_**

Écrire une spécification, c'est bien ; l'étape suivante, c'est d'avoir une implémentation réelle, un composant facilement accessible au commun des mortels. Paradoxalement, ce sont des nouveaux venus (pour moi) sur le marché qui ont sorti les premiers composants. Le site riscv.org liste les fondeurs et les fournisseurs de matériel (cartes électroniques). Dans un premier temps, ce sont surtout des implémentations non libres qui sont sorties (HiFive, Andes ...) mais des projets libres sont montés (https://www.lowrisc.org/).

Voilà, allez voir la page Wikipedia pour en apprendre plus ainsi que le site de la Fondation : riscv.org.

# Le SoC Freedom E310

Intéressons-nous au SoC (System on Chip) Freedom E310 créé par la société SiFive. Elle va nous permettre de tester facilement le composant en proposant une carte de forme compatible à Arduino, la HiFive version 1 qui existe en deux modèle : l'originale et la version B.

![image]({{ site.url }}/assets/articles/getting-started-riscv/board_hifive1.jpg)

**_Première version_**

![image]({{ site.url }}/assets/articles/getting-started-riscv/board_hifive1_revb.jpg)

**_HiFive RevB_**

Lorsque j'ai regardé les spécifications du microcontrôleur, j'ai été un peu surpris : habitué à des composants modernes bourrées de périphériques, le E310 est ... particulier ! Notez :

* 320MHz, pour un micro, c'est énorme (je traite d'habitude avec des composants entre 8 MHz et 100 Mhz)
* Pas de mémoire flash interne, mais un accès Quad SPI où le code est stocké ; ce n'est pas courant, probablement parce que concevoir de la mémoire Flash est un savoir faire assez spécial
* 16kB de RAM, pas hyper impressionnant surtout si on veut utiliser un RTOS. Les applications seront limitées en taille
* Coeur RV32 'I' (A load-store ISA with 32, 32-bit general-purpose integer registers) avec les options :
  - M : Integer Multiplication and Division
  - A : Atomics
  - C : 16-bit Compressed Instructions
* Du JTAG, on en reparlera plus tard
* PWM, UART, SPI ... okay, mais pas d'I2C ! (corrigé dans la version B)

D'ailleurs la version B est plus intéressante pour une utilisation réelle dans une application IoT :

* Hardware I2C to read from digital sensors
* Additional UART to communicate with other peripherals
* Low-power sleep mode, keeping only the minimal amount of logic in the Always-On domain powered
* On board wireless networking

La deuxième révision est donc la plus intéressante.

Sur la carte, on retrouve un composant de type USB-UART (puce FTDI) qui nous servira à injecter du code dans la mémoire flash externe et à déboguer notre application.

Voici les principaux périphériques :

![image]({{ site.url }}/assets/articles/getting-started-riscv/board_components.png)

Et voici le diagramme bloc de l'intérieur du composant :

![image]({{ site.url }}/assets/articles/getting-started-riscv/e310_block_diagram.png)

Bon, après ce rapide tour d'horizon, quelques remarques peuvent être dégagées :
1. La carte Rev B semble plus intéressante, au moins pour l'I2C
2. Pas de mémoire flash interne au micro ce qui impose d'ajouter un composant supplémentaire : dommage pour les petits designs.
3. Des périphériques un peu limités, c'est brut de décoffrage, le minimum syndical

# Premier branchement

Branchez la carte à votre ordinateur via le port USB. Une liaison série apparaît, genre un /dev/ttyUSBx. Lancez votre terminal série préféré, GtkTerm de mon côté, et appuyez sur le bouton reset pour redémarrer pour observer le petit message suivant (les paramètres de la liaison série sont affichés en bas) :

![image]({{ site.url }}/assets/articles/getting-started-riscv/demo_console.png)

Vous noterez le petit effet d'arc-en-ciel généré par la LED RGB. C'est bon, le programme embarqué livré de base. Le microcontrôleur dispose d'un chargeur de démarrage embarqué dans la mémoire flash, un truc du genre Arduino, qui ne sera pas effacé par votre programme utilisateur.

# Essai de logiciels

Maintenant que l'on a le hardware, il nous faut du logiciel, notamment le compilateur mais aussi le débogueur. Pour cela, SiFive a préparé un bundle autour d'une version d'Eclipse modifiée, FreedomStudio.

Téléchargez le à partir du site du fabricant, dézippez et lancez le. Au premier démarrage, plusieurs écrans vont apparaître vous proposant de choisir les différentes cibles supportées et vos outils de développement.

Choisissez les paramètres suivants :

![image]({{ site.url }}/assets/articles/getting-started-riscv/freedomstudio_project.png)

Normalement le projet devrait compiler automatiquement et avec succès puis la fenêtre de lancement du débogage apparaît :

![image]({{ site.url }}/assets/articles/getting-started-riscv/freedomstudio_launch_debug.png)

Laissez les paramètres par défaut et programmez la cible. Le log OpenOCD s'affiche en rouge, ça fait peur mais tout se passe (normalement) bien. Sur votre terminal série devrait s'afficher le message suivant (touche reset sinon pour relancer).

![image]({{ site.url }}/assets/articles/getting-started-riscv/example_hello.png)


Vous pouvez tenter de modifier le programme en ajoutant par exemple une petite boucle infinie et un peu de code. Tentez de placer un point d'arrêt sur une ligne de code (Maj+Ctrl+B) ; magique, ça fonctionne, le code s'arrête.

SiFive n'est pas le seul à fournir des outils de développement. Outre GCC et LLVM, l'éditeur propriétaire IAR fournit aussi son compilateur (et probablement bien intégré à son IDE). Au niveau des débogueurs professionnels, les sondes Segger et Lauterbach sont compatibles.

Nous voyons donc que l'industrie s'intéresse à cette plateforme et les grands éditeurs sont au rendez-vous. On attend maintenant les fondeurs !

# Conclusion

La carte HiFive est vraiment sympa ; compatible Arduino, elle vous permettra d'utiliser bon nombre de cartes d'extensions existante. Tout a fonctionné du premier coup, cette première approche de l'architecture RISC-V est un succès.

Notez que l'architecture RISC-V est large : la société SiFive fournit également une carte disposant d'un processur plus puissant capable de faire fonctionner Linux dessus. Le futur s'annonce captivant.

