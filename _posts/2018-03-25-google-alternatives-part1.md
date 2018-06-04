---
layout: post
title: "Dégooglisation personnelle - partie 1"
date: 2018-03-25 18:43:00 -0700
comments: false
published: true
category:
- cloud
- tools
---

> Il est difficile de se passer des outils Cloud des mastodontes Microsoft/Google/Apple. Il faut dire que leurs applications sont plutôt bien pensées et efficaces. Je commence quelques articles décrivant ma tentative de sortir des outils obscurs et gagner plus de maîtrise dans le stockage de mes données.

# Cahier des charges

Mon cahier des charges est relativement simple :

  * Je ne veux pas à avoir à administrer mon propre serveur (pas le temps)
  * Je souhaite, dans la mesure du possible, éviter les créations de comptes à tout va sur d'autres nuages alternatifs
  * Privilégier les stockages ou les entreprises éthiquement à peu près au goût du jour
  * Prendre en compte la sécurité et notamment le chiffrement des données
  * Trouver des solutions multi-plateformes (Windows/Linux/Mac/iOS/Android)

# Outils d'organisation

## Besoin

Dans cette première partie, je vais me focaliser sur les outils d'organisation du travail, à savoir :

  * Mail
  * Agenda
  * Contacts
  * Prise de notes
  
## Le mail

Lorsque j'ai commencé mon activité de consultant, la première chose que j'ai faite est de me créer une adresse mail professionnelle. Il n'y a pas à hésiter, il vous faut un nom de domaine. Oubliez les adresses liées à votre fournisseur d'accès aux services limités en plus de montrer une image un peu amateure.

Pour cela, j'ai choisi mon registrar favori et français : Gandi (https://gandi.net). Ils sont petits mais éthiquement responsables, transparents et acteurs du logiciel libre. Un conseil : optez pour un nom de domaine assez court et facilement prononçable, vous aurez à le faire souvent ! Concernant le prix, c'est dérisoire, environ 14€ TTC par an, pour une extension en .fr en tout cas.

Un nom de domaine chez Gandi vous offrira en plus plein de petits services et notamment un email ! Dans le pack de base, vous avez le droit à 3Go, et une possibilité de monter à 50Go en prenant une option payante.

Côté logiciel :
  * Sur PC, Thunderbird reste sans failles. Je lui reprocherais un éditeur un peu daté et difficile à configurer. Mais il est Open Source et fourmille d'options, de fonctionnalités et d'extensions.
  * Sur Android, le mail "stock" fourni par Google est suffisant pour mon besoin.
  
Alternatives :
  * Le Webmail : Gandi passe par SoGo, un webmail vraiment sympa et dans les standards actuels. Je trouve cependant que les serveurs Gandi sont assez lents.
  * Sur Android, Outlook est vraiment un bon outil tout intégré qui s'intègre bien avec le reste du monde. Je l'ai lâché uniquement à cause d'un bug bloquant sur mon téléphone, sinon je pense que j'aurais continué avec lui.


## L'agenda

Point d'organisation sans un Agenda consultable à tout moment. Il faut donc trouver une solution simple et efficace sur PC et sur smartphone. Gandi, grâce à SoGo, permet d'utiliser le standard ouvert CalDav. 

>  Astuce ! Pour que les invitations au format "Outlook" ou "google" soient gérée par Thunderbird, pensez à nommer le calendrier importé par votre adresse email professionnelle. Sinon, il n'arrivera pas à faire le lien et à ajouter automatiquement l'événement à votre calendrier.

Concernant Android, il va falloir passer par une extension CalDav payante. Il en existe quelques-unes, pour ma part j'ai choisi DroidDav car le logiciel est Open Source, cela permet de rémunérer l'auteur pour son travail. Ce logiciel va tourner en tâche de fond pour synchroniser votre calendrier. Ainsi, vous pouvez choisir (normalement) n'importe quelle application.

>  Astuce ! Pour migrer mon calendrier, j'ai exporté celui de Google au format ICS puis je l'ai importé via SoGo. Aucune perte détectée, même une règle un peu compliquée a été importée avec succès.

Côté logiciel :
  * Installer l'extension Lightning pour Thunderbird, cela vous fait un excellent calendrier.
  * Pour Android, j'ai acheté aCalendar+ qui offre un calendrier vraiment bien fait et pratique à l'usage. Sinon, le calendrier par défaut de Google peut faire l'affaire pour un usage basique.

