---
layout: post
title: "Tanca - Méthode d'appariement pour un tournoi de type Suisse"
date: 2017-06-17 20:00:00 -0700
comments: false
published: true
category:
- blog
- pétanque
- tanca
---

> Tanca est le nom d'un logiciel de gestion d'adhérents, de tournoi et de championnat conçu pour le Club de Pétanque de ma commune. Durant l'année, nous organisons
quelques tournois au format "Suisse" qui est assez bien adapté pour nous. Cet article explique comment, avec quelques techniques simples, il est possible de résoudre
les problèmes d'appariement.

# La problématique

Une fois les joueurs inscris à un tournoi, il va falloir générer les rencontres entre les équipes. Plusieurs styles de tournois existent, tout dépend de l'organisateur.
Le format classique est le tournoi à élémination directe ou encore à double élémination. L'élémination directe peut être précédée d'une phase de poule pour faire jouer aux
participants un nombre minimal de parties.

Car c'est bien là l'intérêt ; en tant que Club de Pétanque amateur, il nous est paru évident d'offrir à tout inscrit qui a payé son droit d'entrée un minimum de parties. Le tout
devant se dérouler en une petite journée, sans terminer trop tard le soir (vers 18h étant l'idéal).

## Organisation du tournoi

En partant de cette situation, nous avons décidé de faire jouer quatre parties à tous les participants, deux le matin, deux l'après-midi. Généralement, nous avons environ vingt équipes. Il
y a alors deux méthodes pour la création des rencontres :

  * La méthode "toutes rondes" (round robin), toutes les rencontres sont tirées au hasard, 4 tours de jeu
  * La méthode dite "Système Suisse": seule la première rencontre est tirée au hasard, les suivantes sont en fonction du classement intermédiaire

Bien entendu, pour complexifier l'affaire, il faut prendre en compte la règle suivante : deux équipes ne peuvent pas se rencontrer plus d'une fois.

Autre règle : en cas de nombre d'équipes impair, une équipe tirée aléatoirement est forfaitaire pour ce tour (elle ne joue pas). Une équipe ne peut être forfaitaire qu'une seule fois
bien entendu ! Ceci est à prendre en compte lors de la création des rencontres.

## Système Suisse

Le Système Suisse permet donc à tous les participants de jouer un nombre égal de parties mais tout en étant plus "juste" que le format "toutes rondes". L'inconvénient avec la méthode
"toutes rondes" est qu'on peut se retrouver avec des rencontres injustes pour tout le monde : l'équipe la plus forte peut ne tomber que sur des rencontres faciles et inversement. D'une part,
ce n'est drôle pour personne, d'autre part cela ne donne pas une bonne image du concours.

## Classement des équipes

Avant de parler de la méthode appariement, voyons comment réaliser un classement. On n'invente rien, le classement est réalisé selon le nombre de matches gagnés puis, en cas
d'ex-aequos, on regarde le nombre de points marqués et encaissés. Cela donne, dans l'ordre :

  1. Nombre de matches gagnés
  2. Différence de la somme des points marqués avec la somme des points encaissés
  3. Buchholz : la somme des points des adversaires recontrés

Cette dernière méthode de départage permet de favoriser l'équipe ayant eu comme adversaire des équipes plus fortes.

Abréviations utilisées :

 * P: Games played (matches joués)
 * W: Games won (matches gagnés)
 * D: Games drawn (matches nuls)
 * L: Games lost (matches perdus)
 * PS: Points scored (points marqués)
 * PA: Points against (points encaissés)
 * +/-: goal difference (ie goals scored minus goals against) (points marqués moins les points encaissés)
 * Pts: Points
 * Bz: Buchholtz

## Appariement naïf

Imaginons huit équipes, numérotées de 1 à 8. Pour la première manche, il n'y a pas le choix, toutes les rencontres seront tirées au hasard. Voici ce que cela donne, avec en
parenthèses le résultat des matches.

```
3    contre    6   (13-1)
2    contre    8   (13-11)
4    contre    1   (8-13)
7    contre    5   (13-10)
```

Maintenant, réalisons un classement basé sur la méthode de départage décrite au chapître précédent.

| Equipe | W | L | PS   | PA  | +/- |
|--------|---|---|------|-----|-----|
| 3      | 1 | 0 |  13  |   1 |  12 |
| 1      | 1 | 0 |  13  |   8 |   5 |
| 7      | 1 | 0 |  13  |  10 |   3 |
| 2      | 1 | 0 |  13  |  11 |   2 |
| 8      | 0 | 1 |  11  |  13 |  -2 |
| 5      | 0 | 1 |  10  |  13 |  -3 |
| 4      | 0 | 1 |   8  |  13 |  -5 |
| 6      | 0 | 1 |   1  |  13 | -12 |

Maintenant, créons les rencontres pour la deuxième manche. Pour cela, dans le cas d'un appariement naïf, nous allons créer les couples d'équipes deux à deux en commençant par le
haut du tableau. On respecte donc notre but initial de faire jouer les équipes en fonction de leur niveau. Il faut également faire attention à ne pas faire jouer les équipes une deuxième fois
entre elles.

Cela donne les matches suivants, avec le résultat entre parenthèses :

```
3    contre    1   (13-8)
7    contre    2   (12-13)
8    contre    5   (2-13)
4    contre    6   (13-7)
```

Le classement intermédiaire devient :

| Equipe | W | L | PS   | PA  | +/- |
|--------|---|---|------|-----|-----|
| 3      | 2 | 0 |  26  |   9 |  17 |
| 2      | 2 | 0 |  26  |  23 |   3 |
| 5      | 1 | 1 |  23  |  15 |   8 |
| 7      | 1 | 1 |  25  |  23 |   2 |
| 4      | 1 | 1 |  21  |  20 |   1 |
| 1      | 1 | 1 |  21  |  21 |   0 |
| 8      | 0 | 2 |  13  |  26 | -13 |
| 6      | 0 | 2 |   8  |  26 | -18 |

On continue pour la troisième manche, même algorithme :

```
3    contre    2
5    contre    7
4    contre    1
8    contre    6
```

Aïe, nous avons un problème, l'équipe 5 a déjà joué contre l'équipe 7, de même avec les équipes 4 et 1. Il se peut que la troisième manche tombe bien, mais cela ne fera que repousser
le problème à la manche 4.

Une méthode semi intelligente consiste à "sauter" l'équipe qui a déjà été jouée ; par exemple ici pour trouver l'adversaire de l'équipe 5, on saute l'équipe 7 et on tombe sur
l'équipe 4. 7 et 1 joueront donc ensemble, ce qui résoud le problème pour ce tour. Encore une fois, cela ne fait que repousser le problème et il y a for à parier que
le tour 4 vous jouera des soucis d'appariement ...

Il faut donc trouver un moyen qui nous assure un appariement le plus juste possible tout en respectant la règle de l'unicité des adversaires rencontrés.

## Littérature sur le sujets

La littérature et les théories abondent sur le sujet, car il n'est pas simple. La théorie qui permet de trouver le couplage idéal se nomme la théorie des graphes. Je ne vais
pas résumer ce sujet ici, je ferai plus mal que Wikipedia, je vous invite donc à consulter les différents articles à ce sujet.

Autres mots clés pour ceux que cela intéresse : méthode Hongroise, Dutch algorithm, Jack Edmonds, Leaguevine.

La solution adoptée ici s'inspire d'un peu tout cela et surtout du dernier laron : Leaguevine.

# Méthode des matrices

La méthode que nous allons utiliser, très empirique et ne reposant sur aucune théorie, vise à trouver le couplage parfait entre deux équipes à l'aide de trois outils :

  * La séparation en deux du classement (les "forts" et les "faibles")
  * L'affectation d'un poids (un nombre) très élevé aux coupages non désirés (typiquement les équipes qui ont déjà jouées ensemble)
  * Un poids en fonction de la force de chaque rencontre potentielle (basé sur la différence de points)

## Etude d'un cas réel

Prenons un exemple tiré d'un vrai concours. Chaque jeu est enregistré dans une base SQL, les différents identifiants sont à mettre en relation avec d'autres tables de la base (table des joueurs et   tables des équipes).

Voici l'extrait de la base, la liste des jeux et leur résultat pour 10 équipes (concours de doublettes, 20 joueurs).

![tanca]({{ site.url }}/assets/articles/tanca-swiss-pairing/turn0_games.png)

Le classement après ce premier tour est le suivant :

| Equipe | W | L | D   | +/- | Bz |
|--------|---|---|------|-----|-----|
| 166 | 1 | 0 | 0 | 13 | 0 |
| 168 | 1 | 0 | 0 | 6 | 7 |
| 161 | 1 | 0 | 0 | 5 | 8 |
| 164 | 1 | 0 | 0 | 3 | 10 |
| 160 | 1 | 0 | 0 | 1 | 12 |
| 165 | 0 | 1 | 0 | -1 | 13 |
| 169 | 0 | 1 | 0 | -3 | 13 |
| 167 | 0 | 1 | 0 | -5 | 13 |
| 162 | 0 | 1 | 0 | -6 | 13 |
| 163 | 0 | 1 | 0 | -13 | 13 |

Passons donc le résultat de cette première manche à travers la moulinette de notre algorithme. Dans un premier temps, on coupe le classement en deux : les meilleurs d'un côté, les plus mauvais de l'autre.

Intéressons-nous à la liste des meilleurs (on procèdera de la même manière avec la deuxième liste). Pour cela, nous allons créer une matrice et, pour
chaque combinaison de rencontre on assigne notre fameux poids.

Voici ce que donne notre matrice:

```
---------------  WINNERS COST MATRIX -------------------
	166	168	161	164	160	165
166 [ 	10000	7	8	10	12	14 ]
168 [ 	7	10000	1	3	5	7 ]
161 [ 	8	1	10000	2	4	6 ]
164 [ 	10	3	2	10000	2	4 ]
160 [ 	12	5	4	2	10000	10000 ]
165 [ 	14	7	6	4	10000	10000 ]
```

Nous pouvons remarquer deux choses :

  1. La matrice est symétrique, inutile donc de s'occuper de toutes les cases
  2. Le poids élevé (>=10000) pour les équipes s'étant déjà rencontrées (c'est le cas ici pour les équipes 165 et 160) ou les paires impossibles (deux équipes identiques)

