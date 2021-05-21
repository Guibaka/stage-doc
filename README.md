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
### 21/05/2021
Rendez-vous avec Monsieur Palix et Victor à l'IMAG pour nous attribuer les badges ainsi que de faire une petite visite du laboratoire : 
* J'ai poursuivi la création d'environnement avec kameleon. Cependant, kameleon utilise des package obsolète (polipo). Je pourrais suivre ces 2 liens pour continuer la création de l'environnement pour la prochaine fois : [lien1](https://www.grid5000.fr/w/User:Pneyron/PMEM-environment#Deploy_the_environment) [lien2](https://www.grid5000.fr/w/User:Pneyron/ARM64-custom-environment)
* J'ai essayé de lancer les benchmarks fournis dans le répo [benchmark](https://gitlab.inria.fr/ipanema/benchmarks).  Le benchmark nécessite d'avoir toutes les potiliques d'ordonnancement dans le fichier /proc/ipanema/policies
* Il faudra que je regarde comment fonctionne le benchmark 

Note important : 
* Pour les modules générés par le compilateur, il faut utilisé la commande insmod pour les intégrer.
* Pour les modules compilé avec le ipanema-kernel, il suffit d'utiliser la commande modprobe pour les intégrer


### 20/05/2021
* Premier abord sur la création d'environnement avec kameleon
* On pourra retrouver la documentation [ici](https://www.grid5000.fr/w/Environments_creation_using_Kameleon_and_Puppet#Modifying_an_image)
* Utiliation de la commande ipastart avec les modules généré par le compilateur
* Il faudra regarder l''utilisation des benchmarks et régler les multiples déclaration des fonctions dans [sched/ipanema](https://gitlab.inria.fr/ipanema/ipanema-kernel/-/tree/linux-4.19-ipanema/kernel/sched/ipanema).
### 19/05/2021
Rendez-vous en visio-conférence avec Monsieur Palix et Victor : 
* Installation ipanema-kernel réussi. Il fallait prendre le fichier config utilisé du debian10-x64-nfs pour ensuite faire un *make olddefconfig*. 
* Premier abord sur la commande ipastart et comment charger les modules intégrés (politque d'ordonnancement)
* Pour les politiques d'ordonnancement situés dans [sched/ipanema](https://gitlab.inria.fr/ipanema/ipanema-kernel/-/tree/linux-4.19-ipanema/kernel/sched/ipanema), il n'est pas possible de toutes les intégrer lors du *make config* (problème : multiple function definition. Les autres politiques sont implémenté à partir du fichier [cfs.c](https://gitlab.inria.fr/ipanema/ipanema-kernel/-/tree/linux-4.19-ipanema/kernel/sched/ipanema/cfs.c))
* Il faudra pour la prochaine fois regarder comment fonctionne **kameleon** afin de créer mon propre environnement, regarder l'utilisation des benchmarks ainsi que de régler les multiples déclaration des fonctions

### 18/05/2021
* Lecture du code implémenté dans [sched/ipanema](https://gitlab.inria.fr/ipanema/ipanema-kernel/-/tree/linux-4.19-ipanema/kernel/sched/ipanema)
* Continuer l'installation. Cette fois j'ai utilisé l'image généré lors du make, mais cela ne fonctionne pas non plus


### 17/05/2021
* Compréhension des tests utilisés sur le compilateur
* En attendant la réponse de Monsieur Palix, j'ai poursuivi l'installation de l'ipanema kernel


Rendez-vous en visio-conférence avec l'équipe travaillant sur ipanema : 
* Différent résultat sur les NAS-benchmark suite

Lecture de la documentations : 
* CFS is the vanilla CFS scheduler of Linux v4.19, used as a baselinecomparison. It is the only tested scheduler directly written in C.
* CFS-CWC is  a  simplified  and  slightly  modified  version  of  thealgorithm of CFS proven to be work-conserving
* CFS-CWC-FLAT is the same algorithm except that it does not ac-count for the hardware topology.
* ULE and ULE-CWC are  simplified  versions  of  the  scheduler  ofFreeBSD, ULE. The latter is slightly modified and proven to bework-conserving



### 13/05/2021
* Poursuite de l'installation de l'ipanema kernel (Lors du reboot, start load kernel modules ne fonctionne pas puisque le fichier /etc/selinux/targeted/policy/policy.31 n'existe pas ?)
* Compréhension sur la génération du code-C par le compilateur
* Le fichier config généré par make menuconfig prend pas les bonnes configuration



### 12/05/2021
Rendez-vous en viso-conférence avec Monsieur Palix, pour l'installation ipanema-kernel : 
* Echec d'installation 
* Utilisation du grub-reboot '1>2' pour rebooter sur le bon kernel version voulu
* Problème lors du reboot : [FAILED] Failed to start Load Kernel Modules

De mon côté j'ai tenté d'utiliser make menuconfig pour en sélectionnant les nouvelles paramètres d'ipanema ainsi que de forcer load modules.
Cette journée n'a pas été productive car l'installation n'est toujours pas réussi

### 11/05/2021
* Poursuite de l'installation de ipanema-kernel qui n'est pas encore terminé
* Sauvegarde de l'envirionnement créé
* Compréhension du code situé dans [tools/ipanema](https://gitlab.inria.fr/ipanema/ipanema-kernel/-/tree/linux-4.19-ipanema/tools/ipanema)
* Modification du script *deploy* pour charger l'envirionnement



### 10/05/2021
Rendez-vous en visio-conférence avec l'équipe travaillant sur ipanema. Nous avons principalement discuter sur les optimisation implémenté sur la répertation des processus sur un processeur. Voici ce que nous avons abordé durant ce visio : 
* Intra-socket algorithm sur Linux5.9. Il existe 2 façon. 1) Chercher tous les cores d'un socket à partir du target pour trouver le core et son hyperthread en état IDLE. 2) Chercher sur les N cores à partir du target pour trouver celui en état IDLE   (problem : N n'est pas work conserving)
* Autre optimisation. Garde un flag sd_llc_shared par socket indiquant s'il existe un core ayant la propriété idle
* Présentation des résultats avec NAS-benchmark 
Grâce à cette séance, j'ai pu comprendre l'objectif de la recherche.

Rendez-vous en visio-conférence avec Victor Malod et Monsieur Palix : 
* Succès de la création d'une image à partir de l'image debian10-x64-nfs. 
* Compilation de l'ipanema-kernel, cependant je n'ai pas sauvegarder l'image avec la commande *tgz-g5k*. Il va falloir que dans refaire les mêmes manipulation. 
* Clarification des tâches à accomplir durant ce stage. 

Concernant les tâches durant ce stage : 
1. Il existe dans le répertoire [kernel/sched/ipanema](https://gitlab.inria.fr/ipanema-public/ipanema-kernel/-/tree/linux-4.19-ipanema/kernel/sched/ipanema) des politque d'ordonnancement implémenté à la main. 
2. Le compilateur Ipanema génére du code C selon la politique implémentée avec le DSL d'Ipanema.
3. Effectuer des évaluations de performance sur 1 et 2 et montrer que le code du générateur est plus performant
4. Effectuer une migration à linux-5.9 à partir de la branche linux-4.19-ipanema
5. Ecrire d'autre politque d'ordonnancement avec le Ipanema

Il est important pour demain de réussir d'installer kernel-ipanema pour tester la commande *ipastart*.
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
