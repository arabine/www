---
layout: post
title: "Introduction au protocole CoAP - partie 2"
date: 2019-01-20 8:55:00 -0120
comments: false
published: true
category:
- embedded
---

> Continuons notre découverte du protocole CoAP. Après une petite introduction sur le principe et quelques manipulations, nous allons complexifier un peu la chose en développant notre propre client et serveur.

# Le client

Commençons par le plus simple, tout du moins par ce qui est le plus facile à tester : le développement d'un client vu que l'on dispose de serveurs de test (voir la partie 1).

Nous allons utiliser la librairie libcoap pour tous nos développements. La librairie dispose d'un exemple de client et de serveur. Le client est assez complet car il s'agit d'un programme en ligne de commandes avec tout un tas d'options. Pourquoi ré-implémenter un client ? Pour deux raisons :

  1. Comprendre mieux la librairie en vue de développer la partie serveur
  2. Alléger le code : on désire un client tout simple, sans options
  3. Créer une couche intermédiaire de façon à avoir un client fonctionnel sous forme de librairie intégrable en C++ (création d'un wrapper)
  4. Utiliser la librairie dans le cas d'un gestionnaire de télé-relève d'appareils CoAP

Commençons par créer un dépôt git pour notre projet avec licoap en tant que sous-module. Le dépôt officiel est ici : `https://github.com/obgm/libcoap.git`

La meilleur méthode, lorsque vous utilisez une librairie tierce, est de créer dans un premier temps un projet minimaliste avec une fonction main() et d'utiliser une en-tête de la librairie : cela va permettre de régler les chemins. Puis, appelez une fonction de la librairie, typiquement une fonction d'initialisation, cela va permettre de régler les options de l'édition des liens.







