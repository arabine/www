---
layout: post
title: "Make Something Horrible (1)"
date: 2019-07-30 8:55:00 -0120
comments: false
published: true
category:
- jam
- games
---

> Make Something Horrible est une Game Jam organisée régulièrement par le magazine papier (et Web maintenant) Canard PC. Le concours consiste à créer un jeu original, surtout drôle et forcément laid car créé par des gens sans talents graphiques. Moi compris ! Cette année j'ai décidé de participer en ayant comme objectif secondaire la découverte de certaines technologies et la mise en pratique d'architectures.

# Outillage et technologies utilisées

Bien que je sois développeur embarqué à l'origine, donc programmeur C/C++, j'ai toujours utilisé les technologies Web pour plusieurs raisons :

  1. Certains produits embarqués disposent d'un petit serveur Web permettant de configurer le produit
  2. La mode des objets connectés demande de plus en plus d'interopérabilité avec les technologies Internet (TCP/UDP/IP...)
  3. Les technologies Web faisant travailler le client web (framework Front-End) sont parfait pour les microcontrôleurs aux faibles ressources : le dialogue entre le front-end et le back-end ne véhicule donc que des données brutes.

Donc oui, notre jeu sera basé sur des technologies Web, du moins en partie. Pourquoi ? Tout simplement parce que c'est une solution totalement portable. Dès lors, je me suis tourné vers les choix technologies suivantes :

* Vue.js comme bibliothèque front-end : je suis fan, notamment de par son architecture simple et évolutive
* D3.js : la plupart du rendu sera réalisé en SVG, à par les dessins
* Qt : pour la version desktop, j'utiliserai Qt pour embarquer Chromium et le jeu
* C/C++/WebSocket : le back-end sera en C++, le dialogue avec le front se fera soit par WebSocket soit par API REST

Maintenant, on fait un jeu. Donc il va falloir à un moment créer des illustrations. J'ai toujours bien aimé dessiner, mais je n'ai pas de tablette graphique. En fouillant un peu dans mon pot à stylos, j'en ai trouvé un qui possède un embout mou compatible écran tactile. Il n'est pas d'une précision d'enfer mais pour notre usage, ça ira.

![image]({{ site.url }}/assets/articles/make-something-horrible-part1/tablette_du_pauvre.jpg)

Concernant l'application Android, j'en ai trouvé une géniale, il s'agit d'Autodesk SketchBook, gratuite. Vu que je n'ai pas de talent, j'utilise la méthode suivante :

  1. Recherchez une image sur Internet de l'objet que vous voulez dessiner
  2. Mettez l'image sur une couche dédiée
  3. Désactivez la couche "background", l'arrière-plan est maintenant transparent
  4. Sur une nouvelle couche dédiée, suivez les contours de l'image, changez le type de crayon en fonction du besoin

Voilà ! Au niveau logiciels PC : Paint.net pour les retouches, Visual Studio Code pour l'éditeur, Qt pour le C++ et le back-end et enfin Inkscape pour le SVG.

Python peut être utile, notamment au début lorsque le back-end n'existe pas : un petit :

```python
python3 -m http.server
```

Et votre logiciel "web" prend vie.

# Quel jeu ?

Abordons maintenant la partie game-play. Il faut trouver un type de jeu (FPS, JDR, stratégie, board game ...) et un thème. Cette année, la game jam impose que le jeu soit tiré d'une oeuvre quelconque. Au moment de la découverte de cette Game Jam, j'étais en train de regarder un film de X-Men. Bingo, je vais utiliser le thème des super-héro, mais en version plus loufoque. Des super héros bien franchouillards ! L'avantage est que je vais pouvoir utilser des images du quotidien pour illustrer mon jeu, c'est plus facile que de la SF ... surtout quand on n'est pas graphiste comme moi.

Maintenant, il faut choisir le style du jeu. Vu les technologies que je me suis imposées, il va falloir rester en 2D. Dans le passé, j'avais fait un jeu de Tarot, j'avais bien aimé et j'avais plein d'idées de variantes stupides à base du jeu. J'ai également un passé de joueur de cartes à collectionner de type Magic. Bref, voici donc notre thème : un jeu de carte à deux joueurs, chacun incarne non pas un sorcier mais un B.E.A.U.F qui détient le gène B octroyant au porteur une, et une seule capacité particulière.

