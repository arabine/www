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

