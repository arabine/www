---
layout: post
title: "Make Something Horrible (3)"
date: 2019-08-03 8:55:00 -0120
comments: false
published: true
category:
- jam
- games
---

> Make Something Horrible est une Game Jam organisée régulièrement par le magazine papier (et Web maintenant) Canard PC. Le concours consiste à créer un jeu original, surtout drôle et forcément laid car créé par des gens sans talents graphiques. Moi compris ! Cette année j'ai décidé de participer en ayant comme objectif secondaire la découverte de certaines technologies et la mise en pratique d'architectures. Voici le troisième chapitre.

# Empaquetage et distribution

Pour le moment, nous n'avons que des pages web propulsées par Vue.js. Il faut quand même lancer un serveur pour que les pages s'affichent. Pour distribuer le jeu sous la forme d'un exécutable nous allons avoir besoin de fournir un tel serveur et un navigateur. Je pourrais utiliser Electron, qui est fait pour ça, mais il est très connoté Node.js. Moi, je suis avant tout un programmeur embarqué c/C++ donc mes back-ends je les fait en C moi môssieur.

Mes outils :

* Gulp (optionnel) qui permet de regrouper les fichiers Javascript et éventuellement de les compresser (uglify)
* Qt pour le navigateur et un peu tout ce qui concerne le système
* Et on zippe le tout hein,on va pas s'embêter avec un installeur sauf si je m'ennuie.

# Etape 1 : empaquetage du Web

On commence avec Gulp. Le but ici va être de générer un répertoire de livraison nommé "dist" qui va contenir tous les Javascript ; d'une part les librairies externes, celles-ci seront seulement copiées, et une librairie applicative contenant tous nos fichiers Vue.js amalgamés et compressés. On parle ici de compression au sens Javascript, c'est à dire que c'est une compression textuelle : les variables sont remplacées par des lettres uniques, les espaces sont supprimés etc. Le but sera de réduire la taille de téléchargement. Le serveur Web lui rajoutera une couche de compression binaire, Gzip.

Tout d'abord, on install gulp et les plug-ins utilisés par notre script :

```sh
npm install gulp -g
npm install
```

Puis, on invoke dans un stript nos tâches gulp :

```sh
gulp bmen-lib
gulp bmen-css
```

Deux fichiers vont être générés dans /dist : bmen.min.js et bmen.min.css. Maintenant, on va utiliser un autre script, en Perl cette fois, pour générer une arborescence de serveur de fichiers :

```sh
perl ./embed.pl dist images i18n fonts sounds index.html favicon.ico > src/embedded_files.c
```

Allez hop, on dump tous les fichiers et répertoires nécessaires à notre WebApp pour pouvoir fonctionner, ce qui génère un *GROS* fichier .c contenant des tableaux en "const char ...". La fin du fichier liste toute notre arborescence virtuelle, la taille et le type de chaque fichier. 

```c
static const struct embedded_file {
  const char *name;
  const char *mime;
  const unsigned char *data;
  size_t size;
} embedded_files[] = {
  {"/dist/bmen.min.js", "application/javascript", v0, sizeof(v0) - 1},
  {"/dist/style.min.css", "text/css", v1, sizeof(v1) - 1},
  {"/images/background.svg", "image/svg+xml", v2, sizeof(v2) - 1},
  {"/images/card.svg", "image/svg+xml", v3, sizeof(v3) - 1},
  {"/images/cover.png", "image/png", v4, sizeof(v4) - 1},
  {"/images/logo.png", "image/png", v5, sizeof(v5) - 1},
  // ...
};
```

Attention à bien séparer les répertoires contenant les librairies tierces, les sources brutes (images, sons) et le code source de votre site proprement dit.

# Etape 2 : Qt à la rescousse

Nous sommes prêts pour la seconde étape qui consistera à compiler tous ces fichiers C dans une application executable. On utilisera donc la librairie Qt qui dispose d'un composant QWebEngine : il s'agit du moteur de rendu Chromium embarqué à la sauce Qt, c'est-à-dire avec une API génial et simple d'utilisation ainsi que les passerelles nécessaires pour partager des données entre QWebEngine et le reste des classes Qt.

On ajoute au projet un serveur de fichier, notre fichier généré contenant un système de fichier "virtuel".

