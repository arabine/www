---
layout: post
title: "Packager vos applications Qt sous Linux"
date: 2018-06-27 15:10:00 -0120
comments: false
published: true
category:
- linux
- qt
---

> Il fut un temps où il fallait générer des packages pour chaque distribution (Ubuntu, Fedora, Arch ...) en utilisant à chaque fois un format spécifique et
des commandes alamniquées au possible. De plus, il fallait également suivre de près les évolutions de chaque distribution pour la gestion des dépendances. Horrible,
surtout lorsque l'on est seul. Heureusement, les temps ont changé !

# Les concurrents

Si la génération de packages au format des distribution a un sens si votre application est officiellement supportée (suivi des patchs de s&écurité, partage des librairies ...),
cela n'as pas vraiment grand intérêt lorsque vous voulez distribuer une application seule par votre propre canal de diffusion.

Depuis quelques années, nous voyons plusieurs formats concurrents permettant de générer un package "tout compris" contenant votre exécutable et toutes les ressources tierces
nécessaires à son fonctionnement (images, sons, librairies dynamiques ...). Cependant, ces nouveaux standards ne reposent pas tous sur la même architecture. On notera donc :

  * AppImage
  * Snap (le format créé par Ubuntu)
  * Flatpak

Pour la comparaison entre les trois, je vous redirige vers des tableaux comparatifs très bien faits, ici : https://github.com/AppImage/AppImageKit/wiki/Similar-projects

# Préparations

Tout d'abord, il faut télécharger une application, au format AppImage
justement, qui permet de générer notre package. C'est le pendant Linux des outils windeployqt et macdeployqt disponibles en standard dans votre installation de Qt pour respectivement Windows et
MacOS. Il s'agit de linuxdeployqt : https://github.com/probonopd/linuxdeployqt.

Une fois téléchargé, ajoutez la propriété "exécutable" au fichier, soit via votre gestionnaire de fichier, soit en ligne de commandes (chmod +x linuxdeployqt-continuous-x86_64.AppImage)

![libpst]({{ site.url }}/assets/articles/appimage/set_exec.png)

Ensuite, compilez votre application en release puis créer une arborescence

```shell
└── usr
    ├── bin
    │   └── your_app
    ├── lib
    └── share
        ├── applications
        │   └── your_app.desktop
        └── icons
            └── <theme>
                └── <resolution>
                    └── apps
                        └── your_app.png
```

Remplacer <theme> et <resolution> par le standard Freedesktop. Voici une série de commandes permettre de créer le squelette :

```shell
mkdir -p usr/bin
mkdir -p usr/lib
mkdir -p usr/share/applications
mkdir -p usr/share/icons/hicolor/512x512
```
Voici notre fichier contenant les informations standards, toujours en suivant les recommandations de Freedesktop :

```ini
[Desktop Entry]
Type=Application
Name=Ultimate QR Code
Comment=QR Code generator
Exec=UltimateQRCode
Icon=ultimateqrcode_logo
Categories=Graphics;2DGraphics;
```

Au final, voici notre arborescence :

```shell
usr/
├── bin
│   └── UltimateQRCode
├── lib
└── share
    ├── applications
    │   └── ultimateqrcode.desktop
    └── icons
        └── hicolor
            └── 512x512
                └── ultimateqrcode_logo.png

7 directories, 3 files
```

# Exécution

Maintenant, lançons la commande de génération, attention c'est un peu longuet, surtout à la première exécution. J'ai rencontré plusieurs difficultés ici :

  * Ayant Qt d'installé sur le système ET dans mon répertoire 'home', il faut préciser à l'outil lequel utiliser (sans le tilde, il ne fait pas l'expansion du chemin comme bash)
  * Il faut l'option -qmldir si vous utilisez QML (comme dans notre exemple), et pointer vers la racine de votre dépôt de sources
  * J'ai échoué quelques fois sur la syntaxe du fichier .desktop, c'est pénible car la vérification est faite vers la fin

Voici la commande utilisée :

```shell
~/Applications/linuxdeployqt-continuous-x86_64.AppImage usr/share/applications/ultimateqrcode.desktop -qmake=/home/anthony/Qt/5.11.1/gcc_64/bin/qmake -appimage -qmldir=/home/anthony/git/ultimateqrcode/
```

Oui ça fait peur, la commande a tendance à ne rien faire pendant longtemps mais ça finit par terminer, avec succès on espère. Pour ma part, j'ai un fichier nommé Ultimate_QR_Code-x86_64.AppImage, et qui fonctionne !
Notez qu'il est mis "exécutable" automatiquement par le script.

Enfin, j'ai demandé à ce que mon application, UltimateQRCode, soit référencée dans AppImageHub, un projet communautaire de référencement d'applications au format AppImage (Flatpak et Snap ont des équivalents).
Que reste-t-il à faire ? Un petit script Bash regroupant toutes ces étapes à partir du dossier source !

Retrouvez l'application pour Linux - N'IMPRTE LEQUEL ! - ici : https://github.com/arabine/ultimateqrcode/releases/download/latest/Ultimate_QR_Code-x86_64.AppImage

![libpst]({{ site.url }}/assets/articles/appimage/ultimateqrcode.png)