![image]({{ site.url }}/assets/articles/make-something-horrible-part1/beauf.jpg)

# Game design

Nous allons faire un truc simple ; 4 écrans de jeu découpés ainsi :

1. L'écran du menu d'accueil, un titre, sous-titre, une image et un menu
2. Une page "à propos", parce qu'il faut toujours indiquer qui est l'auteur de l'oeuvre
3. Une page "histoire", qui introduit brièvement le scénario (oui car je ne sais pas faire d'animation 3D hein, sinon bien sûr je l'aurais fait, hum.)
4. Et enfin, le jeu lui même.

L'écran de jeu sera assez sommaire car on n'a pas trop le temps de le peaufiner, et beaucoup de temps sera consacré au moteur de jeu, au design des cartes et à leurs effets.

Il y aura probablement une zone centrale où seront posées les cartes. Le joueur verra ses cartes, en main, étalées façon Belote en éventail (plus ou moins). Pas la peine de montrer les cartes de l'adversaire, il faut simplifier le codage.

Chaque joueur sera symbolisé par un portrait grossier, on y associera une jauge de vie ainsi que le fameux pouvoir spécial (genre barre de mana qui se charge). Restons simples pour une game jam !

Le joueur humain aura toujours le même portrait, le nom du joueur pourra être choisi via une fenêtre popup avant de lancer la partie. On essaiera de dessiner au moins deux adversaires différents avec des noms différents.

# Style graphique

Alors que la programmation générale ne me pose pas de problème, la programmation graphique est toujours un peu délicate pour moi. Je ne parviens jamais à être totalement satisfait du résultat. Je n'ai pas d'idée particulière sur le style : la seule chose est de passer le moins de temps possible sur la conception des dessins. Donc, pas de couleur, que du crayonné pour les images brutes, le noir et blanc sera de mise.

Pour les éléments géométriques, j'aime bien le style destructuré, genre des polygones un peu tordus. Je pense que ça fait bien super héro. Pour la police de caractères originale, je vais sur un site de fontes quelconque et je tape "super hero" ou un truc dans ce genre. Il y a mille réponses, reste plus qu'à faire son choix. Voici celle que j'ai utilisé, pas parfaite car certains caractères français ne sont pas gérés. 

![image]({{ site.url }}/assets/articles/make-something-horrible-part1/font_badaboom.png)

# C'est parti ! Montons les fondations

Je commence par un squelette classique pour toute application basée sur Vue.js. Un point central, app.js, contenant la structure générale de la page, un plug-in de routage, un autre pour le store (la centralisation des données avec détection des changements).

Voici larborescence du début du projet. Il y a des fichiers qui serviront "au cas où" je réalise des évolutions, par exemple le fichier de traductions i18n. Pour la GameJam, tout sera en français.

Les quatres vues décrites plus haut seront matérialisées par les fichiers dans le répertoire 'Views'. Le répertoire 'Components' contiendra des composants Vue.js réutilisables. Il ne faut pas hésiter à faire des composants avec Vue.js : cela découple votre projet et simplifie les vues.

Le reste est du grand classique : des fontes (car le logiciel pourra fonctionner off-line), des librairies JavaScript tierces (popup, temps, sécurité, combobox avancé ...) toutes ne serviront pas mais elles font parties de ma bibliothèque habituelle.

Le code CSS dédié au jeu sera quant à lui stocké entièrement dans le fichier style.css.

