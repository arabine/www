---
layout: post
title: "Introduction au protocole CoAP"
date: 2018-12-05 8:55:00 -0120
comments: false
published: true
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

Voici la tête de la trame, c'est un extrait de la RFC :

```
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |Ver| T |  TKL  |      Code     |          Message ID           |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |   Token (if any, TKL bytes) ...
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |   Options (if any) ...
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |1 1 1 1 1 1 1 1|    Payload (if any) ...
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

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

Que se passe-t-il au niveau du protocole ? Regardons un exemple tiré de la RFC :

```
REQ: GET /.well-known/core
RES: 2.05 Content
</sensors/temp>;rt="temperature-c";if="sensor",
</sensors/light>;rt="light-lux";if="sensor"
```

Cette requête de découverte nous retourne deux URLs situées sous '/sensors' permettant a priori de récupérer les données de plusieurs capteurs.

Réalisons un simple get pour obtenir les données de l'URL /hello :

```
coap://coap.me:>get /hello
----------------------------------- Response -----------------------------------
GET coap://coap.me/hello
MID: 18961, Type: ACK, Token: [98d43c14048de4a1], RTT: 54ms
Options: {"Content-Format":"text/plain"}
Status : 205-Reset Content, Payload: 5B
................................... Payload ....................................
world
--------------------------------------------------------------------------------
```

Voyons avec Wireshark ce que donne cette requête :

![coap]({{ site.url }}/assets/articles/coap-intro/get_packet.png)

Nous avons ici la pile de protocoles classique longue de 69 octets :
  * En vert la trame Ethernet de 14 octets
  * En rose la trame IPv4 de 20 octets
  * En bleu la trame UDP de 8 octets
  * En orange la trame CoAP de 27 octets

Notez que pour des paquets encore plus courts il faudrait passer sur de l'IPv6 et une une en-tête compressée 6LowPAN. Le contenu de la trame CoAP ne contient que peu d'informations :

![coap]({{ site.url }}/assets/articles/coap-intro/coap_packet.png)

La place est prise essentiellement par l'URI hello, le nom de l'hôte coap.me et un token sur 8 octets. Ce 'token' est généré par le client, le serveur ne faisant que le recopier. Il sert au client à trier les réponses reçues en fonction de son applicatif. Cet token s'avérera utile pour une utilisation en observateur: il indiquera la nature la trame reçue de façon asynchrone. Le message ID est quant à lui plus utilisé au niveau protocolaire pour détecter des paquets re-transmis. On peut ajouter tout un tas d'options à la trame (le token est lui aussi une option !). 

Voici un essai avec la méthode PUT, permettant d'envoyer des données :

```
coap://coap.me:>put /sink --payload "Coucou"
----------------------------------- Response -----------------------------------
PUT coap://coap.me/sink
MID: 48890, Type: ACK, Token: [8ccee7e647b4990d], RTT: 55ms
Options: {"Content-Format":"text/plain"}
Status : 204-No Content, Payload: 6B
................................... Payload ....................................
PUT OK
--------------------------------------------------------------------------------
coap://coap.me:>get /sink
----------------------------------- Response -----------------------------------
GET coap://coap.me/sink
MID: 48891, Type: ACK, Token: [a234b4173035967e], RTT: 55ms
Options: {"ETag":0xb9a6b9ae201a9d71, "Content-Format":"text/plain"}
Status : 205-Reset Content, Payload: 20B
................................... Payload ....................................
you put here: Coucou
--------------------------------------------------------------------------------
```

Dernier détail rigolo du protocole, l'URI est découpée en options. Exit les slash, chaque "répertoire" à la REST est envoyé sous forme d'option. Cela facilite le travail du serveur et on comprend bien l'orientation très REST du protocole.

![coap]({{ site.url }}/assets/articles/coap-intro/get_uri_split.png)

Voilà un bref tour de la partie protocolaire de CoAP, orienté application comme le HTTP, mais en "compressé".

# Test de la fonction observateur


Essayons maintenant un autre serveur hébergé chez la fondation Eclipse. On se connecte et on liste les services observables.

```
connect coap://californium.eclipse.org
discover --query obs
┌────────────────┬──────────────────┬─────────────────┬──────────────┬─────────┬────────────────┐
│Path [href]     │Resource Type [rt]│Content Type [ct]│Interface [if]│Size [sz]│Observable [obs]│
├────────────────┼──────────────────┼─────────────────┼──────────────┼─────────┼────────────────┤
│/obs            │observe           │text/plain (0)   │              │         │observable      │
│/obs-large      │observe           │                 │              │         │observable      │
│/obs-non        │observe           │                 │              │         │observable      │
│/obs-pumping    │observe           │                 │              │         │observable      │
│/obs-pumping-non│observe           │                 │              │         │observable      │
└────────────────┴──────────────────┴─────────────────┴──────────────┴─────────┴────────────────┘
```

Ok, un get /obs nous renvoie l'heure, c'est un serveur de temps.

```
oap://californium.eclipse.org:>get /obs
----------------------------------- Response -----------------------------------
GET coap://californium.eclipse.org/obs
MID: 37018, Type: ACK, Token: [ee175ae6ba1c2ec3], RTT: 124ms
Options: {"Content-Format":"text/plain", "Max-Age":5}
Status : 205-Reset Content, Payload: 8B
................................... Payload ....................................
20:30:37
--------------------------------------------------------------------------------
```

Maintenant, enregistrons nous pour recevoir des trames dès que l'heure change.

```
coap://californium.eclipse.org:>observe /obs
OBSERVE START coap://californium.eclipse.org/obs
```

Attendons un peu, et listons les messages reçus :


```
coap://californium.eclipse.org[OBS]:>observe show messages
----------------------------------- Response -----------------------------------
OBSERVE Response (coap://californium.eclipse.org/obs):
MID: 37021, Type: ACK, Token: [1f765efaeebc7cc2], RTT: 199ms
Options: {"Observe":155554, "Content-Format":"text/plain", "Max-Age":5}
Status : 205-Reset Content, Payload: 8B
................................... Payload ....................................
20:37:22
--------------------------------------------------------------------------------
----------------------------------- Response -----------------------------------
OBSERVE Response (coap://californium.eclipse.org/obs):
MID: 50642, Type: CON, Token: [1f765efaeebc7cc2], RTT: 1ms
Options: {"Observe":155555, "Content-Format":"text/plain", "Max-Age":5}
Status : 205-Reset Content, Payload: 8B
................................... Payload ....................................
20:37:27
--------------------------------------------------------------------------------
----------------------------------- Response -----------------------------------
OBSERVE Response (coap://californium.eclipse.org/obs):
MID: 50643, Type: CON, Token: [1f765efaeebc7cc2], RTT: 0ms
Options: {"Observe":155556, "Content-Format":"text/plain", "Max-Age":5}
Status : 205-Reset Content, Payload: 8B
................................... Payload ....................................
20:37:32
--------------------------------------------------------------------------------
```

Nous avons reçu trois messages espacés de 5 secondes. C'est le serveur CoAP qui décide de la fréquence d'envoi, soit en dur dans le code soit on peut imaginer un système de configuration.

# La suite ...

Dans un prochain article, nous mettrons en œuvre le protocole côté serveur, en C sur PC, et nous le validerons avec un client.
