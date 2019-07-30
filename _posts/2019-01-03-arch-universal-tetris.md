---
layout: post
title: "Architecture par composant - Tetris générique (1)"
date: 2019-01-03 8:55:00 -0120
comments: false
published: true
category:
- architecture
- embedded
- games
---

> Le code source embarqué (ou non) d'un logiciel termine toujours par un tas de code difficilement maintenable. Il existe néanmoins quelques solutions simples pour éviter le pire et permettre d'avoir, dès le début d'un projet, une base de code évolutive et flexible. Nous verrons cela dans une série d'article en mettant en œuvre le jeu de Tetris dans une architecture par composant.

# But de notre travail

Le but sera de porter le jeu Tetris sur plusieurs plateformes. Votre projet ne vise qu'une seule plateforme ? Insérez une deuxième cible dès le début du projet, ceci pour plusieurs raisons :

  * Cela vous forcera à créer des couches d'abstractions
  * Cela vous forcera à avoir un système de construction générique
  * Utiliser un système différent, et si possible un compilateur différent, permet de tester un peu le code à moindre coût

Dans le cas d'une cible principale embarquée, il est par exemple intéressant d'avoir comme seconde cible l'ordinateur hôte. Celui-ci est plus rapide et permet par exemple d'effectuer des simulations et du test automatisé rapide du code.

![image]({{ site.url }}/assets/articles/arch-tetris/tetris-js.png)

# Technologies utilisées (et limitations imposées)

Le code principal sera en langage C. Déjà parce que le code source d'origine est en C et qu'il est facile de porter du code C à peu près partout. Ce langage est également facilement intégrable à des langages de plus hauts niveaux (langages de scripts) via des appels natifs ou des librairies pré-compilées.

Ce ne sera pas notre unique langage : selon les plateformes cibles, notre code de base sera enrobé de différentes couches en C++ ou en Javascript... nous verrons cela par la suite.

En ce qui concerne le langage C proprement dit, nous allons le limiter au maximum, toujours dans l'optique de la portabilité :

  * Faible dépendance sur la librairie standard C : on limitera les en-têtes au script minimum
  * PAS d'allocation dynamique : et oui, on garde à l'esprit les cibles embarquées
  * PAS de C99, on va essayer, c'est dommage mais par exemple à l'heure d'écriture de cet article Visual C++ ne supporte toujours pas cette norme. La honte. (alors que la plupart des compilateurs embarqués sont ok là dessus comme IAR ou Keil).

# Trois notions importantes et complémentaires

De part mon expérience, on peut classer dans trois catégories les éléments importants qui vont nous permettre d'avoir une architecture correcte.

  * D'abord, il nous faut une organisation modulaire des répertoires et du système de construction
  * Puis, il faut s'occuper de l'architecture statique, on visera ici à limiter les dépendances entre les composants
  * Enfin, il faudra travailler sur la partie dynamique (dialogue entre composants et contextes d'exécutions)

Le fil rouge sera de toujours limiter au maximum les interactions dépendantes qui font qu'un composant A nécessite un composant B. Pour cela, on utilisera des techniques de messagerie, d'événements et de liaisons dynamiques (listeners). En prenant en compte tout ceci, la base de notre code sera saine et portable à souhait. Bien entendu, rien n'est parfait, et il y a toujours une tonne de particularités liée à une plateforme.

Dans tous les cas, pensez générique : en général, essayez d'utiliser les particularités d'une spécification tordue (issue d'un service marketing ou d'un client) pour rendre cette fonction générique et commune au reste de votre code.

Au niveau du code proprement dit, nous éviterons les #ifdef le plus possible, cela ne rend pas le code facile à porter.

# Architecture par composants

Une architecture par composant va nous permettre de bien organiser le code source au niveau répertoire et au niveau des API : un composant fournit une interface (fonctions), son implémentation est cachée. Demain, vous pouvez tout à fait ré-écrire l'implémentation sans bouger une ligne de code dans le reste du projet.

Dans les faits, cette architecture génère deux gros problèmes :

  * La dépendances entre les composants, nous verrons comment s'en sortir avec quelques principes logiciels
  * L'architecture dynamique difficile à contrôler (phases de démarrage, d'arrêts, synchronisation inter-composants et événements temps réels)

