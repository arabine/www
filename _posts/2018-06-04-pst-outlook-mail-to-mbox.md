---
layout: post
title: "Comment lire des fichiers PST sous Linux"
date: 2018-06-04 18:43:00 -0700
comments: false
published: true
category:
- PC
- tools
- mail
---

> Si comme moi vous avez travaillé de force sous Windows en entreprise, il est fort probable qu'Outlook fut votre client mail quotidien. Il est toujours
intéressant de garder des archives de votre expérience passée, mais voilà : impossible de lire nativement le format de stockage PST de Microsoft sous Linux.

# La librairie qu'il nous faut

Allons télécharger tout de suite LA librairie nécessaire à l'opération : ![LibPST}(http://www.five-ten-sg.com/libpst/)

Décompresser l'archive, entrez dedans via la ligne de commandes puis tapez les classiques :

```shell
./configure
make -j4
sudo make install
```

Vous aurez besoin d'au moins les bibliothères suivantes (sous Ubuntu), en plus de la chaîne complète de compilation (build-essential) :

```shell
sudo apt-get install libgsf-1-dev libboost-python-dev
```


## Lançons la conversion

La librairie LibPST va nous permettre de convertir le format de Microsoft vers le format MBOX, plus standard. Le programme est maintenant installé dans le système et l'outil peut être appelé de n'importe où. Allez dans le même répertoire que le fichier PST à convertir
et tapez :

```shell
anthony@anthony-desktop:~/Téléchargements/pst$ readpst -o . -k  Archives2017.pst
Opening PST file and indexes...
Processing Folder "Éléments supprimés"
Processing Folder "SentIn2017"
        "SentIn2017" - 337 items done, 0 items skipped.
        "Archives2017" - 1783 items done, 1 items skipped.
```

Voici l'arborescence obtenue, on voit bien le fichier MBOX généré :

```shell
.
├── Archives2017
│   ├── mbox
│   └── SentIn2017
│       └── mbox
└── Archives2017.pst
```

## Maintenant, la lecture des mails

Il existe des tas de moyens pour lire un fichier MBOX. Étant utilisateur de Thunderbird, je vais l'utiliser. Il suffit de copier le fichier mbox généré dans un sous répertoire de votre répertoire local de Thunderbird. Celui-ci se situe dans un chemin du genre /home/<votre home>/.thunderbird/<nom aléatoire compliqué>.default/Mail/Local Folders/. Chez moi il est là :

```shell
/home/anthony/.thunderbird/14qmrf0r.default/Mail/Local Folders/
```
Voilà, maintenant relancez Thunderbird et oh miracle vous pouvez accéder à vos emails !

![libpst]({{ site.url }}/assets/articles/libpst/pst_thunderbird.png)