Le code QML est quant à lui assez léger, il se contente d'instancier une vue Web et de charger l'adresse locale du serveur :

```javascript
Window {
    width: 1024
    height: (width*9)/16
    visible: true

    WebEngineView {
        id: webView
        anchors.fill: parent
        url: "http://127.0.0.1:8081"
    }
}
```

Côté serveur en C++ : on démarre un serveur HTTP et on sert les pages web embarquées précédemment sous forme de fichier .c. Nous créons une classe appelée "BMen" qui sera le coeur de notre back-end.

```cpp
  BMen bmen;
  tcp::TcpServer tcpServer(bmen);

  if (bmen.Initialize())
  {
      if (tcpServer.Start(100, true, 8081, 8083))
      {
          bmen.Start();
      }
  }

  QQmlApplicationEngine engine;
  engine.load(QUrl(QStringLiteral("qrc:/main.qml")));
```

Voilà le résultat, nous avons une fenêtre seule contenant le navigateur et le code back-end, le tout dans un exécutable (avec ses DLL autour, ce qui n'est pas un détail avec Qt).

![image]({{ site.url }}/assets/articles/make-something-horrible-part3/stand_alone_qt.png)

Nous n'allons pas plus loin pour le moment, nous verrons par la suite comment, toujours avec Qt, facilement distribuer notre application.

# On continue le front-end : vive les drop

Nous pouvons effectuer un "drag", essayons maintenant de détecter un "drop" ; nous allons définir des zones sensibles sur lesquelles le joueur pourra interagir avec sa carte.

Nous allons développer un code générique : pour transformer n'importe quel composant SVG en zone "draggable", on va :

1. Le définir en tant que classe 'draggable'
2. Au démarrage, scanner tous les composants ayant cette classe, puis créer un rectangle SVG par dessus
3. Lorsque le drag est en cours, utiliser les coordonnées du curseur de la souris pour détecter si l'on est au dessus d'une telle zone
4. Si oui, alors on change une propriété de type 'data-xxxxx' du rectangle en question
5. Un CSS dédié à cette propriété permettra de mettre en surbrillance cette zone

Exemple pour la poubelle, que l'on définit en tant que draggable :

```html
<Trash
    x="200" 
    y="400"
    class="droppable"
  >
</Trash>
```

Le code au démarrage, création des carrés en surbrillance, avec un data-state vide par défaut :

```javascript
d3.selectAll(".droppable").each(function(d,i) {
  let rect = d3.select(this);
  d3.select('#mainsvg')
    .append('rect')
    .attr('x', rect.attr('x'))
    .attr('y', rect.attr('y'))
    .attr('width', rect.attr('width'))
    .attr('height', rect.attr('height'))
    .attr('class', 'drop-area')
    .attr('data-state', '');
});
```

Le code CSS correspondant, lorsque le curseur au dessus de la zone est détecté, on affiche un contour et un remplissage semi-transparent :

```css
.drop-area {
  fill: transparent;
  stroke-width:0;
}

.drop-area[data-state='ok'] {
  fill:rgba(255,255,255,0.5);
  stroke-width:10;
  stroke: green;
}
```

Maintenant, le code de détection : nous appelons une fonction classique de détection de collision à chaque fois que la souris est bougée :

```javascript
.on("drag", function () {
        d3.select(this)
            .attr("x", d3.event.x + deltaX)
            .attr("y", d3.event.y + deltaY);
        app.isSelected(d3.event.x, d3.event.y);
    })

isSelected: function(x, y) {

  d3.selectAll(".drop-area").each(function(d,i) {
    let rect = d3.select(this);
    let xmin = parseInt(rect.attr('x'));
    let ymin = parseInt(rect.attr('y'));
    let xmax = xmin + parseInt(rect.attr('width'));
    let ymax = ymin + parseInt(rect.attr('height'));
    
    if ((x >= xmin) && (x < xmax) && (y >= ymin) && (y < ymax)) {
      console.log("Detected !!");
      d3.select(this).attr('data-state', 'ok');
    } else {
      d3.select(this).attr('data-state', '');
    } 

  });

}
```

Et voilà notre super détection de zone en action :

![image]({{ site.url }}/assets/articles/make-something-horrible-part3/drop_zone.png)

C'est tout pour cette fois-ci, au prochain numéro !
