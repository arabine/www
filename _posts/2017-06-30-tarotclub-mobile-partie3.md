---
layout: post
title: "Version mobile de TarotClub - partie 3"
date: 2017-07-10 18:43:00 -0700
comments: false
published: true
category:
- TarotClub
---

> Je développe depuis quelques années un jeu de tarot libre appelé TarotClub, créé à l'origine pour combler le manque de jeux de cartes sur Linux. Il est pour le moment
disponible uniquement sur les trois grands systèmes d'exploitation du monde PC. Cette série d'articles expliquera le cheminement, j'espère logique, du portage du jeu sur
mobile mais également jouable sur une page Web.


# POC (Proof of Concept)

N'étant pas particulièrement enthousiasmé par les solutions existantes (voir article précédent), je tente, pour Android tout d'abord, un essai complet sans utilisation de framework. Voici le cadre du POC.

La première chose à faire est de se créer un code d'exemple mimant notre architecture. Pour le GUI, réalisons un "Hello, World" classique, l'affichage d'une primitive en WebGL. Nous utilisons la librairie Three.js pour également tenter l'usage d'une librairie tierce. Notre moteur se contentera, en C++, de créer un serveur TCP/IP et attendra la connexion d'un client en utilisant le protocole WebSocket (encore un truc à tester).

Les données d'entrées sont donc :
  * Un code d'exemple de développement natif (NDK), appel d'un code C à partir de Java
  * Un code d'exemple d'utilisation du composant WebView (navigateur embarqué dans les Android)
  * Un max de forums et autres entrées Stackoverflow pour activer et tweeker notre application (accès à des librairies Javascript, activation du Websocket ....)

L'architecture visée est la suivante :

![tarotclub]({{ site.url }}/assets/articles/tarotclub-mobile-partie3/tarotclub_android.png)

Normalement, cette architecture nous permttra de garder au moins 90% de code similaire entre les plateformes. Le code dédié à une plateforme sera donc :
    * Du code pour initialiser le widget principal, un WebView
    * Du code du côté C/C++ pour encapsuler les librairies ICL et TarotCore
    * Tout un tas de fichiers de configuration (XML, gradle ...) afin de paramétrer convenablement le package généré
    * L'empaquetage des icônes et autres ressources

Concernant la plateforme d'Apple, un peu plus obscure pour moi ne possédant pas d'appareils de la marque, devrait se dérouler convenablement. Le nouveau langage maison, Swift, permet d'appeler du code C directement !

# Etape 1: compilation de TarotCore et ICL

Je pars du programme d'exemple du NDK fourni par Google appelé "hello-libs". Après compilation, impossible de lancer l'émulateur (sous Ubuntu 16.04). Je tente deux choses, je ne sais pas lequel fcontioone mais je l'écris pour l'histoire :

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

Bingo ! Notre code C++ avancé compile et la librairie semble bien liée à l'exécutable. On bouge notre projet dans l'arboresence du dépôt TarotClub et on le renomme proprement. Maintenant, on continue le travail laborieux d'ajout du code source complet d'ICL et de TarotClub.

# Etape 2: un serveur de fichiers HTTP

Comme nous l'avons dit en introduction, le but est de limiter le code dédié à chaque plateforme. Le stockage des ressources étant complètement propriétaire, j'ai décidé de créer mon propre serveur de fichiers en HTTP. Quelque chose de très limité bien entendu, rien à voir avec un serveur Web complet, même embarqué du style Mongoose. Ma librairie ICL possède déjà une classe TCPServer à tout faire, j'ajoute
j'ajoute quelques lignes de code pour encoder et décoder des en-têtes HTTP et c'est presque tout.

Les fichiers HTML et associés (CSS, Javascript...) sont passés dans une moulinette générant des tableaux constants en langage C: les fichiers sont donc liés au binaire final. C'est une pratique courante dans l'embarqué, mon domaine d'origine.

```
void ReadData(const tcp::Conn &conn)
{
    std::string resource = Match(conn.payload, "GET (.*) HTTP");

    std::cout << "Get file: " << resource << std::endl;

    if (resource == "/")
    {
        resource = "/index.html";
    }

    const char *data;
    const char *mime;
    size_t size;

    data = find_embedded_file(resource.c_str(), &size, &mime);

    std::string output;
    if (data != NULL)
    {
        std::cout << "File found, size: " << size << " MIME: " << mime << std::endl;

        std::stringstream ss;

        ss << "HTTP/1.1 200 OK\r\n";
        ss << "Content-type: " << std::string(mime) << "\r\n";
        ss << "Content-length: " << (int)size << "\r\n\r\n";
        ss << std::string(data, size) << std::flush;

        output = ss.str();
    }
    else
    {
        std::stringstream ss;
        std::string html = "<html><head><title>Not Found</title></head><body>404</body><html>";

        ss << "HTTP/1.1 404 Not Found\r\n";
        ss << "Content-type: text/html\r\n";
        ss << "Content-length: " << html.size() << "\r\n\r\n";
        ss << html << std::flush;

        output = ss.str();
    }
    tcp::TcpSocket::SendToSocket(output, conn.peer);
}
```

Et voilà, notre serveur de fichiers est réalisé en 42 lignes, sans faire exprès. Qui a dit que le C++ était verbeux ?

# Etape 3: Paramétrage du WebView

Après avoir ajouté le composant WebView dans le fichier XML 'activity_main.xml', on l'initialise et on appelle l'adresse de notre serveur
de fichiers:

```
public class MainActivity extends AppCompatActivity {


    private WebView mWebView;

    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initialize();

        mWebView = (WebView) findViewById(R.id.activity_main_webview);
        mWebView.clearCache(true);
        mWebView.getSettings().setJavaScriptEnabled(true);
        mWebView.loadUrl("http://127.0.0.1:8000");
    }

    static {
        System.loadLibrary("tarotclub_server");
        System.loadLibrary("icl");
    }

    public native void initialize();
}
```
# Essais

A l'heure de l'écriture de cet article l'émulateur Android ne permet pas d'afficher du WebGL dans un composant WebView. Essayons donc directement sur le téléphone :

![tarotclub]({{ site.url }}/assets/articles/tarotclub-mobile-partie3/screenshot_android.png)

Par contre, problème, le touch screen ne semble pas fonctionner, je ne sais pas pourquoi pour le moment. J'éditerai cet artcile lorsque j'aurai l'explication technique !

# Conclusion

Cest terminé ! Le code dédié à Android a été réduit à son plus strict minimum. Bien sûr il grossira un peu avec l'ajout de paramétrages
et peaufinages divers mais le reste, tout le jeu, sera développé en Javascript uniquement.

Pour le stockage des donnés (paramétrage), nous utiliserons le stockage offert par le navigateur, limité en taille mais suffisant pour notre application (essentiellement des fichiers de configuration en JSON).