```cmd
│   app.js
│   i18n.js
│   index.html
│   LICENSE
│   README.md
│   store.js
│
├───components
│       MenuItem.js
│
├───css
│       fontawesome.min.css
│       regular.min.css
│       solid.min.css
│       style.css
│       tingle.min.css
│
├───fonts
│       badaboom.ttf
│       fa-regular-400.woff2
│       fa-solid-900.woff2
│       Roboto-Black.ttf
│       Roboto-BlackItalic.ttf
│       Roboto-Bold.ttf
│       Roboto-BoldItalic.ttf
│       Roboto-Light.ttf
│       Roboto-LightItalic.ttf
│       Roboto-Medium.ttf
│       Roboto-MediumItalic.ttf
│       Roboto-Regular.ttf
│       Roboto-RegularItalic.ttf
│       Roboto-Thin.ttf
│       Roboto-ThinItalic.ttf
│
├───i18n
│       i18n.json
│
├───images
│       cover.png
│       menu_1.svg
│       menu_2.svg
│       menu_3.svg
│
├───js
│       d3.js
│       d3.v5.min.js
│       moment-with-locales.min.js
│       reconnecting-websocket.min.js
│       selectr.min.js
│       sjcl.js
│       tingle.min.css
│       tingle.min.js
│       ulog.min.js
│       vue-i18n.js
│       vue-i18n.min.js
│       vue-router.js
│       vue-router.min.js
│       vue.js
│       vue.min.js
│       vuex.js
│       vuex.min.js
│
└───views
        GameView.js
        MenuView.js
```

Un conseil : créez dans un premier temps des vues vides pour vous aider à monter toute l'arborescence sans voir d'erreurs dans la console. C'est souvent rébarbatif car il est facile de se planter dans les chemins relatifs.

# Le menu d'accueil : structure et titre

Comme tout bon jeu qui se respecte, nous allons proposer un joli menu pour guider le joueur, expliquer les touches et les règles. Cela peut paraitre simple mais on va déjà utiliser toute la chaîne de production logicielle. C'est un bon entraînement avant d'attaquer le jeu, l'élément central !

On commence par structurer la page. Nous voulons tout faire en SVG, donc notre conteneur principal sera un tag `<svg>` classique avec un système de coordonnées : nous faisons un jeu, il va donc nous falloir placer nos divers éléments à des endroits fixes pour que le rendu soit toujours identique. Pour cela, nous utilisons la propriété viewBox du SVG, ici nous avons une zone de 1920x1080 "pixels" pour dessiner. L'unité n'est pas vraiment du pixel car le SVG est vectoriel, mais vous comprenez l'idée.

```html
<svg id="mainsvg"  viewBox="0 0 1920 1080">
  <defs>
  </defs>

  <rect x="0" y="0" width="1920" height="1080"  style="fill: transparent; stroke: black; stroke-width: 3"/>
</svg>
```

On dessine un petit rectangle tout autour pour bien montrer les limites et voir tout de suite si on dépasse. On verra si on le retire dans la version finale.

Quant au titre, c'est tout simple, le SVG nous offre un tag pour cela.

```html
<!-- Title -->
<text x="660" y="150"
    font-family="Badaboom"
    font-size="150">
    The B-Men
</text>

<text x="660" y="250"
    font-size="60">
    Le commencement
</text>
```

Au niveau de Vue.js proprement dit, notre routeur n'a pour le moment qu'une seule route, la page d'accueil, on affiche notre composant.

```javascript
const router = new VueRouter({
  mode: 'hash',
  routes: [
    { path: '/', name: 'menu', meta: { }, component: MenuView   },
    { path: '*', redirect: '/'}
  ]
});
```

On fixe une couleur unie pour le fond. Voilà le résultat après une bonne demi-heure de tests :

![image]({{ site.url }}/assets/articles/make-something-horrible-part1/menu_part1.png)

# Le menu d'accueil : les éléments du menu

On va maintenant afficher un joli menu digne d'un jeu vidéo. On va commencer à donner une petite personnalité à notre jeu. Nous avons trois entrées chacune est constituée d'un texte et d'un polygone en arrière plan. Pour parer au future et simplifier le code du fichier MenuView.js, nous allons créer un composant ré-utilisable représentant une entrée du menu.

Le template d'un menu est simple : un groupe constitué de l'image de fond (un fichier au préalablement SVG chargé dans la zone 'def' du SVG) et un texte. On lie certaines propriétés vers les données du composants passées en paramètres comme la position X et Y.

