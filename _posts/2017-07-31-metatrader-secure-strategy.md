---
layout: post
title: "Sécuriser une stratégie sous MetaTrader 5"
date: 2017-07-10 18:43:00 -0700
comments: false
published: false
category:
- MetaTrader
---

> MetaTrader est un logiciel de trading en ligne et permet d'écrire son propre robot. Un robot prend la forme d'un script en langage MQL5, un espèce de C++ simplifié, qui aura la charge de lancer des ordres d'achat ou de vente en fonction de tout un tas d'indicateurs plus ou moins mystiques sur la bourse et son évolution au court du temps. Cet article présente un moyen de cacher le coeur du robot, la stratégie.


# Introduction à MetaTrader

Vous voulez gagner de l'argent facilement ? Eh bien ne jouez pas en bourse. Pourtant, c'est ce que le logiciel MetaTrader vous permet de
faire, tout en vous offrant la possibilité de créer votre robot trader qui fera votre forturne (ou pas).

Pour découvrir le logiciel, installons tout d'abord MetaTrader5 (https://www.metatrader5.com). Ceci fait, tentons de faire tourner un script
simple : ce sera notre point de départ pour sécuriser notre stratégie.

Mais tout d'abord, pourquoi sécuriser une stratégie de trading ? Si vous la gardez pour vous-même, il n'y a pas besoin, à par sécuriser votre ordinateur et l'endroit où vous stockez vos scripts. Par contre, si vous souhaitez vendre votre stratégie, alors il est important de s'intéresser à la chose de plus près afin de protéger votre savoir-faire.

Un script QML5 est un fichier texte tout simple (extention .mq5) qui, une fois compilé, génère un code binaire "exécutable" (extension .ex5). Ce binaire est exécuté par la machine virtuelle propriétaire à MetaTrader.

Qu'en est-il de la sécurité des scripts MQL5 et du format exécutable EX5 ? Malheureusement, il semble exister des décompilateurs qui transforment le code binaire en source lisible par un humain. Un format populaire est de toute façon attaqué de toute part.

En général, j'aime bien "casser" les données d'entrées (le cahier des charges) : faisons de même ici. En regardant de plus près la documentation en ligne, on voit qu'il est possible d'appeler des fonctions stockées dans une DLL.

Bingo ! Laissons tomber MQL5, développons notre code dans une DLL ... ou mieux, utilisons la DLL somme proxy (intermédiaire).

Voyons cela de plus près.

# Les bonnes pratiques et les limites de la sécurisation

Avant toute chose, soyons clairs. N'importe quel hacker acharné arrivera à venir à bout de nos protections. Pourquoi ? Parce que, à un moment donné, notre code sera exécuté par le processeur. Donc, un "homme du milieu" pourra aisément regarder les instructions, les analyser et reconstruire notre algorithme.

Nos protections sont là uniquement pour éviter que le margoulin de base n'utilise un utilitaire tout fait (un décompilateur) pour espionner notre code. Le margoulin de base ne s'amusera pas à fouiller au delà et s'arrêtera à la première difficulté rencontrée.

Notre but sera donc de lui pourrir la vie, de mettre un maximum de barrières à la rétro-ingénieurie.

# Cahier des charges

Donc, listons ce que nous voulons :

  * On ne veut pas trop utiliser MQL5, donc le coeur stratégique sera développé dans un autre langage
  *


# Aller plus loin
