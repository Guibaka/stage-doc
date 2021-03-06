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
### 19/07/2021-23/07/2021
* L'utilisation des variables implémentés dans la fonction **steal_for_dom** fonctionne (le code C généré est correct)
* J'ai commencé à implémenter l'utilisation des variable en cache pour la fonction **migrate_from_to** 

Pour généré le code C  dans la fonction **migrate_from_to**, ce dernier appel la fonction **pp_seq2** avec l'argument *migrstmt* de type **stmt** (statement). L'idée est donc de changer les variables généré par nos variable lb_env. Pour cela on doit donc modifier l'ast dans l'argument *migrstmt* 

Fichier ipanema.ml : 
* Ajout d'un flag lcache déterminant si on utilise le champs cload d'un core

Fichier ast.ml : 
* Ajout de la fonction **convert_to_cache** convertit les noms des variables par les noms des variables d lb_env
* Ajout des fonctions auxilière utilisée **convert_exp** et **select_field**


Fichier steal2c.ml : 
* Modification pour utiliser les variables lors de la génération du code C pour la fonction **steal_for_dom**
* Modification pour utiliser les variables lors de la génération du code C pour la fonction **migrate_from_to** (en cours)

Question : Concernant l'implémentation des variables en cache, est-ce qu'il serait plus astucieux de déterminer les variables à mettre en cache au moment de la complétion d'ast (fichier compile.ml) ou au moment de générer le code C  ?

Réponse : Sur le long terme, la phase de compilation est mieux, car plus souple et tu peux faire plusieurs passes sur l'arbre facilement. (peut-être plus couteux car on doit parcourir tout l'ast)
### 12/07/2021-16/07/2021
* Implémentation des variables en cache lb_env (génération de la structure *struct* <politique_ipa> _lb_env fonctionne) : 
```c=
struct lb_env{
 /* cache environments variable */
 int busiest_grp_cload; // Internal
 int thief_grp_cload; // Internal
 int busiest_grp_runnable; // Internal
 int thief_grp_runnable; // Internal
}
```

Pour le choix de la génération du code C avec les variable en cache, on utilise un flag **cache** dans le fichier ipanema.ml.

J'ai implémenté différente fonction en suivant le modéle de génération du code C du compilateur.

Fichier bossa2c.ml : 
* Définition de la fonction *pp_cacheddecls* pour générer les champs de notre structure qui appel la fonction *pp_decls*
* L'appel de cette fonction est effectué dans le point d'entrée de ce fichier
* Modification de la fonction **pp_ipa_struct** pour faire appel à la fonction G.pp_cache_struct_cs défini dans génaux.ml
* Question : Le module plazy sert est-il nécessaire dans l'implémentation de la structure lb_env ?

Fichier objects.ml :
* Ajout d'un champ *CACHE of identifier* dans le type **typ**
* *CACHE of identifier* permettra de générer le code c puisque identifier est de type (string * int) où int permet d'intentifier l'objet. 

Fichier genaux.ml : 
* Définition d'une fonction **pp_cache_struct_cs** permettant de générer le nom de la structure selon le nom de la politique implémenté

