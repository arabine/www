---
layout: post
title: "Version mobile de TarotClub - partie 2"
date: 2017-06-30 18:43:00 -0700
comments: false
published: false
category:
- blog
---

> Je développe depuis quelques années un jeu de tarot libre appelé TarotClub, créé à l'origine pour combler le manque de jeux de cartes sur Linux. Il est pour le moment
disponible uniquement sur les trois grands systèmes d'exploitation du monde PC. Cette série d'articles expliquera le cheminement, j'espère logique, du portage du jeu sur
mobile mais également jouable sur une page Web.

# Une plateforme pour les gouverner tous

Notre voyage commence par résumer notre cahier des charges de manière plus syntétique, c'est-à-dire sous la forme d'un tableau récapitulatif.


# Essais et échecs

## Qt WebEngine

Qt propose depuis plusieurs versions un composant appelé Qt WebEngine embarquant le moteur Chromimum que l'on retrouve dans le navigateur Chrome. Dès lors, n'importe quel
programme fonctionnant sur un navigateur quelconque pourra être embarqué au sein d'une application. Attention par contre, impossible d'utiliser le compilateur MinGW car
Chromimum ne compile pas avec. Il faudra donc installer Visual Studio Community, la version gratuite du compilateur de Microsoft.

Essayons le programme minimal fourni par Qt ; nous le compilons en "Release" et regroupons dans le même répertoire que l'exécutable les DLL nécessaires à son fonctionnement.
Voici le résultat :

![tarotclub]({{ site.url }}/assets/articles/tarotclub-mobile-partie2/webengine-qt-release-dynamic.png)

**_Gasp, 80 Mo pour un programme qui ne fait rien_**

Le ticket d'entrée est cher :  80Mo utilisés uniquement par les DLL avec 60Mo uniquement pour le moteur Chromium. On peut espérer que les versions mobiles (iOS et Android)
aient une emprunte plus faible car il est possible d'utiliser le moteur WebView natif des plateformes. D'ailleurs sur iOS nous n'avons pas le choix car il est interdit de faire
autrement.

# Résultats de l'étude

| Solution | Compilateur Windows | Compilateur MacOS | Compilateur Linux |  |
|---|---|---|---|---|
|  |  |  |  |  |
|  |  |  |  |  |
|  |  |  |  |  |

# Conclusion

## Liens

  * Programme d'exemple : http://www.osd.net/blog/web-development/3d-board-game-in-a-browser-using-webgl-and-three-js-part-2/
