---
layout: post
title: "Version mobile de TarotClub - partie 1"
date: 2017-05-05 18:43:00 -0700
comments: false
category:
- blog
---

> Je développe depuis quelques années un jeu de tarot libre appelé TarotClub, créé à l'origine pour combler le manque de jeux de cartes sur Linux. Il est pour le moment
disponible uniquement sur les trois grands systèmes d'exploitation du monde PC. Cette série d'articles expliquera le cheminement, j'espère logique, du portage du jeu sur
mobile mais également jouable sur une page Web.

# Sous le capot

Attardons nous dans un premier temps sur l'aspect technique du logiciel existant. Ce bilan va nous aider dans les choix technologiques pour les plateformes web et mobiles.

## Etat actuel

Il est développé en C++ en utilisant Qt. Les premières versions utilisaient la librairie dans
toutes les classes du jeu. Depuis plusieurs versions majeures, je m'attarde à découpler les différents modules pour ne plus avoir de dépendances entre le moteur du jeu (la partie
appelée "logique") et la librairie graphique.

La dernière version de TarotClub dispose ainsi:

  * D'un client graphique
  * Un serveur dédié (en pur C++ sans aucune librairie)
  * D'un client console (utilisant des caractères graphiques)
  * D'un client "robot" dont les scripts d'IA sont développés en JavaScript

Voici donc la preuve de la parfaite réussite du découplage entre le moteur de jeu et son affichage graphique.


![tarotclub]({{ site.url }}/assets/articles/tarotclub-mobile-partie1/windows-game.png)

**_La version desktop du jeu utilisant Qt_**

## L'interface utilisateur graphique

Le GUI est développé en utilisant complètement Qt car il excelle en la matière. Le moteur graphique choisi à l'époque est le QGraphicsScene qui est spécialisé dans l'affichage 2D de primitives. Il est puissant, facile à
développer mais c'est le parent pauvre du framework. Autrefois au centre de la communication de l'éditeur, il est délaissé par la stratégie du tout QML/QtQuick. Il est vrai que ce dernier a tout pour plaire : accélération
graphique, interpréteur Javascript, moteur d'animation et d'effets graphique et écriture très simple de l'interface utilisateur dans un style JSON.

Le framework Qt possède donc deux grandes façons de programmer une interface (ou un jeu !), l'ancêtre QWidgets et le nouveau QtQuick/QML. Il est temps de réfléchir à remplacer le grand ancien car il risque tôt ou tard de terminer
dans la corbeille de l'éditeur.

![tarotclub]({{ site.url }}/assets/articles/tarotclub-mobile-partie1/qtquick_example.png)

**_Il est vrai que QtQuick/QML semble séduisant_**

## Interpréteur Javascript

Pour l'information technique, l'interpréteur Javascript utilisé se nomme Duktape. Il est particulièrement "petit" en terme d'empreinte mémoire (il peut tourner sur des systèmes
enfouis type microcontrôleurs) et son développement est super dynamique. Son auteur travaille comme un titan pour sans cesse apporter des améliorations et est particulièrement à
l'écoute d'une communauté très active. Les discussions techniques sur Github sont d'un très haut niveau technique.

Attention, si vous cherchez la performance, passez votre chemin même si pour la plupart des usages Duktape s'avère largement suffisant. Restez sur les moteurs de Chrome et Firefox
si la performance est nécessaire pour votre application. Avec Duktape, l'auteur vise surtout la portabilité maximale et le respect de la norme.

Pourquoi avoir choisi de développer une IA en langage scripté ? Le but premier était de fournir une interface plus simple que le C++ pour motivier les contributions externes.
Puis, l'idée d'une version web cheminant, ce langage devenait naturel car c'est le langage natif des navigateurs.

## Le réseau

L'architecture interne utilise nativement les sockets BSD. C'est-à-dire que derrière une partie locale contre l'IA se cache un serveur de jeu qui dialogue en TCP/IP avec trois clients
IP de type "bot" (les adversaires) et un client IP graphhique humain : vous.

Dès lors, il est très facile de créer mon serveur dédié : je supprime toute la partie graphique et le tour est joué. De plus, étant seul développeur, cela me permet d'éviter de dupliquer du code
et de tester l'ensemble par une seule suite de tests automatisés.

Le découplage va encore plus loin car la partie TCP/IP proprement dite est elle aussi bien séparée du code se chargeant d'encoder et de décoder les données transférées sur le réseau (la sérialisation). Vivent les interfaces
bien pensées, oui je me félicite mais il faut bien se motiver un peu!

Enfin, le code réseau (classes utilitaires serveur/client) contient le support du protocole WebSocket. Toujours en avance de phase, j'ai choisi de l'intégrer rapidement lors d'un refactoring du net code afin de l'intégrer
nativement dès le départ de façon transparente pour éviter de tout casser par la suite.


![tarotclub]({{ site.url }}/assets/articles/tarotclub-mobile-partie1/tarotclub_current_arch.png)

**_L'architecture actuelle de TarotClub, le code GUI est décorellé du moteur de jeu._**

## Architecture statique

