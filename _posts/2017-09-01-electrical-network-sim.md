---
layout: post
title: "Simulation d'un réseau électrique sur ARM avec Modbus/RTU"
date: 2017-09-01 18:43:00 -0700
comments: false
published: false
category:
- Tools
---

> Voici la présentation d'un sujet mixant plusieurs technologies. Le sujet principal consiste en la création d'échantillons afin de simuler un réseau électrique avec différentes paramètres permettant de faire varier lescaractéristiques du réseau (et ajouter des déformations). Nous allons utiliser de la simulation sous Scilab, de la génération de code, du Modbus, de l'Uart, du Qt
pour nos outils graphiques et du STM32 !

# Introduction

## Théorie

## Modbus

## Architecture

# En pratique

## Cahier des charges

Essayons notre moteur de build avec un exemple simple, et assez concret. Nous travaillons sur un projet réalisant la simulation d'un réseau électrique. Nous allons avoir :

  * D'un côté le simulateur, qui possède un protocole "serveur" de communication servant à accéder aux données. Ce simulateur fonctionnera sur PC et sur cible emarquée, avec donc des fichiers en commun
  * De l'autre un client, pour aller chercher ces données, il fonctionnera sur PC
  * Un logiciel de test automatique

Nous avons donc quatre logiciels à générer, trois cibles hôte (PC), et une cible emarquée. Le protocole de communication sera du Modbus, en TCP/IP pour le simulateur et en Modbus/RTU sur une liaison série pour la cible embarquée.

## Arborescence projet

Notre projet est donc composé de code applicatif, notre projet, qui reposera sur plusieurs librairies.

  * Une librairie de test automatisé
  * Une librairie Modbus client et serveur

Côté applicatif, on va distinguer les composants réutilisables des composants dédiés, notamment les composants embarqués (drivers). Nous aurons donc :

  * Un module "client"
  * Un module "samples", qui aura la charge de générer les échantillons du réseau électrique
  * Un module "simulator", sur PC, un outil graphique réalisé en Qt
  * Un module "mcu" qui contiendra le code source embarqué
  * Un module "tests" qui contiendra notre moteur de tests et le plan de test automatique

Nous voyons donc que le module "samples" sera utilisé par deux programmes différents, comme l'est le module modbus.

## Création des cibles

Le Makefile principal va contenir nos différentes cibles.

# Conclusion

Nous voici donc dotés d'un sytème simple, reproductible et extensible pour nos projets modulaires.
