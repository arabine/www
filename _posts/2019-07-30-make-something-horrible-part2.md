---
layout: post
title: "Make Something Horrible (2)"
date: 2019-08-15 8:55:00 -0120
comments: false
published: false
category:
- jam
- games
---

> Make Something Horrible est une Game Jam organisée régulièrement par le magazine papier (et Web maintenant) Canard PC. Le concours consiste à créer un jeu original, surtout drôle et forcément laid car créé par des gens sans talents graphiques. Moi compris ! Cette année j'ai décidé de participer en ayant comme objectif secondaire la découverte de certaines technologies et la mise en pratique d'architectures. Voici le second chapitre.

# Quelques ajustements et tests

Jusqu'à présent, nous n'avons pas fait grand chose mais cela n'empêche pas d'avoir des problèmes. Le premier a été de tester le jeu avec le navigateur Chrome, test cross-browser que tout développeur Web doit réaliser. Evidemment, le menu ne fonctionna pas. Après quelques recherches, j'ai pu déterminer les causes ce qui a rendu le code plus simple et plus propre.

Autre ajustement, j'ai ajouté une musique d'ambiance sur la page d'accueil. J'ai puisé sur le site opengameart.org qui possède tout ce qu'il faut pour le développeur amateur de jeu. J'ai converti le fichier OGG au format WebM, ce qui a réduit la taille de moitié en plus de rendre la chose plus multiplateforme, a priori.

![image]({{ site.url }}/assets/articles/make-something-horrible-part2/icone.png)

Ensuite, j'ai ajouté un favicon.ico à la racine, les navigateurs en font par défaut la requête. Pour l'icône, j'ai là encore fait fans le rapide, une capture d'écran de la lettre 'B' du titre, arrangée sur Paint.net au niveau du remplissage et de la taille. Plus qu'à utiliser un convertisseur en ligne pour générer un .ico.

# Phase 2 : le gameplay prend forme

Donc, c'est un jeu de cartes. Avant de créer les cartes proprement dites, je souhaitais dessiner un peu le plateau de jeu pour me fixer les idées sur les mécaniques de jeu qui n'existent toujours pas à l'heure où j'écris ces lignes. Dans tous les cas, il faut simplifier les choses au maximum car tout élément dynamique ou interactif nécessitera du code derrière ...

Un jeu de cartes à la Magic ? Oui, pourquoi pas, mais l'aspect camping est alors relayé au second plan. Je lance Inkscape en ayant dans l'idée de créer un décors camping. Uniquement un décors, sans interaction. Je récupère un plan de camping en SVG, vous savez celui avec les emplacements numérotés et les blocs sanitaires ?

![image]({{ site.url }}/assets/articles/make-something-horrible-part2/plan-du-camping.jpg)

Oui, ça peut faire une bonne illustration, mais alors l'image est un peu chargée, avec les cartes par dessus ... puis je trouve que les emplacements pourraient faire un bon jeu de bataille de territoire, à la Crusader King. Mais oui ! Je tiens le truc, on pourrait faire un jeu de gestion carrément complexe avec gestion des ressources pour chaque emplacement ... mais je m'emballe. On va simplifier la carte et 
créer une grille d'emplacements, un peu un camping imaginaire.

![image]({{ site.url }}/assets/articles/make-something-horrible-part2/background.png)

Et voilà, je crée au pif un certain nombre d'emplacements, je ne sais pas si ça va tenir la route ...

Au niveau du code, je m'imagine déjà une matrice en mémoire représentant cette grille. Une carte sera mise sur une case par un glissé-déoposé.

Pour simplifier encore, et réutiliser le glisser déposé, j'imagine deux autres zones : une poubelle (la poubelle du camping bien sûr !) permettant de défausser une carte et une "réserve de Bibine", équivalent à une réserve de Mana. Si une carte est envoyée dessus, la réserve de Bibine augmente. Une fois le maximum atteint (représenté par un verre de Pastis bien entendu), le pouvoir unique du joueur pourra se déclencher.

On résume :

* Tout fonctionne par drag and drop, avec une librairie comme Draggable.js ça peut être super simple
* Plusieurs types de cartes :
   - Des cartes "Bibine" pour recharger
   - Des cartes "Emplacement" à poser sur une case et restent en place le temps qu'il faut
   - Des cartes "Coups spéciaux", événement éphémère

La fin de la partie : lorsque le joueur a conquis tous les emplacaments. Oui, ça me plaît, à la fois simple dans l'action, mais compliqué dans la gestion des cartes, il va falloir faire un système bien générique et pouvoir faire des cartes rapidement, à la volée. J'imagine un fichier de configuration listant toutes les cartes, type JSON.

# L'IA

Franchement, j'ai la flemme de créer une IA, et en plus je n'y connais rien. Je pense que dans un premier temps, le joueur adverse jouera au hasard, et si j'ai du temps j'ajouterai quelques if. J'essaierai de tricher pour rendre l'IA plus dure : elle connaitra les cartes en main du joueur humain.

Dans tous les cas, il faudra se réserver du temps pour cette partie

# Création du HUD

Maintenant que nous savons à peu près où nous allons, ajoutons quelques éléments graphiques. Il nous faut :

* Un menu permettant de quitter le jeu et revenir au menu principal (réutilisation du composant MenuItem.js)
* Une jauge de Bibine, avec le portrait et le nom de nos héros du jour (le joueur humain et l'ordinateur)
* La poubelle (défausse)



