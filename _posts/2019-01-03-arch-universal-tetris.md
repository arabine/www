---
layout: post
title: "Architecture par composant - Tétris générique (1)"
date: 2019-01-03 8:55:00 -0120
comments: false
published: false
category:
- architecture
- embedded
- games
---

> Le code source embarqué (ou non) d'un logiciel termine toujours par un tas de code difficilement maintenable. Il existe néanmoins quelques solutions simples pour éviter le pire et permettre d'avoir, dès le début d'un projet, une base de code évolutive et flexible. Nous verrons cela dans une série d'article en mettant en oeuvre le jeu de Tétris dans une architecture par composant.

# But de notre travail

Le but sera de porter le jeu Tétris sur plusieurs plateformes. Votre projet ne vise qu'une seule plateforme ? Insérez une deuxième cible dès le début du projet, ceci pour plusieurs raisons :

  * Cela vous forcera à créer des couches d'abstrations
  * Cela vous forcera à avoir un système de construction générique
  * Utiliser un système différent, et si possible un compilateur différent, permet de tester un peu le code à moindre coût

Dans le cas d'une cible principale embarquée, il est par exemple intéressant d'avoir comme seconde cible l'ordinateur hôte. Celui-ci est plus rapide et permet par exemple d'effectuer des simulations et du test automatisé rapide du code.

![image]({{ site.url }}/assets/articles/arch-tetris/tetris-js.png)

# Trois notions importantes et complémentaires

De part mon expérience, on peut classer dans trois catégories les éléments importants qui vont nous permettre d'avoir une architecture correcte.

  * D'abord, il nous faut une organisation modulaire des répertoires et du système de construction
  * Puis, il faut s'occuper de l'architecture statique, on visera ici à limiter les dépendances entre les composants
  * Enfin, il faudra travailler sur la partie dynamique (dialogue entre composants et contextes d'exécutions)

Le fil rouge sera de toujours limiter au maximum les interactions dépendantes qui font qu'un composant A nécessite un composant B. Pour cela, on utilisera des techniques de messagerie, d'événements et de liaisons dynamiques (listeners). En prenant en compte tout ceci, la base de notre code sera saine et portable à souhait. Bien entendu, rien n'est parfait, et il y a toujours une tonne de particularité liée à une plateforme.

Dans tous les cas, pensez générique : en général, essayez d'utiliser les particularités d'une spécification tordue (issue d'un service marketing ou d'un client) pour rendre cette fonction générique et commune au reste de votre code.

Au niveau du code prorement dit, nous éviterons les #ifdef le plus possible, cela ne rend pas le code facile à porter.

# Architecture par composant

Une architecture par composant va nous permettre de bien organiser le code source au niveau répertoire et au niveau des API : un composant fournit une interface (fonctions, méthodes virtuelles), son implémentation est cachée. Demain, vous pouvez tout à fait ré-écrire l'implémentation sans bouger une ligne de code dans le reste du projet.

Dans les faits, cette architecture génère deux gros problèmes :

  * La dépendances entre les composants, nous verrons comment s'en sortir avec quelques principes logiciels
  * L'architecture dynamique difficile à contrôler (phases de démarrage, d'arrêts, synchronisation inter-composants et événements temps réels)

# Présentation du code d'origine
