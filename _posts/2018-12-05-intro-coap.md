---
layout: post
title: "Introduction au protocole CoAP"
date: 2018-12-05 8:55:00 -0120
comments: false
published: false
category:
- embedded
---

> Le protocole CoAP a été créé il y a quelques années pour permettre aux objets connectés aux ressources limitées de transmettre des données de façon standardisée. Cette introduction va nous permettre de manipuler et de comprendre le protocole , puis verrons par la suite une mise en œuvre sur un microcontrôleur.

# Un peu de documentation pour commencer

Je ne vais pas copier-coller Internet, cela ne sert à rien, car nous allons découvrir le protocole de façon pratique, par contre je mets quelques références sur le protocole en lui-même :

  * https://fr.wikipedia.org/wiki/CoAP
  * http://coap.technology/
  * https://www.bortzmeyer.org/7252.html
  * https://tools.ietf.org/html/rfc7252

la RFC 7252 est la spécification "de base" du protocole. À celle-là s'ajoute une tripotée de spécifications additionnelles, les deux premières sont optionnelles, la troisième nous servira lors de nos essais :

  * Block-wise transfers : https://tools.ietf.org/html/rfc7959
  * Observing resources : https://tools.ietf.org/html/rfc7641
  * https://tools.ietf.org/html/rfc6690 (discovery protocol)


La première option vous permettra de transférer des "gros" paquets segmentés, la deuxième vous permettra de faire l'équivalent d'un "push" : une donnée asynchrone est envoyée du serveur au client.

Donc, si vous avez lu les quelques documents présentés ci-dessus, vous voyez que le format d'en-tête est relativement petit, la trame minimale fait 4 octets. Voici les atouts majeurs de CoAP :

  * Fonctionne sur UDP, standard et à faible empreinte mémoire
  * Peut utiliser optionnellement DTLS, le pendant de TLS pour UDP
  * L'option "observing" est intéressante pour les objets connectés de monitoring
  * Utilise une API de type "REST" (des URL) avec des méthodes GET, PUT et tout le tintouin, ce qui est standard aussi et proche du monde Web

Voilà, c'est un résumé très grossier, il y a plein d'autres avantages et options/extensions possible mais dans un premier temps cela suffira.

# Amusons-nous avec coap.me

Le site http://coap.me propose un serveur de test doté de plusieurs URL. Il est possible d'effectuer les requêtes via le site, qui est doté d'une passerelle, ou directement avec le protocole. Essayons : http://coap.me/coap://coap.me/hello retourne un "world".

Plus intéressant, voyons ce que répond l'URL de base : http://coap.me/coap://coap.me/.well-known/core. C'est une URL assez bizarre en apparence, mais néanmoins complètement standard : il s'agit de l'implémentation de la RFC6690. En quoi consiste-t-elle ? C'est très simple, elle liste les services du serveur (les URL, n'oubliez pas on est en API REST). On retrouve donc notre '/hello' précédent, et bien d'autres que nous allons découvrir.


![coap]({{ site.url }}/assets/articles/coap-intro/coapme.png)

Le format renvoyé par une requête GET sur cet URL est un peu spécifique et décrit notamment dans la RFC8288 (Web Linking). Mais en gros, pour chaque URL gérée (href) par le serveur, on peut y associer un gros tas d'attributs sous la forme d'une paire de clé/donnée. Voici quelques attributs standardisés dans CoAP :

  * rt (Resource Type) : un espèce de mot-clé le plus court possible permettant d'identifier le HREF, a priori utilisé pour des recherches ou des robots
  * title : du texte humainement lisible décrivant le HREF
  * if (Interface Description) :  de ce que je comprends, permet de préciser le format des données renvoyées. On peut par exemple renvoyer sur une norme ou un lien internet décrivant le format.
  * sz (Maximum Size Estimate) : taille maximale de la donnée, je ne pense pas que cela soit très utile car pour du CoAP on s'attend à des petites données. Je ne sais même pas comment cet attribut peut être utilisé par un client.
  * ct (Content Format) : a priori une indication sur le format de la donnée, ce n'est pas à prendre stricto-sensus, pour cela il vaut mieux s'appuyer sur le Content-Format du protocole.

Bref, ces attributs ne vont pas nous servir beaucoup ; cela peut être pratique pour savoir comment parler à une machine dont on ne possède pas de documentation. Notez que l'on peut a priori effectuer des recherche/filtres via l'URL.

Ok, maintenant tentons avec un client CoAP, nous en aurons besoin de toute façon pour tester notre serveur.

Quelle librairie et logiciel client choisir ? Alors là c'est assez compliqué ; au moment de l'écriture de cet article, peu d'implémentations sont en même temps vraiment complètes et bien maintenues. Il ne reste guère de choix dès que l'on requiert du DTLS, c'est un minimum en 2018.

Au bout du compte, on a :
  * https://github.com/obgm/libcoap : un peu la référence, complète et bien maintenue. On peut lui reprocher un code source un peu brouillon, une mauvaise séparation des couches et un code assez volumineux en RAM/ROM. Cependant, les choses s'améliorent, et la dernière version commence à être intéressante. Pour le DTLS, on a en plus le choix entre TinyDTLS et OpenSSL, la classe. Dernier avantage, la librairie propose un client et un serveur de référence fonctionnant sur Linux et Windows.
  * https://github.com/tzolov/coap-shell : ça a l'air sympa avec un résultat des requêtes bien formaté ; il est complet, en Java mais ce n'est pas trop grave pour un client. Nous allons le découvrir ici.

Bien que l'on pourrait faire nos essais avec libcoap uniquement, il est toujours intéressant de tester un dialogue client/serveur avec deux implémentations différentes : si ça communique, on peut dire que le système doit globalement respecter la norme.

Commençons avec coap-shell. Il suffit de télécharger le binaire pré-compilé (un fichier JAR, l'URL se trouve dans le README à la racine du dépôt), puis de lancer la console interactive via la commande ava -jar coap-shell-1.0.7.jar. Connectez-vous au serveur, puis lorsque l'on lance la commande de découverte, on obtient les mêmes informations que via le site web.

```
server-unknown:>connect coap://coap.me
available
coap://coap.me:>discover
```
![coap]({{ site.url }}/assets/articles/coap-intro/coap-shell-coapme.png)


# Quelle librairie embarquer choisir ?





# Installation de CoAP et essais

+

