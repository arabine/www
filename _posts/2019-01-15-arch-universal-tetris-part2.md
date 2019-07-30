---
layout: post
title: "Architecture par composant - Tetris générique (2)"
date: 2019-01-15 8:55:00 -0120
comments: false
published: false
category:
- architecture
- embedded
- games
---

> Nous avons expliqué dans notre premier article le but de notre travail et l'architecture générique par composants que nous allons mettre en place en mettant en œuvre le jeu Tetris sur plusieurs plateformes. Continuons l'aventure avec une première implémentation simple.

# Définitions : composant, entité, module, système ?

Aïe, on attaque quelque chose de sacré, selon moi chacun peut définir les concours de tel ou tel mot en fonction de sa vision et son besoin. Il y a cependant des modes ou des nouvelles techniques liées à l'évolution du matériel.

Quelques nouvelles méthodes de design ont émergées ces dernières années : ECS (Entity Component System), DOD (Data oriented Design). A priori, elles sont apparues dans le domaine du jeu vidéo pour résoudre des problèmes de performances liées à la gestion mémoire de milliers d'objet de façon efficace (sur les consoles limitées et éviter des ennuis de mémoire cache). Ces deux méthodes sont un peu liées, voire complémentaires. Dans tous les cas, essayez d'oublier le sacro-saint héritage et design objet appris lors de vos études. Il est utile, mais pas toujours. L'architecture logicielle répond surtout à des contraintes, il faut savoir y répondre en utilisant la méthode qui convient le mieux sans être dogmatique. Si on fait le parallèle avec les design patterns du C++, on peut dire que la mode est à la composition plutôt qu'à l'héritage, l'idée générale est de regrouper les données au même endroit en mémoire.

Voici ce que l'on peut dire sur ces nommages :
  * Un composant peut être vu comme un composant électronique : il a des broches définissant son interface ; c'est avec ces broches que nous allons communiquer avec lui (des fonctions). Il est multipliable à volonté et sera utilisé par des entités concrètes
  * Une entité représente quelque chose de réel dans le monde, elle sera composée de composants (la boucle est bouclée)
  * Un module = un composant
  * Un système regroupera tout cela et se chargera de la partie dynamique

Notre Tetris sera porté sur plusieurs plateformes ; dès lors, nous aurons à créer un système différent à chaque fois, avec ses propres contraintes, ses spécificités. Ce système créera des entités en ré-utilisant nos modules (composants) logiciels génériques.

# Bonnes pratiques en C

