---
layout: post
title: "Version mobile de TarotClub - partie 3"
date: 2017-06-30 18:43:00 -0700
comments: false
published: false
category:
- TarotClub
---

> Je développe depuis quelques années un jeu de tarot libre appelé TarotClub, créé à l'origine pour combler le manque de jeux de cartes sur Linux. Il est pour le moment
disponible uniquement sur les trois grands systèmes d'exploitation du monde PC. Cette série d'articles expliquera le cheminement, j'espère logique, du portage du jeu sur
mobile mais également jouable sur une page Web.


# POC (Proof of Concept)

N'étant pas particulièrement enthousiasmé par les solutions existantes (voir article précédent), je tente, pour Android tout d'abord, un essai complet. Voici le cadre du POC.

La première chose à faire est de se créer un code d'exemple mimant notre architecture. Pour le GUI, réalisons un "Hello, World" classique, l'affichage d'une primitive en WebGL. Nous utilisons la librairie Three.js pour également tenter l'usage d'une librairie tierce. Notre moteur se contentera, en C++, de créer un serveur TCP/IP et attendra la connexion d'un client en utilisant le protocole WebSocket (encore un truc à tester).

Les données d'entrées sont donc :
  * Un code d'exemple de développement natif (NDK), appel d'un code C à partir de Java
  * Un code d'exemple d'utilisation du composant WebView (navigateur embarqué dans les Android)
  * Un max de forums et autres entrées Stackoverflow pour activer et tweeker notre application (accès à des librairies Javascript, activation du Websocket ....)

## Etape 1: compilation de TarotCore et ICL

Je pars du programme d'exemple du NDK fourni par Google appelé "hello-libs". Après compilation, impossible de lancer l'émulateur. Je tente deux choses, je ne sais pas lequel fcontioone mais je l'écris pour l'histoire :

{% highlight shell linenos %}
sudo apt-get install lib64stdc++6:i386
sudo apt-get install mesa-utils
{% endhighlight %}

Puis j'édite le script de lancement de AndroidStudio (~/android-studio/bin/studio.sh) et j'ajoute la ligne suivante avant toute autre instruction :

{% highlight shell linenos %}
export LD_PRELOAD='/usr/lib/x86_64-linux-gnu/libstdc++.so.6'
{% endhighlight %}

Maintenant tout se passe bien j'arrive à lancer l'application sur émulateur. Maintenant, on va essayer d'ajouter le code source d'ICL, la librairie d'utilitaires utilisée par TarotClub.

Il y a deux fichiers à modifier : le projet Gradle, essentiellement pour lui dire quelle librairie standard C++ nous allons utiliser et pour passer des paramètres à CMake. Puis, on va modifier CMake pour lui
indiquer quels codes sources modifier.

Dans le fichier 'build.gradle', on va utiliser la librairie libc++ et le compilateur Clang, tous deux offrent un très bon support de C++0x14 :

```
cmake {
                arguments '-DANDROID_PLATFORM=android-21',
                          '-DANDROID_TOOLCHAIN=clang',
                          '-DANDROID_STL=c++_shared'
                // Enables RTTI support.
                cppFlags "-frtti"
                // Enables exception-handling support.
                cppFlags "-fexceptions"
                // For TarotClub and ICL, specify the target platform
                cppFlags "-DUSE_UNIX_OS"
            }
```

Enfin, notre CMakeLists.txt ressemble à cela, nous avons ajouté ici qu'un seul fichier pour nos tests:

```
#[[
 CMake file to build ICL and TarotClub source files on Android
]]

cmake_minimum_required(VERSION 3.4.1)

# New path definition
set(ICL_DIR ~/git/tarotclub/lib/icl)

# Switch on the C++0x14 standard for the compiler
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED on)

add_library(hello-jnicallback SHARED
            hello-jnicallback.c)

# Add the ICL library and specify the source files to build
add_library(icl SHARED ${ICL_DIR}/util/Util.cpp)


# Include libraries needed for TarotClub lib
target_link_libraries(hello-jnicallback
                      android
                      icl
                      log)

# End of Cmake file
```

Bingo ! Notre code C++ avancé compile et la librairie semble bien liée à l'exécutable. On bouge notre projet dans l'arboresence du dépôt TarotClub et on le renomme proprement. Maintenant, on continue le travail laborieux d'ajout
 du code source d'ICL et de TarotClub.



# Conclusion

## Liens
