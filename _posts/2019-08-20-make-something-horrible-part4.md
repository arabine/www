---
layout: post
title: "Make Something Horrible (4)"
date: 2019-08-23 8:55:00 -0120
comments: false
published: true
category:
- jam
- games
---

> Make Something Horrible est une Game Jam organisée régulièrement par le magazine papier (et Web maintenant) Canard PC. Le concours consiste à créer un jeu original, surtout drôle et forcément laid car créé par des gens sans talents graphiques. Moi compris ! Cette année j'ai décidé de participer en ayant comme objectif secondaire la découverte de certaines technologies et la mise en pratique d'architectures. Voici le quatrième chapitre.

# Galères et bugs

Alors dans les chapitres précédents, j'expliquais que je voulais développer le back-end en C++. Malheureusement, j'ai eu des gros soucis de programmation Socket sous Windows (alors que sous Linux ça fonctionne au poil). Bon, pour ne pas perdre trop de temps dans le développement, je suis passé à Electron.js : le back-end se fera donc sous Node.js.

J'ai développé un serveur HTTP vite-fait bien fait, sans utiliser de framework pour éviter les dépendances, uniquement chargé de répondre aux requêtes sur une API REST.

# On construit les cartes

Attaquons nous maintenant aux cartes. Côté serveur, je dispose d'un fichier JSON flexible listant toutes les cartes du jeu. Voici l'exemple du carte, modélisée par les paramètres suivants :

```json
{
    "title": "Tente",
    "category": "defense",
    "picture": "tent",
    "text": "Planter sa tente marque le territoire du Campeur. You shall not pass, comme dirait l'autre.",
    "value": 1
}
```

Là, on entre dans le coeur du Game Play : forcément, les paramètres ont un impact sur les règles du jeu, pas encore totalement définies à ce moment là du développement. J'ai décidé de rester simple :

* On définit trois catégories : défense (à jouer sur le plateau), attaque (à jouer contre l'adversaire ou le plateau) et bibine (pour remplir son niveau d'énergie)
* Une carte a une valeur, un chiffre dont la signification dépendra de la catégorie
* Le reste c'est du graphisme : un titre, un texte et une illustration

Normalement, je ne devrais pas passer beaucoup de temps dans le moteur de jeu, mais je ne sais absolument pas ce que donnera le jeu en terme de "jouerie" : facile ? difficile ? Ennuyant ? C'est ma première GameJame, je vous avoue que je n'avais pas anticipé cette crainte, de devoir attendre très tardivement dans le développement du jeu pour ... jouer !

Au niveau du front-end, je dispose déjà d'un composant Card.js bien séparé (règle d'or : dans le doute, toujours créer un composant dédié !!). La vue générale passera l'objet JSON regroupant les propriété d'une carte à chaque carte affichée. Le composant Card.js se chargera de gérer l'affichage au bon endroit !

![image]({{ site.url }}/assets/articles/make-something-horrible-part4/cards.png)

# On construit la grille

Attaquons nous maintenant à l'autre élément central qui n'existe toujours pas : la grille repréentant le Camping. Normalement, cela devrait aller vite car j'ai déjà passé du temps sur le moteur de 'drop' générique. Je vais donc définir N rectangles.

Astuce : commencez par dessiner un rectangle en affichant ses contours, puis positionnez le au pif, ajustez les tailles et les positions. Ensuite, constuisez dans la fonction 'created()' du composant GameView.js la matrice en mémoire :

```javascript
// Create the grid
for (let i = 0 ; i < 9; i++) {
    for (let j = 0 ; j < 3; j++) {
    this.grid.push( {
        state: 0,
        camp: "transparent",
        x: 765 + i*123,
        y: 190 + j*175
    });
    }
}
```