Voici ce que donne un code qui est soit disant portable sur plein d'architecture. Voici ce que l'on ne *veut pas*. Franchement, le jour où vous avez à porter le code sur une nouvelle architecture, bon courage. 

```c
#if ENC28J60_USE_SPILIB
   #if defined(ARDUINO)
     #if defined(STM32F2)
       #include <SPI.h>
     #elif !defined(STM32F3) && !defined(__STM32F4__)
       #include <SPI.h>
       extern SPIClass SPI;
     //#elif defined(ARDUINO_ARCH_AMEBA)
       //SPIClass SPI((void *)(&spi_obj), 11, 12, 13, 10);
       //SPI _spi(SPI_MOSI,SPI_MISO,SPI_SCK,ENC28J60_CONTROL_CS);
     #else
       #include "HardwareSPI.h"
       extern HardwareSPI SPI(1);
     #endif
   #endif
   #if defined(__MBED__)
     SPI _spi(SPI_MOSI,SPI_MISO,SPI_SCK);
     DigitalOut _cs(ENC28J60_CONTROL_CS);
     Serial LogObject(SERIAL_TX,SERIAL_RX);
   #endif
#endif
```

Dans cet exemple, il faudrait refactoriser le code en composants. Chaque plateforme cible aura son implémentation de SPI, et implémentera une même interface. Maintenant, comment un composant peut faire appel à une fonction définie ailleur ?

Plusieurs méthodes. L'une que je préfère est d'utiliser des callbacks, à définir dans votre interface de composant et à initialiser au démarrage de votre application. Sinon en désesespoir de cause un bon vieux extern, et c'est au moment du link qu'il faudra régler les problèmes.

# Présentation du code Tetris d'origine

Le code de Tetris d'origine que nous utiliserons pour illustrer notre architecture provient d'un easter egg caché au sein d'un produit industriel doté d'un écran graphique monochrome. Il s'agit d'une centrale de mesure disposant de deux boutons, le bouton de gauche permettant de valider et le bouton de droite disposant des quatre directions de navigation. Pour lancer le jeu, aller dans l'écran de la date-heure, appuyez trois fois sur "OK" puis "OK + gauche".

![image]({{ site.url }}/assets/articles/arch-tetris/tetris-enerium.png)

Nous voyons déjà ce que nous aurons à abstraire : l'affichage et les contrôles. Un équipement électronique embarqué sara totalement différent par rapport à un portage Web, PC ou mobile.

Notre but sera d'abstraire au maximum pour isoler le moteur Tetris proprement dit et ainsi permettre de réutiliser le code au maximum.

# Vers quelles plateformes cibles ?

Alors nous allons nous amuser un peu en variant les contraintes, et dans le désordre :

  * Un portage natif sur PC, avec mettons un affichage graphique en utilisant la librairie Qt
  * Un portage en mode console
  * Un portage vers le Web, eh oui, en utilisant le WebAssembly
  * Un portage embarqué sur plateforme RISC-V et un écran LCD

# Travail préparatoire

Ok, attaquons la chose. Dans un premier temps, nous allons nous extraire le code source du programme d'origine. Pas de problème légal, c'est moi qui l'ai programmé et c'est une fonction cachée, donc aucun soucis là dessus (en espérant que mes anciens collègues aient retiré le code en production ;D). Le code a été écrit en langage C sur cible DSP (un Texas TMS320C5502). Heureusement, le code est assez indépendant du reste de l'application, j'avais bien fait les choses à l'époque. Néanmoins, j'ai dû réaliser les tâches de nettoyage suivantes :

  * Passer tous les types propriétaires en types standards (uint32_t, uint8_t, bool) à l'aide des en-têtes stdint.h et stdbool.h
  * Retirer les inclusions d'en-têtes spécifiques à la plateforme (C5502.h)
  * Déménager quelques définitions externes et une variable globale : la mémoire graphique virtuelle