Les autres points sont l'écart des différences entre les deux équipes. Plus cette valeur est faible, plus les équipes sont sensées être de force équivalente. La prochaine étape est de créer
les rencontres avec cette matrice. Nous allons donc créer un arbre: l'arbre des rencontres possibles.

## Détermination des rencontres possibles

Notre algorithme va chercher, dans la moitié de la matrice uniquement cu qu'elle est symétrique, toutes les
rencontres possibles. Pour chaque rencontre, on calcule la somme des poids (voir le chapitre précédent). On
rejette systématiquement les solutions ayant un total supérieur à 10000, notre limite.

Nous obtenons 12 combinaisons possibles :

```
----------- Solution 1 cost is :15
168 <--> 166
160 <--> 161
165 <--> 164

 ----------- Solution 2 cost is :15
168 <--> 166
165 <--> 161
160 <--> 164

 ----------- Solution 3 cost is :17
161 <--> 166
160 <--> 168
165 <--> 164

 ----------- Solution 4 cost is :17
161 <--> 166
165 <--> 168
160 <--> 164

 ----------- Solution 5 cost is :21
164 <--> 166
160 <--> 168
165 <--> 161

 ----------- Solution 6 cost is :21
164 <--> 166
165 <--> 168
160 <--> 161

 ----------- Solution 7 cost is :17
160 <--> 166
161 <--> 168
165 <--> 164

 ----------- Solution 8 cost is :21
160 <--> 166
164 <--> 168
165 <--> 161

 ----------- Solution 9 cost is :21
160 <--> 166
165 <--> 168
164 <--> 161

 ----------- Solution 10 cost is :17
165 <--> 166
161 <--> 168
160 <--> 164

 ----------- Solution 11 cost is :21
165 <--> 166
164 <--> 168
160 <--> 161

 ----------- Solution 12 cost is :21
165 <--> 166
160 <--> 168
164 <--> 161
```

