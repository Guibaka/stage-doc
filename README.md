# Stage INFO4 - Language Design for Linux I/O Scheduling
## Stagiaire
Guillaume Vacherias

## Tuteur de stage
Nicolas Palix

## Professeur référent
Vincent Danjean

## Note de compréhension
Les notes de compréhension se trouve dans [note.md](https://github.com/Guibaka/stage-doc/blob/main/note.md)

## Suivi de stage
### 07/05/2021
* Compréhensions sur le fonctionnement des fichiers parser.mly et lexeur.mll. La documentation est fourni [ici](https://caml.inria.fr/pub/docs/oreilly-book/html/book-ora107.html). Monsieur Palix nous a notamment donner plus d'explication sur ces derniers
* Abord sur comment le code-C généré par le compilateur est relié avec linux-kernel. J'ai pu aussi comprendre comment l'ajout d'une nouvelle politque d'ordonnancement est gérer. Ceci utilise les pointeurs de fonctions définit dans la structure *struct ipanema_policy* de ipanema.h. Elle permet selon la politque choisis de choisir la bonne fonction lors d'une transition d'état.
* J'ai encore du mal à saisir tous l'ast de l'ordonnanceur mais j'ai pu comprendre grâce à Monsieur Palix que avec les .ml du répertoire compiler, on effectue plusieurs transformation sur notre ast. 
* Pour la prochaine fois il sera nécessaire de poursuivre sur la compréhension du compiler (regarder verifier/automaton.ml) ainsi que les fonctions définis dans ipanema.c

Réponse aux questions de la séance dernière : 
* Leon code est système de vérification pour le language Scala. Le répertoire leon_generator est encore en développement. 
* Il est nécessaire d'avoir un fichier dans laquelle se situe un *authorized_key* dans le .ssh afin de pouvoir installer les paquets avec sudo.


### 06/05/2021
* Lecture du code sur le compilateur en particulier le fichier main.ml ainsi que les AST utilisé (scheduler, event hanlder, ...)
* Déploiement d'une image avec le script *deploy* de Monsieur Palix. Cependant je n'ai pas les droits d'installer les packages nécessaire pour la compilation. 
* Lecture de la documentation sur comment créer son propre image sur Grid5000

Concernant la compréhension du compilateur, j'ai du mal à comprendre ce qu'est le *Leon code* système de vérification pour le language Scala ?. Dans ce cas à quelle moment le language Scala intervient-il ? .

Dans la partie d'AST. Chaque *type* de notre AST est associé un identificateur définit dans le module Object de objects.ml. Il faudra que je regarde la partie compilateur et parser.

### 05/05/2021
* Obtention d'identifiant Grid5000 et premier abord sur le déploiement d'une image.
* Compréhension des différentes politques d'ordonnancement implémentées avec Bossa ainsi que Ipanema 
* Compréhension de l'utilisation du langage Ipanema sur la création d'une politque d'ordonnancement
* Poursuite de la lecture de la documentation 

### 04/05/2021
Rendez-vous en visio-conférence avec Victor Malod et Monsieur Palix : 
* Eclaircissement sur le fonctionnement **IPANEMA** en particulier de la classe SaaKM.
* Exemple détaillé concernant la création d'image système 
* Exemple d'erreur récurente lors du déploiement d'image sur un noeud
* Poursuite de la lecture de la documentation

Explication sur la partie SaaKM : 
Le fonctionnement consiste à itérer sur un ensemble de politique d'ordonnancement afin de trouver un runnable thread. Utilisé pour relier les modules généré par Ipanema au Linux-kernel.

Concernant l'erreur récurente sur le déploiement d'image, il se peut que le nom du noyau soit modifié créant ainsi un problème d'incohérence avec la version du noyau utilisé. Il est alors nécéssaire de vérifier le nom du noyau dans les configurations de l'image avec : 
```bash=
kaenv3 -p debian10-x64-nfs
```

Pour la sauvegarde d'une image on utilise la commande suivante : 
```bash=
tgz-g5k
```

En cas de changement d'un kernel antécédent à un kernel récent, la commande suivant permet de mettre les nouveaux paramètres à leurs valeurs par défaut : 
```bash=
make olddefconfig
```

Lors de la création d'image, il est important de **noter tous les changements effectué**.

Pour la création d'environnement sur Grid5000, on pourra se référer : 
* https://www.grid5000.fr/w/Advanced_Kadeploy
* https://www.grid5000.fr/w/Environment_creation (nécessite les identifiants de login que je n'ai pas encore reçu)

### 03/05/2021
Rendez-vous en visio-conférence avec Victor Malod et Monsieur Palix : 
* Demande d'inscription et introduction à la plateforme Grid5000
* Configuration ssh pour accéder à Grid5000
* Explication concernant les outils utilisé : (attribution d'un noeud du serveur, créer une image sur un noeud ...)

Premier abord sur la compréhension du **DSL IPANEMA** voici les documents utilisé:
* [Thread Scheduling in Multi-core Operating Systems](https://hal.archives-ouvertes.fr/tel-02977242/file/these_archivage_3100161.pdf)
* [Provable Multicore Schedulers with Ipanema:Application to Work Conservation](https://hal.inria.fr/hal-02554342/document)
* [Ipanema : un langage dédié pour le développementd’ordonnanceurs multi-coeur sûrs](https://hal.sorbonne-universite.fr/hal-02111160/document)
