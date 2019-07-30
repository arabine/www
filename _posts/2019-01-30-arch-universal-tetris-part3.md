---
layout: post
title: "Architecture par composant - Tetris générique (3)"
date: 2019-01-30 8:55:00 -0120
comments: false
published: false
category:
- architecture
- embedded
- games
---

> Nous sommes maintenant dotés d'un moteur de jeu Tetris simple. Dans cet article, nous allons nous intéresser au rendu avec une première implémentation sur PC, parce que c'est plus rapide à développer. Une fois ce moteur de rendu bien testé, nous pourrons nous attaquer à d'autres cibles embarquées plus problématiques.

# Rendu d'origine

Comme ceci a été expliqué dans le premier article, le moteur de rendu original utilisait une librairie de primitives qui dessinaiant dans un buffer mémoire. Celui-ci était ensuite transféré au contrôleur LCD. Nous allons garder le concept général, cependant nous allons modifier un peu l'implémentation pour se conformer à notre design de composant : pas de variables globales et utilisation d'un contexte (structure).

Histoire de rendre ce rendu le plus générique possible, nous allons également supprimer les définitions d'origine qui sont utilisées partout dans le code : la taille de l'écran (largeur et hauteur en pixels). Au lieu de cela, nous utiliserons notre structure de contexte :

```c
typedef struct
{
    uint32_t *Vmem;
    uint32_t mem_size;
    uint16_t display_width;
    uint16_t display_height;
} egl_t;
```

Notez que la zone mémoire dans la quelle travaillera la librarie est un pointeur vers une zone mémoire définie ailleurs. Là encore, et pour éviter des allocations dynamiques, c'est au code appelant d'instancier tout cela, dans son contexte à lui qui ne regarde en rien la librairie.

Les coordonnées (0,0) de zone mémoire commencent en haut et à gauche, c'est assez usuel. Cela correspond à notre premier bit du premier octet du tableau.

# Qt et QML

La première version complète du jeu que nous allons développer fonctionnera sur PC et utilisera la bibliothèque Qt. Attention, Qt vous offre deux façons de créer des applications de "bureau" :

  * Soit en utilisant les QtWidgets, la bilbiothèque d'origine surtout faite pour réaliser des applications bureautique avec des formulaires, des boutons, des tables et autres éléments graphiques usuels.
  * Soit en utilisant le récent QtQuick / QML, une toute nouvelle méthode pour écrire des interfaces graphiques en utilisant du Javascript mêlé à un langage desciptif maison.

C'est cette dernière librairie, le QML, que nous allons utiliser. Pourquoi ? La principale raison est que cette librairie desciptive est parfaitement adaptée aux appareils mobiles (téléphones, tablettes) ; les widgets sont bien épurés et il est facile de créer des layouts qui s'adaptent à différentes résolutions, quelques fois un peu faibles.

L'architecture que nous allons adopter se compose de trois couches logicielles, chacun son langage :

![image]({{ site.url }}/assets/articles/arch-tetris-part3/unitris-qt-arch.png)

  1. L'interface est en QML : dessin des boutons, la zone centrale qui affichera une image et un Timer qui sera notre base de temps pour faire "avancer" le moteur de jeu
  2. Qt permet de faire dialoguer facilement le monde QML avec le monde C++ via une classe de type QObject : le code QML peut faire appel à une méthode C++ et vice et versa.
  3. Enfin, nous créons notre première entité qui sera une composition du moteur de jeu Unitris, de la librairie graphique de dessin de primitives et quelques autres variables propres au rendu de l'ensemble de l'écran (les scores, du texte etc.)

Le tick généré par le timer QML sera de 50ms pour garder la logique originelle du moteur Unitris.

Notons une dernière chose sur cette architecture. Comme le montre le dessin ci-dessus, la couche de base va, après la mise à jour du buffer, générer un fichier au format BMP en mémoire. Pourquoi ? Pour deux raisons principales :

  * à l'instar du chapitre précédent, il est pratique de "voir" le contenu de la mémoire de dessin comme on l'a fait dans le chapitre précédent pour déboguer l'application. Imprimer dans la console, c'est bien, mais générer une image c'est mieux !
  * L'interface QML utilise un contrôle de type "affichage d'image", donc a priori cela sera facile d'afficher un bitmap

Enfin, vous voyez également une étape d'encodage de l'image au format Base64 ; là encore, il s'agit d'une astuce : en effet, les types de données partagées entre le monde C++ et QML sont limités et il va nous falloir passer une image complète au format BMP. Le type QString est géré de base (chaîne de caractères) , donc on va l'utiliser pour stocker notre image ... qui est binaire ! On va donc l'encoder dans le format Base64 qui ne contiendra au final que des caractères ASCII.

Le passage de l'image entre le C++ et QML se fait en une seule ligne :

