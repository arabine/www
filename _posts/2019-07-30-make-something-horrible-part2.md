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

* Tout fonctionne par drag and drop, avec une librairie comme Draggable.js ou même D3.js ça peut être super simple
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

La jauge de Bibine sera un verre de Pastis bien connu qui se remplira au fur et à mesure de certaines cartes jouées. Aux choix du joueur : soit il remplit sa jauge pour espérer arriver à 100% et déclancher son pouvoir spécial, soit il joue sa carte ailleurs (défausse, sur le camping, même sur l'adversaire pourquoi pas ?).

Pour cela, j'ai besoin d'une image SVG dont je remplierai le contenant par du fameux contenu jaune anis. Je prends une photo sur Internet que je détoure avec deux mains gauches :

![image]({{ site.url }}/assets/articles/make-something-horrible-part2/numerisation_pastis.png)

Ok, passons maintenant un peu au code parce que c'est marrant. On va créer un nouveau composant, appelé PlayerIcon.js. Ce composant graphique contienra :

* L'image d'un verre de Pastis, une fois rempli il déclenchera le pouvoir spécial
* Le portrait du joueur et son nom

Les éléments graphiques sont tous du SVG. Concernant le verre de Pastis, je donne un nom au chemin représentant l'intérieur du verre et lui associe un clipping :

```xml
<!-- En faisant varier y de 100 (verre vide) à 0 (verre plein), on monte le niveau de lique en faisant monter le cliping -->
<clipPath id="pastisClip">
   <rect x="0" :y="pastisLevel" width="100" height="158" style="fill: white; stroke-width: 0"/>
</clipPath>
```

L'explication est donnée, y'a plus qu'à faire varier la variable du composant Vue.js pour faire bouger le niveau de pasis.

J'instantie deux fois ce composant, bleu étant la couleur du joueur humain, rouge étant la couleur de l'adversaire :

```xml
  <PlayerIcon 
    x="100" 
    y="800"
    color="blue">
  </PlayerIcon>

  <PlayerIcon 
    x="1400" 
    y="800"
    color="red">
  </PlayerIcon>
```

Et voici le résultat. Je ferai évoluer plus tard le composant pour y ajouter le portrait du joueur et son nom, plus certains effets graphiques. Pour le moment, j'ai le minimum pour prototyper la deuxième phase du game play : les cartes.

![image]({{ site.url }}/assets/articles/make-something-horrible-part2/gameview_part1.png)

# Modélisation des cartes

L'atout (!) principal du jeu n'est pas encore modélisé : les cartes ! Voyez-vous, je ne suis pas un expert et j'ai probablement fait une erreur de débutant en tant que développeur de jeu vidéo. J'ai commencé par le menu principal alors que je pense que dans un studio réel, c'est le genre de partie qui est effectuée que bien plus tard.

Reste que développer le menu m'a permis de tester certaines technologies (SVG, l'intégration avec Vue.js, le son  ...). Bref, développer seul change un peu la stratégie de développement.

Bon, une fois de plus, sortons Inkscape. Je ne sais pas encore quelle sera la taille d'une carte, mais j'imagine que le joueur aura en gros 5 cartes en main en permanence. On peut par exemple avoir la séquence de jeu suivante :

1. Le joueur tire une carte de son deck (ah je viens de penser que l'on ne voit pas de deck sur le plateau ... c'est pas grave, on n'a pas le temps, la carte apparaîtra magiquement)
2. Il *doit* jouer une carte à destination :
   * du terrain de camping
   * de la poubelle
   * de lui
   * de l'adversaire

Donc, 5 cartes sur une largeur de ... 1920 moins deux blocs HUD de 400 de large plus les marges de chaque côté cela nous laisse environ 800 de large pour toutes les cartes soit 160 pixels pour une carte, si on ne les chevauche pas. Au niveau des proportions, on copie la proportion des cartes existantes, 65mm x 100mm.

On copie-colle le code SVG dans le template Vue.js, et on instancie 5 cartes en bas de l'écran.

```xml
   <Card x="550" y="800"></Card>
   <Card x="700" y="800"></Card>
   <Card x="850" y="800"></Card>
   <Card x="1000" y="800"></Card>
   <Card x="1150" y="800"></Card>
```

![image]({{ site.url }}/assets/articles/make-something-horrible-part2/gameview_part2.png)

Hou ! Les belles cartes !

# Le drag & drop

Bon, je gameplay central de notre jeu se base sur le drag et le drop des cartes. On va tester cela, pas besoin de nouvelle librairie, D3.js sait tout faire !

Je prends le code à partir d'un petit tutorial sur Internet, a priori il n'y a pas grand chose à faire. J'autorise tous les tags qui ont la class 'card' à être draggable.

```javascript
    let deltaX, deltaY;
    let dragHandler = d3.drag()
    .on("start", function () {
      let current = d3.select(this);
      deltaX = current.attr("x") - d3.event.x;
      deltaY = current.attr("y") - d3.event.y;
    })
    .on("drag", function () {
        d3.select(this)
            .attr("x", d3.event.x + deltaX)
            .attr("y", d3.event.y + deltaY);
    });

    dragHandler(d3.selectAll(".card"));
```

Trop simple, je peux maintenant bouger toutes les cartes sur le plateau de jeu :

![image]({{ site.url }}/assets/articles/make-something-horrible-part2/gameview_part3.png)

Et le 'drop' ? Nous verrons cela au prochain numéro !

# Conclusion

Voilà nous avons terminé pour cette partie. Evidemment, rien n'est totalement finalisé, mais nous avons tous les objets de base au niveau graphisme pour avancer dans le jeu de la carte proprement dit. Pour faire bien BD, nous ajouterons à la fin des bulles pour faire parler nos personnages et leur faire dire des bêtises. On laisse tout le rafinement pour la fin, ne pas oublier qu'un mois c'est court pour faire un jeu complet.

Au prochain numéro, nous nous attaquerons au code qui ne se voit pas : le moteur de jeu.  On fera donc du C++, du réseau, bref c'est la rigolade.
