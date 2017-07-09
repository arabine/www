---
layout: post
title: "Version mobile de TarotClub - partie 4"
date: 2017-09-30 18:43:00 -0700
comments: false
published: false
category:
- TarotClub
---

> Je développe depuis quelques années un jeu de tarot libre appelé TarotClub, créé à l'origine pour combler le manque de jeux de cartes sur Linux. Il est pour le moment
disponible uniquement sur les trois grands systèmes d'exploitation du monde PC. Cette série d'articles expliquera le cheminement, j'espère logique, du portage du jeu sur
mobile mais également jouable sur une page Web.

# Premières modifications

Le deuxième article était consacré au choix de la technologie. Ceci étant fait, commençons à modifier le programme d'exemple que nous avons trouvé. Pour rappel, il s'agit
d'un exemple de jeu d'échecs dont la jouerie n'a pas été développée. Tant mieux, cela fera moins de choses à supprimer.

Ce programme d'exemple affiche le plateau de jeu avec les pièces en place. Chaque pièce peut être sélectionnée et bougée dans les limites du jeu, par contre on ne peut pas
les déposer, l'exemple se terminant là.

## Affichons une carte

La première étape pour affiche une carte est d'installer Blender, le fameux logiciel libre d'édition 3D. Il nous faut ensuite un modèle de carte à jouer. Difficile de trouver
une carte directement au format Tarot (61mm x 120mm) ; qu'à cela ne tienne, nous pouvons trouver un modèle Blender de carte classique, merci la mode du Poker. Ouvrons le
dans blender afin de modifier ses dimensions

![tarotclub]({{ site.url }}/assets/articles/tarotclub-mobile-partie3/card_model.png)

**_Un modèle récupéré sur Internet qui conviendra à notre base_**

# Premier déploiement sur Android

Bon, nous avons une version pour navigateur. Nous avons une bonne base, maintenant assayons de déployer cette version pour Android.

## Qt is your friend

En utilisant Qt, nous avons deux options :

  1. Partir du programme d'exemple utilisant une version modifiée de Three.js fonctionnant avec QtQuick/QML
  2. Utiliser le conteneur Qt QtWebEngine qui embarque le moteur Chromium





# Conclusion

## Liens

  * Blender
  * Programme d'exemple : http://www.osd.net/blog/web-development/3d-board-game-in-a-browser-using-webgl-and-three-js-part-2/