* Pour la prochaine fois il faudra utiliser cette structure dans les fonctions **migrate_from_to** et **steal_for_dom** 
### 05/07/2021-09/07/2021
* Modification du fichier steal2c pour générer le code C utilisant la fonction *migrate_from_to* pour la migration des threads. Cette modification affect les politique ule_ipa et ule_bsd_rand_ipa. On remarquera qu'il y a une hausse de performance en temps d'exécution pour la politique ule_ipa
* J'ai commencé à lancer les benchmarks avec la commande hyperfine. Plus de détail sur la commande [ici](https://github.com/sharkdp/hyperfine). 
* Modification du script deploy pour la réservation de noeud après 19h afin de faire tourner les benchmarks 
* On pourra référer au fichier [steal2c.ml](https://gitlab.inria.fr/ipanema/compiler/-/blob/guillaume/compiler/generator_new/steal2c.ml). D'après la politique ule_ipa.ipanema nous avons : 
```shell=
steal = {
			can_steal_core(core src, core dst) {
			    dst.balanced ? false :
			    src.balanced ? false :
				src.cload > dst.cload
			} => stealable_cores
			do {
			   select_core() {
				first(stealable_cores order = { highest cload })
			   } => busiest
			   steal_thread(core here, thread t) {
				here.balanced = true;
				if(busiest.cload - here.cload >= 2) {
					busiest.balanced = true;
					if(t.prio == REGULAR)
						t => here.timeshare;
					else
						t => here.realtime;
				}
			   } until ( here.balanced )
			} until (true)
		}

```

* Il faudra pour la semaine prochaine attaquer la partie variable en cache

La modification permet d'obtenir le code C suivant pour la migration des threads dans un core idle : 
```c=
/* Special case for ule */
        do{
        	/* Step 4 : select_core*/
                busiest = select_core(policy, NULL, stealable_cores);
                if (!busiest)
                	goto free_mem;
                
                /* Step 5: steal_thread */
                migrate_from_to(policy, busiest, core_30);
                /*Post Operation in core iteration*/
                	cpumask_clear_cpu(busiest->id, stealable_cores);
                
        }while (!(cpumask_empty(stealable_cores) || busiest->balanced));
```

### 28/06/2021-02/07/2021
* Petite modification sur le sript de comparaison sur le code C généré. J'ai remarqué que ma modification fonctionnait que sur une politique spécifique (cfs_ipa) alors que les poliques telles que (ule_ipa) présentent le même soucis
* Avec l'aide de Monsieur Palix, j'ai pu localisé le problème du déférencement NULL. Cependant, j'ai fait certain changement évitant l'apparition du probème de pointeur NULL mais cela n'améliore pas la situation. 
* Il sera envisageable de débugger grâce à Qemu pour regarder en détail l'éxécution du module
* Rédaction d'un script pour reporter les résultats des benchmarks (NAS-benchmark)

### 24/06/2021
* Monsieur Palix m'a donné une la commande **addr2line** permet de convertir l'adresse fournis dans la trace d'erreur. 
* Le bug provient de la fonction **ipanema_cfs_ipa_attach** qui renvoie un boolean. Je ne vois pas à quel moment cette fonction est appelé. (C'est une transition du ipanema_state ?)
* Modification du script pourcomparer la différence entre les code C généré avant la modification du fichier steal2c et après
* Il fraudra aussi trouver un moyen pour interrompre le benchmark lorsque le kernel est en soft/hard lock

### 23/06/2021
* Le bug semblerait apparaître dans la fonction ipanema_cfs_ipa_unblock_prepare. Il se peut que find_idlest_cpu_group ne trouve aucun group et on choisi un core où un thread est bloqué 
* J'essaye de faire des kprint mais ce n'est pas efficace. Faudrait trouver un moyen pour debugger les fichier module .ko

### 22/06/2021
Journée de travail au laboratoire de l’IMAG : 
* Fix d'un problème de parenthèsage sur le code C généré
* Il existe toujours le même problème du code C généré de la politque cfs_ipa (kernel soft/hard lock)

Voici ce que affiche **sudo dmesg** : 
```shell=
BUG: unable to handle kernel NULL pointer dereference at 0000000000000030
```


### 21/06/2021
Rendez-vous en visio-conférence avec l’équipe travaillant sur ipanema : 
* Implémentation fini sur steal2c. 
* Rédaction du script pour comparer la différence après le changement

### 18/06/2021
Journée de travail au laboratoire de l’IMAG : 
* Fini l'implémetation. Il manque à tester les politiques générées. 
### 17/06/2021
* Modification du fichier steal2c.ml. J'utilise un flag pour voir si la condition est générée. Si oui, on remplace *continue* 
* L'idée est de ne pas itérer sur les ensembles de groupes et de core
### 16/06/2021
* Modification du fichier steal2c.ml dans le répertoire generator_new du compilateur. La condition *while(unsatisfaible)* n'existe plus. Cependant, il manque encore : 
```c=
goto free_mem // replace with continue
```

### 15/06/2021
Journée de travail au laboratoire de l’IMAG : 
* Validation du code cfs_ipa.c qui est censé être généré par le compilateur. Manque plus qu'à implémenter 
* Monsieur Palix m'a expliqué l'utilisation des variables en cache dans le fichier cfs.c
```c=
env.thief_grp_cload = grp_load(thief_group);
env.thief_grp_runnable = runnable(thief_group);
```
* On pourra envisager à implémenter l'utilisation des variables en cache soit avec **flag** ou avec un mot clé dans le compilateur pour la génération de code
* On pourra aussi envisager à écrire un script pour reporter plus facilement les résultats des benchmarks

### 11/06/2021
Journée de travail au laboratoire de l’IMAG : 
* Monsieur Palix m'a aidé a cerner le problème. Il va falloir donc regarder dans le fichier steal2c.ml (fichier générant le code C par le compilateur). 
* Le générateur ne prends pas en compte les **until** sans le keyword **do**. 
* Il va falloir que je rajoute dans ce fichier le cas non géré
* J'ai pu modifier le code C généré pour avoir une meilleure compréhension du fontionnement hierachique entre *domain*, *groups*, *core*, *threads*. 
* Il serait intéressant d'ajouter dans le fichier note.md les explications concernant la hierarchie des politique non **FLAT** 

### 10/06/2021
* J'ai comparé le code cfs_ipa et cfs_cwc_ipa. La seule différence observé : l'itération dans l'ensemble des *stealable_group* .
* Dans cfs_ipa.c il n'est pas censé avoir une itération sur l'ensemble des stealable_groups.

L'erreur pourrais être présent dans 3 cas : 
* code cfs_ipa écrit en DSL Ipanema
* le code généré par le compilateur 
* le parsing ne fonctionne pas correctement
### 09/06/2021
* Problème avec &pos->cpus_masks 
* Condition until génére n'est pas le même attendu que dans *cfs_ipa*
* J'ai regardé une exécution en mode ocamldebug pour le fichier cfs_ipa

### 08/06/2021
* Exécution en mode débug sous ocamldebug
* Potentiel problème dans fichier : compile_steal (voir compile_steal until) 
* Génére un *AST.Or* mais avec une condition à **false** 
* Il faudra regarder plus en détail

### 04/06/2021
* J'ai poursuivi la recherche de l'erreur. L'erreur provient du compilateur, qui crée une boucle while où la condition n'est jamais satisfaite : 
```shell=
while (!(bitmap_empty(stealable_groups, __sd->___sched_group_idx) ||true))
```
* Il faudra regarder le fichier [steal2c.ml](https://gitlab.inria.fr/ipanema/compiler/-/blob/master/compiler/generator_new/steal2c.ml) pour effectuer les modifications. 
* Il est aussi possible que l'erreur provient du code ipanema

### 03/06/2021
* Compréhension du code cfs_ipa.ipanema et le code généré cfs_ipa.c
* J'ai comparé avec les code cfs_cwc_flat.ipanema ainsi que cfs_cwc_flat.c. La différence étant la définition de la fonction steal(). En effet, dans cfs_ipa.ipanema, il définit un steal_group() en plus du steal_core() de cfs_cwc_flat.ipanema. 

### 02/06/2021
* J'ai modifié le script trace.sh ainsi que le Makefile dans ipanema-kernel/kernel/sched/ipanema. Les fichiers *.ko* pourra directement être intégré dans /lib/modules/($shell -uname)/kernel/kernel/sched/ipanema/ ainsi pour changer de politique d'ordonnancement un simple **modprobe** suffirait.
* J'ai continué à lancer des benchmark sur les politiques : *cfs cfs_wwc_flat ule* (politique d'ordonnancement écrit)
* Il faudra regarder le code cfs_ipa.c ainsi que cfs_ipa.ipanema

### 01/06/2021
* J'ai réalisé les benchmark sur les politiques : *cfs_cwc_flat_ipa	cfs_cwc_ipa cfs_wwc_flat_4ms_ipa	ule_ipa	ule_cwc_ipa* (polique d'ordonnancement généré par le compilateur)
* En essayant de réaliser un benchmark sur la politque cfs_ipa, le benchmark ne tourne plus. Il est possible que le code généré soit erroné. Il sera nécessaire pour la prochaine fois de jeter un coup d'oeuil

### 31/05/2021
Rendez-vous en visio-conférence avec l’équipe travaillant sur ipanema : 
* Risk condition : L'odre d'affectation des variables *ttw* et *number_running* que les fonctions *available_core()* et *load_balancing()* utilisent,  crée des IDLE core inattendu
* Concernant l'environnement utilisant Kameleon, j'ai envoyé un mail à Monsieur Neyron après avoir réessayer la création avec Monsieur Palix (en attente de la réponse).
* Première abord sur l'outil hyperfine. J'ai lancé des benchmark sur la politique cfs_cwc_flat_ipa généré par le compilateur. Selon la classe des benchs le temps d'exécution varie, ce qui est normal. 
* J'ai pris comme référence le tableau *Bench_result.odt* et j'ai reporté les résultats obtenu.  
### 28/05/2021
Journée de travail au laboratoire de l'IMAG : 
* Monsieur Palix m'a aidé sur la création d'environnement avec kameleon ainsi que les configurations pour la compilation du NAS-benchmark. 
* J'ai poursuivi la création d'environnement mais toujours le même problème après le démarrage de Qemu
* J'envisagerai de passer sur les benchmarks pour la prochaine fois
### 27/05/2021
* Lancement du nas-benchmark (NPB-3.3-OMP) sur les politiques généré par le compilateur. On pourra retrouver le code source du benchmark [ici](https://github.com/minsii/NPB-3.3/tree/master/NPB3.3-OMP). Cependant, il faudra que je regarde en détail les différents benchmarks (BT, SP, LU, CG, MG, FT, and EP) et à quoi correponsdent les *CLASS* lors de la compilation des benchs.
* J'ai regardé comment fonctionne le fichier [trace.sh](https://gitlab.inria.fr/ipanema/exp-results/nas-ipanema/-/blob/master/trace.sh). J'ai effectué quelque changement pour exécuter toutes les potiques générés par le compilateur.
* J'ai continuer sur la création d'environnement avec kameleon. Cependant, je rencontre une nouvelle erreur concernant l'étape *start_vm* lors du bootsrap.
### 26/05/2021
* Poursuite de la création d'envirionnement avec kameleon. Le problème était sur les fichiers tarball il faut les placer dans le répertoire public de notre home. Avec un path : 
```shell=
http://public.site.grid5000.fr/~username/file
```
* J'ai regardé la partie src du benchmark (fork, idle, monitor_tests). Il faudra pour la prochaine fois que je fasse des benchmarks sur les poitiques
* J'ai gardé une trace de l'installation dans le fichier [kameleon.md](https://github.com/Guibaka/stage-doc/blob/main/kameleon.md)

### 25/05/2021
* Poursuite de la création d'envirionnement avec kameleon.
* Le chargement de l'image ipanema.tar.zst ne fonctionne pas. Cependant, les images situant dans /grid5000/image/ chargent correctement

Utilisation des liens : 
* https://www.grid5000.fr/w/User:Pneyron/PMEM-environment
* https://www.grid5000.fr/w/User:Pneyron/ARM64-custom-environment

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