Il existe quelques règles simples pour avoir un code le plus modulaire possible en langage C. On peut par exemple copier un peu le modèle objet : un design pattern que j'utilise fréquemment est de développer un module logiciel de la façon suivante :
  * Un module possède plusieurs variables, publiques (peut être utilisées par d'autres modules) ou privées (variables utilisées uniquement pour le bon fonctionnement du module en lui-même)
  * On regroupe ces variables dans une structure de contexte. Cette structure sera passée par référence (pointeur) à toutes les fonctions du module qui en auront besoin
  * C'est le code appelant qui aura la charge d'instancier cette structure de contexte et de la passer aux fonctions du module pour traitement

Et pour bien ranger votre code :
  * Toutes les définitions propres au modules sont dans les .c, un header n'a pas à stocker forcément toutes les structures de la terre. N'oubliez pas, un en-tête est une *interface* pour l'extérieur. Rangez votre code.
  * Utilisez le mot clé static dans vos fichiers C pour les fonctions privées
  * Evitez le mot clé extern comme la peste, de toute façon si votre code est bien rangé à l'aide de 'static' impossible à un gugusse extérieur à votre composant de faire appel à une de vos fonctions privées.

# Contours d'un composant

Autant la définition d'un composant, au sens UML, est claire, autant sa mise en œuvre est plus compliquée. Déjà, première question : quel est le contour du composant ? Quelles sont ses limites ? Quelles fonctions mettre dedans ?

C'est là le rôle d'un architecte logiciel. Il aura la tâche de découper en composants son application. Il n'y a rien de fondamentalement faux, tout du moins au début, c'est vraiment une histoire de vision personnelle et de stratégie à moyen terme (évolutions que l'on désire apporter dans un avenir proche). En effet, un composant peu être soit "gros", touffu, comportant beaucoup de fonctions ou soit léger et très limité.

Pour ma part, j'opte pour le second choix : je vois un composant applicatif comme quelque chose de petit, un logiciel en C pur, c'est-à-dire qu'il peut potentiellement être compilé par n'importe quel compilateur. On va donc éviter d'embarquer du code d'appel système, d'accès au réseau ou fichier ou d'autres composants.

Dès lors, voici les éléments clés de notre architecture par composant :

  * Des composants léger, sans dépendances, purement logiciels (C pur)
  * Une librairie externe est un composant
  * Des composants systèmes, propres à une plateforme
  * Des entités dotées d'une interface commune de gestion dynamique (init, start, stop...)
  * Un système principal, le point d'entrée, de plus haut niveau se chargeant de piloter l'ensemble

Reprenons le diagramme d'architecture du code d'origine. Nous avons, dans un seul fichier, le code de la logique du Tetris (déplacement des pièces dans une matrice) et le code du "rendu". Celui-ci est très fortement lié à l'écran monochrome de la centrale de mesure et de la librairie de primitives utilisée dans les écrans de l'appareil.

Il faut casser cela. Pour deux raisons :

  1. Nous avons dit que nous voulions des composants "légers", purement logiciels, réalisant une fonction logique. On voit bien ici que l'on mélange deux notions
  2. Mettre le code de rendu dans le code de la logique va nous bloquer demain : changement de résolution, ajout de couleurs ... il faudra tout changer dans ce composant. Ce qui est dommage, car la logique elle ne changera pas.

Dès lors, on peut déjà créer deux composants : le composant A se chargera de la logique de Tetris, le composant B un rendu. Nous allons d'ailleurs nommer ce composant B explicitement : il s'agira d'un rendu "faible résolution et monochrome". Selon la cible choisie, on choisira tel ou tel composant de rendu.

![image]({{ site.url }}/assets/articles/arch-tetris-part2/arch-final.png)


# Premier composant

Après cette introduction un peu théorique, continuons notre travail sur le Tetris. Nous allons créer notre premier composant : le moteur de jeu. On l'extrait du fichier, en renommant au passage les variables et les fonctions (nommage du style XXX_MaFonction(), XXX étant le nom du module). On n'oublie pas ses fondamentaux en C : les fonctions privée préfixée par "static". On en profite pour renommer quelques variables et fonction en français.

La gestion des événements du clavier était avant réalisée par un appel à une fonction globale : on oublie cette mauvaise pratique et on les passe plutôt par arguments.

Enfin, on supprime toutes les variables globales et on les positionne dans une structure de contexte. Notre code pourra donc opérer sur un nombre indéfini de contextes ... pensez au futur ! (multijoueurs par exemple). Au final, notre module ne comprend que des fonctions et quelques constantes servant à décrire les niveaux de jeu. On pourra imaginer plus tard un composant dédié à cet effet pour rendre cette partie plus modulaire !

# Première compilation

Le moteur de construction que nous allons utiliser est lui aussi modulaire. Il est à base de makefile, car disponible partout, et se veut simple et portable. Il a été conçu avec les objectifs suivants en tête :

  * Orienté composant : un simple fichier à ajouter listant les fichiers à construire suffit
  * Multi-plateforme : les compilateurs sont fournis sous forme de plug-in à part
  * Unique fichier Makefile racine listant toutes les cibles
  * Ré-utilisation de composants au sein du Makefile

Un article dédié a été rédigé sur ce blog, je vous renvoie à cette lecture ainsi qu'au dépôt Git correspondant.

Pour construire notre cible, on ajoute les différents composants dont nous avons besoin :

```make
ifeq ($(MAKECMDGOALS), unitris_cli)

APP_MODULES 	:= projects/unitris_cli $(LIB_UNITRIS) $(LIB_SYS_PRINTF) components/debug
APP_LIBPATH 	:= 
APP_LIBS 	:= 

endif

unitris_cli: $(OBJECTS)
	$(call linker, $(OBJECTS), $(APP_LIBS), unitris_cli)
```

# Composant de débogage

Pour tester ce premier composant, nous allons afficher la grille du moteur de jeu dans la console (avec des 1 et des 0). Le code correspondant ne sera pas placé dans le composant du moteur de jeu : cela ajoute une dépendance forte (stdio et printf), il se peut tout à fait qu'une des cibles future ne dispose pas de printf (dans le cas de l'embarqué) et que l'on souhaite déboguer vers une sortie spécifique (UART, SSH, fichier ...).

Voici donc notre code qui va "dumper" la grille de Tetris dans la console :

```c
void DBG_DumpGrid(const unitris_t *ctx)
{
    // init variables
    for (int i = 0; i < 21; i++)
    {
        for (int j = 0; j < 12; j++)
        {
            printf("%d ", ctx->grid[i][j]);
        }
        printf("\n");
    }
}
```

# Fonctionnement du moteur de jeu Tetris

Le jeu de Tetris utilise des formes appelés tétrominos, c'est à dire de figures fabriquées à partir de quatre carrés. Ils tombent sur une grille et il va falloir les imbriquer judicieusement pour faire disparaître des lignes complètes. Le contexte (structure C) contient une grille de 20 lignes sur 10 colonnes. Nous ajoutons une ligne fantôme (en bas) et deux colonnes sur les côtés afin de simuler les bords. Cette astuce va nous permettre de limiter le déplacement du tétromino, un "mur" est vu comme une autre pièce. Au démarrage, voici la grille vide, un zéro est un carré vide, un 1 est une pièce ou un mur.

```shell
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 1 1 1 1 1 1 1 1 1 1 1 
```

Chaque pièce est dessinée de la même façon, en dur dans le code, avec toutes ses rotations possibles. Voici par exemple le L inversé :

```shell
  // le L inversé
  0,0,0,0,
  1,1,1,0,
  0,0,1,0,
  0,0,0,0,

 // le L inversé avec une rotation
  0,1,1,0,
  0,1,0,0,
  0,1,0,0,
  0,0,0,0,
```

La logique du code s'occupera donc :

  1. à chaque "tick", de faire tomber la pièce en cours
  2. De tester les cas de blocage
  3. De gérer les niveaux et les scores

# Exécution du moteur

Notre programme principal va prendre ces deux composants, unitris et debug, et va procéder à 50 appels au moteur de Tetris. Après chaque appel, on affiche le contenu de la grille.

```c
#include <stdio.h>      /* printf, NULL */
#include <stdlib.h>     /* srand, rand */
#include <time.h>       /* time */

#include "unitris.h"
#include "debug.h"
 
int main(int argc, char **argv)
{
    unitris_t ctx;
    
    srand(time(NULL)); // initialize seed
    UNI_Initialize(&ctx);
    
    for (int i = 0; i < 50; i++)
    {
        DBG_DumpGrid(&ctx);
        UNI_Tick(&ctx, 0, 0, 0);
        printf("\n");
    }
    
    return 0;
}
```

Miracle ! On voit notre pièce qui descend progressivement :

```shell
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 1 1 1 0 0 0 0 1 
1 0 0 0 0 0 1 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 0 0 0 0 0 0 0 0 0 0 1 
1 1 1 1 1 1 1 1 1 1 1 1 
```

# Comment déboguer ?

Le débogage va dépendre de la cible. Notre système de build ne fait que générer un exécutable, il va falloir trouver le bon programme. Sur PC, j'utilise Qt Creator, il offre une interface pratique et est disponible sur tous les systèmes. Nous verrons comment faire au cas par cas, surtout pour les cibles embarquées.

# Conclusion

Nous voici donc doté d'un moteur de Tetris simple. Le code en lui même fait 200 lignes et quelques fonctions. Un peu de code statique pour définir différents éléments du jeu et c'est tout !
