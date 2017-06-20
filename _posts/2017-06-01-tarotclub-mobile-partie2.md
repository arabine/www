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

Notre voyage commence par résumer le cahier des charges de manière plus syntétique, c'est-à-dire sous la forme d'un tableau récapitulatif.

## Plateformes cibles

Tout d'abord, voici les différentes variantes et l'usaga des modules actuels de TarotClub :

|   |     Windows  |   MacOS  |   Linux  |   Android |   iOS   |  WindowsPhone   |   Web (1) |
|---|---|---|---|---|---|---|---|
| Moteur du jeu (C/C++)  |    X     |     X     |    X   |      X  |         X    |     X        |     -|
| Outils(2)      |            X      |    X   |      X      |   -     |      -     |     -         |    -   |
| GUI (plateau de jeu)    |   X     |     X    |    X     |    X    |       X      |   X         |    X  |
| GUI (autres, Qt)     |          X   |       X    |     X  |       -   |        -   |      -       |      -  |
| IA (Javascript) (3)     |          X   |       X    |     X  |      X   |       X  |      X       |     X  |

(1) La version Web ne sera qu'un front-end et fonctionnera en réseau uniquement

(2) Les outils sont l'éditeur de donne, le calcul de points etc. Ils sont utiles mais pas indispensables pour jouer.

(3) Pour la version Web, les scripts d'IA pourront être utiliser par le front-end comme une aide

## Priorités