Nous vons donc au final deux solutions possibles ayant les plus faibles totaux (15). Nous prenons n'importe laquelle, nous avons réussis à déterminer notre configuration la plus optimale pour des rencontres
plus intéressantes.

## Implémentation

Vous trouverez un exemple d'implémentation dans le logciel "Tanca" dont le code source est disponible sur GitHub. La classe intéressante se nomme "Tournament.cpp", c'est elle qui implémente les calculs d'appariement et de classement. Peut-être que nous y reviendrons plus en détails sur les algorithmes, si j'ai le temps d'approfondir la chose.

## Nombre idéal de parties

Avec quatre parties jouées, il est possible de tomber sur deux équipes ayant gagnées tous leurs matchs, avec 20 équipes. Bien entendu, nous pouvons les départager à l'aide des autres statistiques (différence des points et le Buchholtz). Idéalement, il faudrait jouer une cinquième partie mais pour un tournoi amateur cela commence à faire beaucoup de matchs.

Je n'ai pas cherché la méthode pour déterminer le nombre idéal de matchs en fonction du nombre d'équipe. Encore quelque chose à creuser, j'éditerai cet article si je parviens à mettre la main sur quelque chose.

Evidemment, au delà de cinq parties pour 20 participants, l'algorithme ne va pas pouvoir trouver de solutions en respectant la règle visant à ne pas rencontrer deux fois le même adversaire.

# Conclusion

Nous voici armé d'un algorithme efficace pour gérer nos tournois de Pétanque, sans stress.
