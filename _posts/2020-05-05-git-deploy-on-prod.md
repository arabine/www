---
layout: post
title: "Déployer une application web en production"
date: 2020-25-05 8:55:00 -0120
comments: false
published: true
category:
- git
- web
- production
---

> La méthode de mise en production proposée ici est une méthode parmi d'autres, je ne prétend pas être un expert en la matière alors il y a peut-être des contre-indications ou mauvaises pratiques que je ne connais pas. Toujours est-il que j'aime les solutions simples et sécurisées, adaptées à mon modèle de développement.

# Création d'un nouveau dépôt distant "bare"

Ce dépôt sera celui où l'on va "pusher" notre code pour enclencher le déploiement automatique de l'application. C'est un dépôt git un peu spécial, appelé "bare" sur lequel vous ne pourrez pas travailler. Il ne contiendra que le répertoire ".git" et c'est tout, aucun fichier de code.

Tapez les commandes suivantes :

```shell
cd
git init --bare prod.git
git clone prod.git/ ./mywww.git
```

Ajoutez ce dépôt à comme nouveau dépôt distant à votre dépôt de développement :

```shell
git remote add prod ssh://test@vps1234.ovh.net/home/test/prod.git
```

Vous devriez maintenant observer ce changement au niveau de votre développement local :

```shell
git remote -v
live    ssh://test@vps1234.ovh.net/home/test/prod.git (fetch)
live    ssh://test@vps1234.ovh.net/home/test/prod.git (push)
origin  https://gitlab.com/monprojet/mywww.git (fetch)
origin  https://gitlab.com/monprojet/mywww.git (push)
```

Poussez une première fois votre code :

```shell
git push prod master
```

Cette première étape est terminée, maintenant vous disposez d'un dépôt dédié à la livraison en production.

# Script de déploiement

Maintenant, nous allons créer un script dans ce dépôt de production qui sera enclenché dès que du code est pushé. Cela s'appelle un hook dans le monde git. Pour cette action particulière, il doit s'appeler 'post-receive' et doit avoir le flag exécutable.

```shell
cd prod.git/hooks
touch post-receive
chmod +x post-receive 
nano post-receive 
```

Voici le contenu de mon script. Là, c'est complètement dépendant de votre application, ici j'utilise un back-end NodeJS et un front-end Vue.js

```shell
echo ‘post-receive: Triggered.’
cd /home/test/mywww.git
echo ‘post-receive: git check out ...’
git --git-dir=/home/test/prod.git --work-tree=/home/test/mywww.git checkout master -f
echo ‘post-receive: npm install...’

# Prepare front-end sources
cd client
npm install
echo ‘post-receive: building’
npm run build

# Prepare server sources
cd ../server/
npm install

# Restart server (depending of your supervisor)
# ...

echo ‘post-receive: → done.’
```

N'oubliez pas de redémarrer votre serveur à la fin de la mise à jour, cela dépend de votre moyen de superviser votre processus de serveur (pm2, systemd ...).