Un dernier mot enfin sur le découplage des fichiers. Toutes les classes "génériques" (réseau, JSON, utilitaires) sont situées dans un répertoire bien séparé au sein d'une librairie que j'ai appelée "ICL" signifiant Independant
C++ Library. Eh oui là encore, j'essaie de ne pas lier les fichiers entre eux pour ne pas tomber dans le plus grand défaut des framework de type Qt où la modularité n'est pas très fine. Avec ce dernier par exemple, il faut
redistribuer un paquet de grosses DLL si on ne compile pas statiquement le projet.

## Perspectives

Nous voici donc avec sur les bras un logiciel en C++ et une interface graphique en Qt. A priori, les deux sont portables assez facilement sur les plateformes mobiles (iOS, Android, Windows Phone). Reste que rien n'a été
pensé "mobile first" donc même s'il est possible de générer un exécutable sur une plateforme donnée, toute l'interface graphique est à revoir.

Pour le web, n'en parlons pas, à part quelques choix technologiques rien n'a été pensé pour l'exécution à partir d'une page HTML.

## Inspirations

J'ai quelques idées en tête sur mon but final. Au niveau du portail communautaire, je souhaiterais dans l'idéal quelque chose comme Lichess.

![tarotclub]({{ site.url }}/assets/articles/tarotclub-mobile-partie1/lichess.png)

**_Le site Lichess est vraiment impressionnant en terme de fluidité_**

Au niveau de l'interface de l'application, certains jeux existants possèdent une interface très simple et bien pensées, comme "French Tarot" de Eryod Soft.


![tarotclub]({{ site.url }}/assets/articles/tarotclub-mobile-partie1/app_tarot.png)

**_Le tiroir de statistiques sur la droite est bien pensé_**

Au final, ce genre d'apparence pour un jeu de carte est assez commun. Si on veut vraiment s'amuser et proposer quelque chose de neuf, pourquoi pas aller dans quelque chose de plus rare. Par exemple, j'adore l'aspect du jeu
Splendor (Asmodee) qui est fait en 3D.

![tarotclub]({{ site.url }}/assets/articles/tarotclub-mobile-partie1/splendor.jpeg)

**_Une interface 3D façon Splendor ?_**

# Cahier des charges

Ok maintenant attaquons nous à lister ce que nous désirons, ce que j'appelle ma "lettre au Père Noël". N'ayons pas peur d'intégrer même l'impossible, rêvons un peu puis lors du choix technologique nous pourrons revenir
un peu sur terre.

## Cibles

Alors là c'est facile, il faudra viser en prioriter les plateformes suivantes :

  * Android
  * Le Web (via une page HTML)
  * iPhone
  * Windows Phone

Idéalement, j'utiliserai le même code pour la version "desktop", mais étant donnée qu'elle existe déjà on peut imaginer partir sur quelque chose de radicalement différent puis unifier le tout plus tard.

## Réutilisation du code

Pour un soucis évident de maintenance et de rapidité, il est nécessaire de pouvoir réutiliser le code de l'interface graphique, c'est-à-dire la table de jeu avec les cartes. Ou du moins une bonne partie, sous la forme de
fonctions/classes réutilisables. Voici donc notre critère numéro 1.

Il faudra également utiliser le code existant, et donc la technologie choisie devra pouvoir s'empaqueter avec du code C++ sous forme native ou une librairie.

## Divers

Dans les diverses propriétés moins importantes, on peut essayer d'avoir un résultat le plus faible possible en terme d'empreinte mémoire, dans la dizaine de méga octets. Ce n'est qu'un jeu de Tarot tout de même !

Ensuite, il faudrait de préférence un langage standard, pas trop exotique, pour ne pas se lier à un seul fournisseur. Je déteste également les transpileur de code qui nécessitent de maîtriser deux langages au lieu d'un.

Il faudra s'adapter à pas mal de tailles d'écrans en ne cherchant pas à être compatible avec de trop vielles configurations.

On va viser l'avenir et pourquoi pas utiliser des technologies assez récentes, je ne suis pas nostalgique et à la limite, je dispose déjà d'une versions "console" utilisable sur des vieux tromblons et une version Qt pour les
configurations intermédiaires.

## Outils

A priori un PC sous Linux et un téléphone Android devraient suffirent pour commencer la première cible à pas trop cher. Le développement pour iOS nécessite d'acheter deux machines : l'hôte pour le développement
et un téléphone cible ; gasp ! Le téléphone ne semble pas obligatoire au début, je piquerai donc les iPhone de mes proches pour essayer l'application. Concernant la machine hôte, un Mac Mini fera l'affaire (500 € tout
de même), en n'oubliant pas les frais de dépôt d'une application (100 €) ?

Même chose pour Windows, il va falloir passer à la caisse pour obtenir une licence Windows 10 et un mobile. Sachant que l'avenir de la plateforme est vraiment floue et la popularité du téléphone, cette cible sera
clairement développée en dernier.

## Se faire plaisir

Eh oui ça compte, j'ai envie d'apprendre aussi quelque chose de neuf.

# Conclusion

On se retrouve pour un prochain article dans lequel nous essaierons de lister et comparer différentes technologies par rapport à ce cahier des charges.

## Liens

  * Duktape : http://duktape.org/
  * Lichess : http://lichess.org/