Pour chaque case, on maintient un statut (la valeur de défense de l'emplacement) et à qui appartient l'emplacement. Ici, astuce, on va utiliser cette propriété pour afficher la couleur du rectangle :

* transparent : l'emplacement est à personne
* blue : l'emplacement est au joueur humain
* red: l'emplacement est au joueur adverse (ordinateur)

L'affichage de la grille est réalisée en utilisant le système de template efficace de Vue.js :

```html
  <template v-for="g in grid">
    <rect ry="10" :x="g.x" :y="g.y" width="110" height="155" v-bind:style="{ fill: g.camp }" class="droppable"/>
    <text :x="g.x + 50" :y="g.y + 70" 
        font-family="Badaboom" 
        font-size="40">
        {{g.state}}
    </text>
  </template>
```

Bingo c'est terminé ! Chaque carré aura une classe "droppable" comme vu au chapitre précédent.

![image]({{ site.url }}/assets/articles/make-something-horrible-part4/grid.png)

# On filtre les drop

C'est bien beau, mais pour le moment tous les zones de drop s'affichent. On va filter selon le type de carte sélectonnée. Pour cela, nous allons utiliser la propriété 'data-xxx' de chaque zone et la comparer avec le type de carte.

Nous modifions la fonction de sélection de zone en ajoutant l'index de la carte en cours de drag :

```javascript
app.isSelected(d3.event.x, d3.event.y, parseInt(c.attr('index')));
//...
// On vérifie si la carte peut être jouée ici:
if (type == app.cards[card_id].category) {
    accept_drop = true;
} else if (type == 'trash') {
    accept_drop = true;
}
```

Et plus loin on teste donc le nom de cette zone par rapport à la propriété de la carte (le deck du joueur étant toujours accessible dans les data() du composant Vue.js). On accèpte toutes les cartes dans la poubelle du Camping, bien évidemment.
Notre belle fonction se termine par retourner non seulement ce statut de drop autorisé ou non mais aussi la destination choisie par le joueur : joueur, grille, poubelle ou adversaire.


# Côté serveur, on initialise le jeu et le protocole

Nous allons maintenant commencer à déclarer toutes les variables de notre jeu côté serveur : la grille, le niveau de Pastis de chaque joueur, les cartes de chaque joueur.

Côté protocole réseau, on reste dans le simple :

![image]({{ site.url }}/assets/articles/make-something-horrible-part4/protocol.png)

Pour chaque carte jouée par le joueur humain, on met à jour les différentes variables, on fait jouer l'IA et on renvoit tout le status du jeu : la grille, les cartes et les niveaux de Pastis.

Le front-end, à la réception, copiera ces statuts en local (section data()) et grâce au système réactif de Vue.js on n'a rien à faire, le framework va répercuter graphiquement tous les changements.

On s'évitera également côté serveur tout un tas de vérifications déjà bloquées par le front-end, on va rester simple et efficace pour cette Game Jam ; évidemment, dans le cadre pro ne faites pas ça ...

Regardoons ce qu'il se passe dans le cas où l'on joue une carte dans la poubelle :

```javascript
// On joue cette carte (pas de vérification côté serveur on est en Game Jam :)
if (destination == 'trash') {
    // on vire cette carte et on en tire une autre
    player_cards.splice(index, 1);
    player_cards.push(cards[getRandomInt(cards.length)]);
}
```

Et hop on revoie tout au client :

```javascript
// On renvoit un objet contectant tout le statut du jeu que le front-end mettra à jour graphiquement
res.writeHead(200, {'Content-Type': 'application/json'});
res.end(JSON.stringify({ grid: grid, cards: player_cards }));
```

Là par exemple, on ne gère toujours pas le niveau de Pastis :( 

# On met à jour la grille

On continue sur notre belle lancée. Maintenant, si le joueur joue sur la grille on va la mettre à jour. La règle est la suivante :

* Une carte défense jouée sur un emplacement vide : ok, on le prend
* Une carte défense jouée sur un emplacement à nous : la valeur de l'emplacement augmente d'autant
* Une carte attaque jouée sur un emplacement adverse : la valeur de l'emplacement réduit d'autant
* Une carte attaque n'est valide que sur un emplacement pris par l'adversaire (le rouge dans notre cas)

On effectue une vérification locale pour filtrer les zones de drop acceptées ou non, puis lorsque la carte est lâchée (fin du drop), on l'envoie au serveur.

Côté serveur, on effectue la même vérification, on s'arrange et on anticipe : notre fonction de vérification de la carte jouée aura en paramètre le camp qui joue : camp rouge (joueur adverse, l'ordinateur) ou bleu (joueur humain). On réutilisera cette fonction lors du développement de notre IA.

Notre fonction de dialogue avec le serveur est assez simple : on utilise une Promesse, on envoie la carte jouée et on reçoit une mise à jour de tous les éléments du jeu que l'on va recopier en local. Vue.js fait le reste et met tout à jour graphiquement.

```javascript
dropCard(dest, card_idx) {
    // On envoie au serveur notre carte jouée, en retour on reçoit la conséquence et le jeu de l'adversaire
    Api.sendCard({card_idx: card_idx, dest: dest }).then((action) => {
        console.log("Received: " + JSON.stringify(action));

        // Update the grid
        for (let i = 0 ; i < 9*3; i++) {
            this.grid[i].camp = action.grid[i].camp;
            this.grid[i].state = action.grid[i].state;
        }

        // Update the player's cards
        this.cards = action.cards;
    });
},
```

Voilà le résultat pour l'équipe Bleue, on commence à avoir du Game Play !!

![image]({{ site.url }}/assets/articles/make-something-horrible-part4/gameplay.png)


# On met à jour le niveau de Pastis

Là c'est tout simple, on a déjà l'aspect graphique de développé. Il suffit de communiquer avec notre composant enfant et lui envoyer la nouvelle valeur du niveau de Pastis :

Parent :
```javascript
this.$refs.bibineBlue.updatePastisLevel(action.bibine); // le joueur humain, le bleu
this.$refs.bibineRed.updatePastisLevel(action.opponent); // le joueur ordi, le rouge
```

Enfant :

```javascript
updatePastisLevel(newLevel) {
    // newLevel entre 0 et 10
    newLevel = newLevel * 10;
    this.pastisLevel = 100 - newLevel; // ça fonctionne à l'envers
},
```

# On fait jouer l'adversaire

Concernant l'adversaire, on va se contenter d'une seule IA pour le moment, une IA très agressive :

* On scanne nos cartes en main
* Si le niveau de Pastis de l'adversaire est non nul et que l'on peut l'attaquer, on le fait
* Sinon on attque un emplacement, si on a de quoi

Si rien de tout cela fonctionne :

* On remplit notre bibine
* Sinon on ajoute un emplacement aléatoire (en privilégiant le nombre)

Et si rien ne va, on envoie une carte à la poubelle.

Si j'ai le temps, j'améliorerai l'IA avec quelques subtilités :)

# Condition de victoire

Pour tester la condition de victoire, je fais mal jouer l'IA : elle se contentera de jeter la première carte venue à la poubelle ce qui me laissera le champs libre pour gagner et tester la victoire.

Lorsque toutes les cases de la grille sont remplient d'une couleur, le jeu s'arrête et on affiche une popup de victoire ou de défaite.

L'algorithme est simple, on écrit une fonction qui teste si le camp passé en paramètre est gagnant, que l'on appelle après que chaque joueur ait joué :

```javascript
function hasWon(camp) {
  let won = false;

  let count = 0;

  for (let i = 0 ; i < grid.length; i++) {
    if (grid[i].camp == camp) {
      count++;
    }
  }

  if (count >= (9*3)) {
    won = true;
  }

  return won;
}

// ...
playCard(action, 'blue');

if (hasWon('blue')) {
    game_result = 'victory';
} else {
    // On fait jouer l'ordinateur
    playComputerIA();
}

if (hasWon('red')) {
    game_result = 'lost';
}
```

# Conclusion et reste à faire

Nous voilà dans la dernière ligne droite du concours. Il reste une semaine avant la dead-line. Vu que cette version est jouable, je décide de la diffuser en beta-test sur itch.io. Au pire, si tout se passe mal, c'est cette version qui sera jugée. Si je parviens à faire mieux, alors tant mieux !

Ce qu'il reste à faire d'ici une semaine :

* Déclencher le coup spécial
* Ajouter du du son (son du coup spécial, musique d'ambiance, son poubelle)
* Ajouter les portrait des joueurs
* Faire quelques cartes supplémentaires

S'il me reste du temps :

* Faire parler les joueurs avec une bulle style BD
* Encore plus de carte
* Des effets sur les cartes lors des drops
* Choix du coup spécial au démarrage de la partie
* Améliorer la Popup de Victoire avec une image sympa :)