```javascript
let menu_item_template = /*template*/`
<g @clicked="clicked()" style="cursor: pointer; pointer-events: fill;" :id="id">
    <use :href="getRef()" :x="x" :y="y" style="color: transparent;"></use>
    <text :x="x+160" :y="y+80"
        font-family="Badaboom"
        :font-size="fontSize"
        class="menuTitle">
        {{title}}
  </text>
</g>
`;
```

Notez la propriété 'pointer-events: fill;' qui permet d'élargir à toute la surface du groupe la zone d'événements du pointeur de souris. Sans cela, les "trous" graphiques se seront pas pris en compte.

On applique deux effets : on colorise le poligone et on agrandit le texte du menu lorsque le curseur passe dessus. Facile à faire avec D3.js : 

```javascript
let el = d3.selectAll('#' + this.id);
el.on("mouseenter", function(d){
    d3.select(this).select('text').transition()
    .duration(200)
    .attr("font-size", "100");

    d3.select(this).select('use').style("color", "grey");
});
```

On modifie donc l'attribut 'font-size' lorsque la souris entre dans la surface du groupe. Un astuce SVG concernant le polygone en tâche de fond : on ajoute une variable CSS appellée currentColor comme couleur de remplissage. Ainsi, en modifiant la propriété 'color' dans notre usage (voir la balise '<use>' ci-dessus), on peut modifier la couleur de remplissage en basculant entre 'grey' et 'transparent'.

```html
<path
    style="stroke:#000000;stroke-width:0.38241136;stroke-linecap:butt;stroke-linejoin:miter;stroke-miterlimit:4;stroke-dasharray:none;stroke-opacity:1"
    d="m -403.50044,122.81045 158.26532,-6.1056 -48.70676,12.3443 -92.86124,8.43613 z"
    id="menu_bg"
    fill="currentColor"
    inkscape:connector-curvature="0"
    sodipodi:nodetypes="ccccc"
/>
```

Enfin, l'effet inverse est également ajouté lorsque la souris sort de la surface du groupe. Voici ce que donne le menu après cette étape :

![image]({{ site.url }}/assets/articles/make-something-horrible-part1/menu_part2.png)

# Le menu d'accueil : l'illustration

Ici c'ets simple, on ajoute l'image grâce à la balise <image> du SVG.

```html
<!-- L'image de couverture -->
<svg x="50" y="400">       
    <image xlink:href="images/cover.png" width="1000" height="603" />
</svg>
```

L'image est au format PNG car on souhaite avoir une transparence de fond. Le crayonné a été réalisé à partir de la photo d'une caravane des années 60 au look terrible. La méthode employée est celle décrite en introduction.

![image]({{ site.url }}/assets/articles/make-something-horrible-part1/tiny_caravan.jpg)

Voici le résultat final :

![image]({{ site.url }}/assets/articles/make-something-horrible-part1/menu_part3.png)

# Rafinements finaux

Quelques rafinements sont nécessaires pour parfaire l'expérience utilisateur :

Créer un "loader", un cercle qui tourne à l'infini le temps que le composant se charge ; cela évite de voir apparaître les éléments les uns après les autres. Pour cela on utilise une variable du composant de tyle booléen et le système de template réactif de Vue.js fait le reste. Le rond qui tourne est fait entièrement en CSS.

```html
<div v-show="!loaded" class="loader"></div>

<svg v-show="loaded" id="mainsvg"  viewBox="0 0 1920 1080">
```

Il faut interdire la sélection des textes sur toute la page :

```css
html, body {
    -webkit-user-select: none;
    -khtml-user-select: none;
    -moz-user-select: -moz-none;
    -o-user-select: none;
    user-select: none;
}
```

Enfin, il faut bloquer le drag des images qui affiche une vilaine image fantôme ; un peu de Javascript est ajouté pour chaque image : 

```html
<image xlink:href="images/cover.png" width="1000" height="603" onmousedown="if (event.preventDefault) event.preventDefault()"/>
```

C'est terminé pour cette partie. La prochaine vous expliquera le début de la construction du jeu en lui-même.