Les priorités sont les suivantes :
  1. Mutualiser le code du GUI (plateau de jeu, cartes ...)
  2. Mobile first (gestion des résolutions d'écran)
  3. Pas de langage chelou
  4. S'amuser

## Architecture

À la vue du tableau précédent, on peut se poser une question : est-ce que Qt a un intérêt pour les plateformes mobiles ?
En effet, le moteur du jeu étant en pur C++ nous avons une certaine liberté. Les outils eux pevent rester en Qt et sont très orientés "Widget desktop" traditionels, tout du moins pour le moment. Une version mobile pour eux seraient aussi
envisageable mais je laisse cette partie à d'autres aventures.

Concernant l'architecture, le code C++ tournera en tâche de fond et le dialogue avec le front-end se fera en TCP/IP ;
pas besoin de s'embêter avec des wrappers pour les différents langages mobiles (Java, Objective-C).

Voici donc notre première ébauche de diagramme d'architecture.

![tarotclub]({{ site.url }}/assets/articles/tarotclub-mobile-partie2/tarotclub_new_arch.png)

Nous allons commencer par le haut, choisir la technologie du GUI, pour ensuite terminer par le choix du framework capable d'embarquer tout cela proprement. D'ailleurs la réponse peut être Qt mais nous nous autorisons tout autre
technologie séduisante.

# Choix du GUI

Désirant mutualiser le code GUI, partons de la plateforme a priori la plus restrictive : le Web. Avec un peu de recherche préalable, on voit que l'écosystème est maintenant à peu près mature pour transformer une appli Web en une
application desktop.

Là, il n'y a qu'un seul langage standard, c'est le Javascript. Je n'ai pas de contre indication phylosophique avec ce langage, d'ailleurs je l'utilise pour les scripts d'IA. Je n'aime pas les transpileurs (surcouche qui génère du Javascript) mais si toutefois je change d'avis je sais qu'il en existe de très bons comme le langage TypeScript.

Regardons maintenant ce que nous offre le Web pour afficher des graphismes.

## DOM, CSS et JS

Avec un CSS maintenant très puissant en terme de possibilité (animations et transformations), c'est peut-être un bon
candidat. De plus, les cartes de TarotClub sont au format SVG, du vectoriel, dont l'interprétation est native pour tous
les navigateurs.

Si je poursuis dans cette solution, je pourrais faire tourner le GUI en embarquant un navigateur dans mon application
Desktop, comme le fait l'application Slack ou l'éditeur Atom. En effet ils se basent tous sur Electron pour développer
une application de bureau avec les mêmes outils qu'un développement web (NodeJS notamment pour la partie exécution de code).

Au niveau du réseau, Javascript peut dialoguer en WebSocket qui est déjà intégré au sein de la stack de TarotClub, donc
aucun problème de ce côté là.

Par contre, je dois avouer que faire du CSS ou du HTML s'apparente plus à une punition pour moi. En tout cas sur le
papier, je ne vois pas de contre indication générale.

Voici ce que donne notre architecture potentielle :

![tarotclub]({{ site.url }}/assets/articles/tarotclub-mobile-partie2/tarotclub_new_arch2.png)


## HTML5 Canvas 2D

Le canvas 2D deu HTML5 offre une solution pour afficher des éléments 2D au niveau du pixel et offre un certain nombre
de primitives pour aider le développement, surtout des jeux. Ok c'est un bon candidat, par contre il va falloir créer
tout un système de menu et un HUD au dessus du jeu, mais c'est quelque chose d'assez simple à faire (quelques contrôles
  suffisent comme des boutons ou des checkboxes).

Pour embarquer du code Canvas 2D dans une application desktop, on reprend la même astuce que précédemment. Ou alors, autre astuce avec Qt, on dispose d'un package Canvas2D compatible avec l'API Web. Dès lors, on peut espérer obtenir
un package final plus contenu en terme de taille. Par contre, ce n'est pas un environnement Web complet, donc impossible d'effectuer des requêtes DOM ou du WebSocket.

Voici notre nouvel arbre de solutions :

![tarotclub]({{ site.url }}/assets/articles/tarotclub-mobile-partie2/tarotclub_new_arch3.png)

## HTML5 WebGL

L'architecture ressemble au Canvas 2D sauf que ... ben tout est en 3D! Les avantages et inconvénients sont à peu près les
mêmes, voire plus contraignant car il faut, a priori, que la plateforme hôte supporte l'OpenGL.

Je ne mets pas à jour l'architecture, cela n'a pas bougé.

## HUD et futures évolutions

Réfléchir sur l'affichage des graphismes du jeu uniquement ne suffit pas. On oublie souvent les accessoires : un système de menu et le HUD (Head Up Display), c'est à dire tout ce qui s'affiche, généralement en 2D, devant le jeu. On trouve tout un tas d'indications, des cartes, des menus ...

![tarotclub]({{ site.url }}/assets/articles/tarotclub-mobile-partie2/wow_hud.jpg)

**_Non, ce HUD est tout à fait clair, je ne vois pas de problème_**

Après quelques recherches, il s'avère que le meilleur moyen, lorsque le jeu est affiché sur un Canvas, de réaliser le
HUD en HTML, par dessus. Tout simplement. Le gros avantage est que le HTML est exactement fait pour cela, afficher
des données, des images et des boutons simplement.

Lorsque qu'un joueur se connecte à un serveur TarotClub, il entre dans un salon de jeu disposant d'un certain nombre de
tables de jeux. Il peut alors clavarder (chatter) avec les autres membres, en publique ou en privé. Difficile de réaliser un tel chat en Canvas, généralement limité dans l'affichage des textes et autres ... sans compter le travail à
réaliser ; il va falloir recréer tout un tas de primitives et les adapter selon les résolutions !

Enfin, sur PC, tous les Widgets sont disponibles en Qt, donc nous ne seront pas bloqués de ce côté là.

## Conclusion partielle

Mettons à jour notre architecture de possibilités. On se lance donc dans :
  * L'affichage du plateau de jeu et des cartes sur un Canvas (identique à toutes les plateformes)
  * Le HUD/réseau en HTML/Javascript pour la version Web
  * Le HUD Widgets natifs sur PC, et le réseau en C++

La version mobile sera l'application Web empaquetée avec le moteur du jeu. Tout ça reste bien entendu assez théorique pour le moment, de compliqués essais s'annoncent.

Voici donc l'architecture statique envisagée :

![tarotclub]({{ site.url }}/assets/articles/tarotclub-mobile-partie2/tarotclub_new_arch4.png)

Idéalement, il nous faudrait à cet instant un diagramme de déploiement. Cela permettrait de visualiser les composants à embarquer pour telle ou telle plateforme. Seulement, ayant à ce stage
une connaissance très parcellaire des frameworks, il est difficile d'aller plus loin pour le moment. Nous y repenserons après quelques tests, considérons nous pour le moment en phase de
faisabilité.

# Premiers essais (et échecs)

Avant de se lancer à coprs perdu dans le développement de la GUI, lançons quelques tests pour conforter notre choix et
avoir une meilleur visilité sur les différents écosystèmes (ou framework).

## Solutions rejetées

Commençons tout d'abord par lister ce que l'on ne veut pas.

Certains me lanceront au visage des Godot Engine, des Löve, Haxe ou autres bidules. Je n'en veux pas ! Ou ils ne gèrent pas le Web, ou ils se basent sur un écosystème totalement non-standard. L'argument OpenSource ne tient pas, car tout code peut très bien tomber dans l'oubli par manque de mainteneur. Javascript n'a pas ce problème. Et accessoirement je ne souhaite pas apprendre un
nouveau langage.

Regardons donc ce que nous fournissent quelques solutions existantes, tout du moins celles qui semblent les plus populaires vu la rapidité des évolutions des frameworks dans le Web.

## Qt WebEngine

Qt propose depuis plusieurs versions un composant appelé Qt WebEngine embarquant le moteur Chromimum que l'on retrouve dans le navigateur Chrome. Dès lors, n'importe quel
programme fonctionnant sur un navigateur quelconque pourra être embarqué au sein d'une application. Attention par contre, impossible d'utiliser le compilateur MinGW car
Chromimum ne compile pas avec. Il faudra donc installer Visual Studio Community, la version gratuite du compilateur de Microsoft.

Essayons le programme minimal fourni par Qt ; nous le compilons en "Release" et regroupons dans le même répertoire  l'exécutable les DLL nécessaires à son fonctionnement.
Voici le résultat :

![tarotclub]({{ site.url }}/assets/articles/tarotclub-mobile-partie2/webengine-qt-release-dynamic.png)

**_Gasp, 80 Mo pour un programme qui ne fait rien_**

Le ticket d'entrée est cher :  80Mo utilisés uniquement par les DLL avec 60Mo uniquement pour le moteur Chromium. On peut espérer que les versions mobiles (iOS et Android)
aient une emprunte plus faible car il est possible d'utiliser le moteur WebView natif des plateformes. D'ailleurs sur iOS nous n'avons pas le choix car il est interdit de faire autrement.

Pour une application Desktop, cela peut être acceptable vu la taille des disques durs mais cela fait quand même un peu mal pour un simple jeu de cartes. Venant du monde de l'embarqué où une application complète tient dans une centaine de kilo-octets, ça fait drôle.

Demi-échec. il faudrait pouvoir utiliser le navigateur fournit par le système d'exploitation plutôt que d'en embarquer encore un autre.

## Support de GCC dans Android

GCC va être supprimé du NDK d'Android en faveur de Clang. Donc, notamment sur Windows et Linux, le compilateur fourni dans le NDK est GCC en version 4.9. Malheureusement pour moi,
cette version ne supporte pas bien la norme C++0x11 et je ne peux pas compiler TarotClub avec cette version.

Donc, je ne peux tout simplement pas utiliser Qt pour la version Android ! Impossible de trouver des informations sur l'évolution de Qt au moment d'écrire cet article, mais de toutes façons Qt devra passer ses librairies Android à Clang tôt ou tard.

Nouvel échec.

## Apache Cordova

Lorsque l'on recherche des frameworks mobiles, on tombe tôt ou tard sur "Apache Cordova", soit tel quel, soit en tant que couche basse d'un framework plus complet (PhoneGap, Ionic...). Alors c'est quoi Cordova ? C'est donc un ensemble d'outils permettant d'abstraire la plateforme cible (PC, iOS, Android) pour le développement d'applicatif utilisant HTML/CSS/Javascript. Bon, ça a
 l'air d'être conforme à notre cahier des charges.

![tarotclub]({{ site.url }}/assets/articles/tarotclub-mobile-partie2/cordovaapparchitecture.png)

 **_L'architecture de Apache Cordova_**

Cordova virtualise un système de fichiers ce qui permet de garder le code actuel (au niveau des fichiers de configuration), les ressources utilisées en lecture sont à charger à l'initialisation.
Sur le papier ça va, ça a l'air justifié et assez simple.

Sauf que, il va falloir déployer du C++ en plus du GUI. Après un rapide essai non concluant en essayant de débugguer une application du mon téléphone, je commence à le mettre mentalement
de côté. Il faut que je regarde comment lier une librairie C++ et l'appeler, au moins pour démarrer le serveur local, après tout se passe en TCP/IP.

Echec partiel.

## CoCos2D-X

Quand je suis tombé sur le diagramme d'architecture, j'ai tout de suite pensé avoir trouvé le candidat idéal :

![tarotclub]({{ site.url }}/assets/articles/tarotclub-mobile-partie2/framework_architecture.jpg)

 **_L'architecture de CoCos2D-X_**

Sur le papier, ça a l'air parfait. Reste à essayer un exemple simple avec du code C++ pour tester.

## Emscripten

Au lieu de chercher une solution hybride (gérant le C++ et le Javascript), une autre possibilité est de transpiler le C++ en Javascript ce que permet Emscripten. Oubliez l'utilisation sous
Windows. Sur Linux, j'abandonne rapidement après des erreurs de compilation ne venant pas de mon code, qui lui compile partfaitement avec Clang/LLVM.

Nouvel échec.

## L'avenir : WebAssembly ?

On tient le candidat idéal : à partir de (presque) n'importe quelle source, on compile directement en langage optimisé WebAssembly, un espèce d'assembleur pour navigateur. A l'heure de l'écriture
de ces lignes, c'est une technologie encore en phase de définition.

# POC (Proof of Concept)

N'étant pas particulièrement enthousiasmé par les solutions existantes, je tente, pour Android tout d'abord, un essai complet. Voici le cadre du POC.

La première chose à faire est de se créer un code d'exemple mimant notre architecture. Pour le GUI, réalisons un "Hello, World" classique, l'affichage d'une primitive en WebGL. Nous utilisons la librairie Three.js pour également tenter l'usage d'une librairie tierce. Notre moteur se contentera, en C++, de créer un serveur TCP/IP et attendra la connexion d'un client en utilisant le protocole WebSocket (encore un truc à tester).

Les données d'entrées sont donc :
  * Un code d'exmple de développement natif (NDK), appel d'un code C à partir de Java
  * Un code d'exemple d'utilisation du composant WebView (navigateur embarqué dans les Android)
  * Un max de forums et autres entrées Stackoverflow pour activer et tweeker notre application (accès à des librairies Javascript, activation du Websocket ....)

## Etape 1: compilation de TarotCore et ICL

Je pars du programme d'exemple du NDK fourni par Google appelé "hello-libs". 

# Conclusion

Voici donc, finalement, notre diagramme de déploiement qui indique, pour chaque plateforme, l'architecture utilisée.

Mon objectif de factoriser le code graphique est partiellement atteint. Je garde une certaine déception n'ayant pas trouvé le candidat idéal. Je vais donc me débrouiller par moi
même pour déployer le code sur différents systèmes, mon espoir repose toujours sur Qt qui, je pense, est le seul à pouvoir me fournir la solution idéale, simplement, et standard.

Dans tous les cas la décision est prise et je peux commencer à travailler.

## Liens

  * Programme d'exemple : http://www.osd.net/blog/web-development/3d-board-game-in-a-browser-using-webgl-and-three-js-part-2/
