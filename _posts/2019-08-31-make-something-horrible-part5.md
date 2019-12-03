---
layout: post
title: "Make Something Horrible (5)"
date: 2019-08-23 8:55:00 -0120
comments: false
published: true
category:
- jam
- games
---

> Make Something Horrible est une Game Jam organisée régulièrement par le magazine papier (et Web maintenant) Canard PC. Le concours consiste à créer un jeu original, surtout drôle et forcément laid car créé par des gens sans talents graphiques. Moi compris ! Cette année j'ai décidé de participer en ayant comme objectif secondaire la découverte de certaines technologies et la mise en pratique d'architectures. Voici le cinquième et dernier chapitre.

# Finitions et ajustements

La première livraison en "démo" d'un jeu est importante : vous peaufinez vos scripts de livraison et surtout cela vous permet d'avoir des premiers retours utilisateurs. Pour ma part, j'en ai eu plusieurs par le biais du forum de Canard PC, dans la section dédiée à la Game Jam. Très bien, il me reste une semaine, avec ces remarques et ma TODO list du précédent article j'ai de quoi remplir mes soirées.

# Pouvoir spécial

Je savais qu'il manquait une chose essentielle : le pouvoir spécial de chaque joueur. Voici ce que j'ai ajouté : avant de commencer une partie, le joueur choisit son pouvoir spécial parmi trois :

* Tempête sur le Camping : effacez des emplacements adverses
* Mal de tête : bloquez le joueur adverse pendant plusieurs tours
* Pétanque Master : videz la Bibine adverse

![image]({{ site.url }}/assets/articles/make-something-horrible-part5/choix_pouvoir.png)

Le développement est assez simple côté serveur. 

# Effets visuels et sonores

Maintenant que l'architecture est stable, je peux ajouter des effets pour rendre le jeu plus sympatique. Je souhaitais ajouter des effets graphiques. Je pourrais le faire en SVG, mais je sais que les jeux HTML5 sont plus souvent réalisés en utilisant la balise 'canvas'. Donc, j'aurais plus de chance de trouver du code source d'exemple d'effets graphiques.

J'ajoute donc une couche au sandwich HTML avec une balise canvas prennant tout l'écran.

![image]({{ site.url }}/assets/articles/make-something-horrible-part5/layers.png)

Comme d'habitude, je crée un composant pour cela ce qui permet de découpler un peu les choses. Le premier effet que je mets est la puie dans le cas de l'enclenchement du pouvoir "Tempête". Je récupère du code (libre) sur Internet et l'effet est lancé en parallèle avec un effet sonore de tempête. C'est l'effet sonore qui arrêtra l'effet graphique, toujours à l'aide de la librairie Howler.js :

```javascript
if (action.effects[e] == 'rain') {
    this.$refs.canvasLayer.startRain();
    new Howl({
        src: ['sounds/rain.webm'],
        autoplay: true,
        onend : () => {
        this.$refs.canvasLayer.stopRain();
        }
    });
}
```

Efficace et simple. Pour information, c'est le serveur qui envoie l'ordre d'effet dans un tableau d'événements liés aux actions effectuées par la carte envoyée.

# Ajustements de GamePlay

Pour rendre la jouerie plus sympa et moins longue, j'effectue les modifications suivantes :

* Limite des emplacements à 5 points
* L'IA choisit sa défense aléatoirement
* Stratégie IA aléatoire (choix stratégiques)

Ainsi, les emplacements sont plus facilement prenables et moins défendable, pour le coup. Egalement, cela permettra de créer des cartes plus fortes, genre une attaque à 5 points, mais plus rares !

La modification fondamentale de GamePlay a été de complexifier la carte du Camping. En effet, en l'état, il n'y a pas de dimension stratégique : on conquiert les emplacements où l'on veut.

Je décide donc d'autoriser la pose des cartes uniquement sur des cartes adjacentes (horizontales ou verticales) à d'autres cartes du même camp (sauf le premier coup, bien entendu !!).


# Evolutions graphiques

Attaquons nous maintenant à quelques ajouts graphiques finaux. Dans un premier temps, j'améliore la Police de caractères utilisée pour les nombres du Camping ; plus grosse et plus explicite, surtout entre le chiffre 1 et 7. Merci le forum Canard PC ! Nous avons maintenant par exemple le portrait des joueurs.

De nouvelles cartes sont développées à l'arrache pour rendre le jeu plus sympa.

# Conclusion

Le compte à rebours est terminé, il est temps de livrer la version finale du jeu, non sans défauts. Il n'est pas très sympa à jouer à vrai dire, notamment parcequ'il est compliqué de le terminer. Bon, j'ai tout de même eu du plaisir à le développer, j'ai pu tester la technologie Web avec satisfaction pour créer un jeu en "local". Bonne chance à tous pour cette GameJam !