Ce dernier point nécessite une petite explication et un schéma présentant l'architecture. Afin de maximiser le code en commun sur toutes les plateformes et minimiser les couches d'abstractions, le choix avait été fait à l'époque (et quel bon choix !) de dessiner ce que l'on voit à l'écran dans un buffer virtuel.

Ce buffer est ensuite envoyé vers le contrôleur graphique. Il s'agit, en gros, de dessiner sur un fichier, cela génère une image que l'on peut envoyer n'importe où. Le code de notre Tetris repose donc sur une petite librairie graphique permettant de dessiner des primitives : cercle, rectangle, segments, caractères et quasiment le plus important : afficher une image.

![image]({{ site.url }}/assets/articles/arch-tetris/arch-enerium.png)

A noter que le fait de travailler dans un buffer permet de réaliser des fonctions bien pratiques :

  * Nous avions développé à l'époque un double buffer (image N et N-1) permettant de détecter les lignes modifiées entre deux appels : ainsi, nous soulagions le rafraîchissement du LCD afin de limiter le clignotement qui était assez visible par l'utilisateur.
  * Une capture d'écran : sur une requête, on copie le buffer en cours dans un autre coin en RAM, puis on télécharge cette image et on génère un fichier Bitmap.

# Génération d'images

Nous avons à disposition un écran graphique de résolution 160x128 pixels monochromes. Il est intéressant de pouvoir afficher des images tirées d'un fichier image généré avec un logiciel de dessin type Gimp (ou piqué sur Internet). Or, le format d'une image est un peu trop lourd à décoder par un processeur embarqué et on ne dispose pas, généralement, d'un système de fichier assez gros pour stocker les images brutes. Nous allons donc générer, en dur dans le code, un extrait de ces images, un tableau d'octets brut de ce dessin. Ainsi, c'est de la Flash code qui sera utilisée (et non de la data).

Le format source choisi est le BMP car il permet d'encoder l'image avec un bit par pixel, c'est à dire en monochrome, parfaitement optimisé pour nous. Notez que tous les logiciels ne permettent pas cet encodage sauf ...  Paint, de Microsoft, qui le gère très bien depuis le début, ce qui est assez rigolo pour le noter.

Pour cela, j'avais développé à l'époque un utilitaire pour générer un tableau d'octets en C à partir d'une image monochrome BMP. Vous trouverez cet utilitaire dans le répertoire 'tools' du dépôt du projet.

Dans un premier temps on compile l'utilitaire :

```shell
gcc bmp2c.c -o bmp2c
```
Une petite aide en ligne permet de voir le format d'appel et les options :

```shell
C:\git\unitris\tools>bmp2c.exe
Usage: bmp2c.exe input.bmp output.c [-six_pixels] [-16bits_array] [-32bits_array]
```
On lance la conversion, aucune erreur, tout s'est bien passé !

```shell
C:\git\unitris\tools>bmp2c.exe ..\assets\tetris.bmp output.c  -16bits_array

Option enabled : 16bits array.

width : 96 height : 80
bits utilises : 96 padding : 0 octets ajoutes.
Taille du tableau : 960
```

L'opération a donc transformé une image vers un tableau d'octets.

![image]({{ site.url }}/assets/articles/arch-tetris/bitmaptoc.png)


# Conclusion et suite

Nous venons d'expliquer un peu notre projet, notre but et surtout notre vision concernant l'architecture. Nous avons également fait un petit tour d'horizon sur l'architecture dynamique du code d'origine.

Dans le prochain article, nous porterons le code en console, dans une première version permettant de générer une image Bitmap puis avec Qt !

Vous trouverez le dépôt de ce projet sur github https://github.com/arabine/unitris ainsi que le code source en cours d'écriture.