```c++
QString imgBase64 = "data:image/bmp;base64," + QString::fromLatin1(bytes.toBase64().data());
```

# Affichage des caractères

Il est pratique d'ajouter le support de l'écriture de texte à notre librairie. Notez que l'on pourrait n'afficher que des images, mais cela veut dire qu'il faudrait générer des images pour chaque texte affiché. C'est gros en mémoire et pas très flexible s'il y a des traductions à faire ou des changements dynamiques (édition, paramétrage via l'IHM).

La librairie d'origine inclut une gestion d'affichage des caractères. Chaque caractère de la table ASCII standard a été dessiné (avec MS Paint !) puis mis bout à bout dans un fichier BMP unique.

![image]({{ site.url }}/assets/articles/arch-tetris-part3/characters.png)

La police de caractères utilisée permet de d'utiliser que 6 bits de large ce qui permet d'afficher plus de texte qu'une police sur 8 caractères. Par contre, un caractère est toujours stocké sur un octet, forcément.

Ensuite, la technique utilisée est simple et est un classique dans le domaine des jeux vidéo. Nous avons créé une "carte" où chaque caractère peut être accessible à un rang donné, multiple d'un offet (tous les caractères ont la même taille 8x6 pixels).

  1. Les caractères ASCII "visibles" commencent à 0x20 (l'espace)
  2. On sooustrait 0x20 à chaque caractère qui doit être affiché pour connaître sa place dans l'image
  3. On positionne un pointeur dessus et on affiche le caractère comme si c'était une image

# EGL : Embedded Graphics Library

Nous allons appeler la librairie de dessin de primitives EGL pour identifier ce composant. Un conseil : commencez doucement, une fonction à la fois. Lors d'un portage *avec* modifications, il faut être prudent et éviter de passer des heurs à modifier un gros fichier et voir que rien ne fonctionne au final.

Nous avons commencé ce portage par le dessin de base : mettre un pixel à 1 (noir) ou blanc. La fonction finale est la suivante :

```c
void EGL_SetPixel(egl_t *ctx, uint16_t x, uint16_t y, uint16_t value)
{
   uint32_t *addr;
   uint32_t bitmask;

   // protection
   if ((x < ctx->display_width) && (y < ctx->display_height))
   {
       // >> 5 is equivalent to divide by 32
       addr = ctx->Vmem+(ctx->display_width >> 5) * y + (x >> 5);
       bitmask = 0x80000000UL >> ( x & 31); // modulo 32
       if (value)
       {
          *addr |= bitmask; // set
       }
       else
       {
          *addr &= ~bitmask; // clear
       }
    }
}
```

Cette fonction doit être assez bien optimisée de base en C, même si de nos jours les compilateurs font bien leur travail. On applique les bonnes pratiques d'optimisation : des masques et décalages au lieu des opérateurs de division et modulo.

On continue, avec les primitives pour dessiner des segments verticaux et horizontaux puis un rectangle. Vient ensuite les premières difficultés, les fonctions les plus difficiles : l'affichage d'images.

Comme on l'a vu dans le premier chapitre, nous disposons d'un petit logiciel en ligne de commanes permettant de générer un tableau en C à partir d'une image au format BMP monochrome. Rappelons-nous, le code d'origine tourne sur un processeur DSP de Texas Instruments, le C5502. Or, cette architecture possède deux particularités :

  * Le type "char" est sur 16 bits, oui oui, pas d'accès au niveau octet en mémoire
  * C'est une architecture Big Endian

Donc, nous nous retrouvons avec des fonctions graphiques générant ... une bonne bouillie de pixels! En effet, le code a été optimisé avec des décalages et masques spécifiques à cette architecture. Pas très portable !

L'astuce est la suivante : on va modifier ces fonctions pour n'appeler que la fonction de base qui dessine un pixel, qui elle a été bien testée et fonctionne. Les accès mémoire ne se feront donc qu'en octet, complètement portable (ce sont les octets qui sont rangés dans un ordre différent entre Little et Big Endian, les bits eux ne bougent pas : les bits de poids fort sont toujours "à gauche" !).

Après avoir modifié les routines d'affichage d'image, voici ce que donne notre écran de démarrage :

![image]({{ site.url }}/assets/articles/arch-tetris-part3/unitris-qt.png)

Parfait, nous sommes capable d'afficher une image à n'importe quel endroit de l'écran.

# Dynamique : faisons descendre la pièce

Là c'est facile, on traverse les couches une à une : le callback du Timer QML fait appel à une méthode de UnitrisWrapper (tick) qui elle même fait appel au "Tick" de l'entité du rendu graphique. En retour, on a la l'image BMP de l'état en cours de l'écran. Celui-ci est donc rafraichi toutes les 50ms, soit à 20Hz.

# Gestion des touches