>  Astuce ! Abonnez-vous à des calendriers partagés des Vacances scolaire ! Pour certains événements, comme par exemple des grèves de train, créez des règles récurrentes.

Dans tous les cas, les logiciels supportant CalDav vous demanderont un emplacement (URL), un login et un mot de passe. L'URL est fournie via le Webmail de SoGo : connectez-vous, allez dans la partie calendrier et via le menu à côté du nom de votre calendrier choisissez "Liens vers ce calendrier".

## Les contacts

Bien entendu, la partie "contacts" va de pair avec le calendrier. Là encore, SoGo offre un moyen d'y accéder via CardDav, le pendant de CalDav pour les contacts.

Vous aurez sûrement à gérer l'exportation de vos contacts : rien de plus simple, exportez les contacts au format VCF via votre application d'origine (ou le Webmail), puis importez les via votre Webmail de destination ou votre client local déjà connecté.

La procédure de connexion avec votre logiciel et SoGo est la même que CalDav : dans le Webmail, via l'onglet "contact", cliquez sur "Liens vers ce carnet" afin d'obtenir l'URL de synchronisation.

Côté logiciel :
  * J'utilise l'extension CardBook pour Thunderbird, excellente et très complète
  * Sur Android, l'application DroidDav précédemment installée offre la synchronisation des contacts. Vous pouvez choisir ce compte dans les paramètres des contacts.


## Les notes (TODO list)

Trouver un remplacement à Google Keep a été pour moi le plus difficile pour plusieurs raisons. Déjà, parce que le logiciel de Google est excellent, sans fioriture et très rapide. Le widget permet de lister et de naviguer simplement dans la liste des notes. L'application Web est accessible de partout, sur tous les systèmes.

Malheureusement, côté application Android, c'est le désert complet. Je me suis rabattu sur OpenTasks, logiciel OpenSource, mais le Widget proposé n'est pas terrible, il ne permet pas de voir le contenu des notes, uniquement les titres.

L'application de calendrier aCalendar+ offre bien théoriquement une compatibilité avec les notes de CalDav mais elle est boguée à l'heure où j'écris ces lignes. Bref, je vais sûrement changer d'application dans les mois qui viennent en fonction du support de ce protocole dans les applications de notes.

Je n'ai rien eu de particulier à faire au niveau de la configuration sur Android car DroidDav assure l'abstraction complète tout seul.

Concernant l'application PC, j'utilise l'onglet "tâches" sous Thunderbird, disponible avec l'extension Lightning.

# Sauvegardes, derniers réglages et conclusion

## Les sauvegardes 

Concernant les sauvegardes de ces données, je ne m'y suis pas encore penché dessus. Là où Google offrait une sécurité bien pratique en sauvegardant l'historique, je n’ai aucune visibilité sur la sauvegarde des données côté serveur. Côté client, tous vos logiciels offrent un "cache", c'est à dire un enregistrement local des données pour permettre la consultation off-line. 

Dans l'idéal, il faudrait un serveur autonome qui se charge d'effectuer des sauvegardes périodiques. Je ne veux pas passer par là, n'ayant pas envie de passer du temps là-dessus. J'éditerai ce billet lorsque j'aurai trouvé quelque chose !

## Bye bye Google

Il y a un dernier réglage qui fait plaisir : la suppression des services Google. A partir de votre compte Google, supprimez toutes les données (calendrier, notes, contacts), y compris l'historique des modifications.

Du côté smartphone, allez dans la gestion de votre compte Google et désactivez toutes les synchronisations (sauf celles que vous utilisez).

## Conclusion

Et voilà, d'un coup on se sent mieux, on a vraiment l'impression de maîtriser ses données sans avoir à trop mettre les mains dans le cambouis et à devoir gérer un serveur personnel.

Côté PC c'est pratique, tout est regroupé sous un même logiciel : Thunderbird.


![tarotclub]({{ site.url }}/assets/articles/outils-partie1/thunderbird.png)


Finalement, mon compte Google ne me sert plus qu'au Drive et à acheter des applications. Nous verrons dans un prochain billet comment là encore trouver des alternatives.
