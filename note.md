# Familiarisation du Stage

## IPANEMA
La compréhension est basé sur le document suivant : 
[Thread Scheduling in Multi-core Operating Systems](https://hal.archives-ouvertes.fr/tel-02977242/file/these_archivage_3100161.pdf).

### Origine
Ipanema est une "extension" de Bossa qui permet un scheduling à multi-coeur. La raison pour Ipanema est de fournir un logiciel permettant d'aider les end-user de développer des nouveaux ordonnanceur puisque ceci nécessite beaucoup de connaissance concernant *scheduling theory*, *low-level kernel development* et *hardware architecture*.

Problème *frequency inversion* : 
* schduler is unaware of CPU frequency in its thread placement decision 
* high frequency cores are idle and low frequency cores are busy


### Fonctionnement 
Ipanema offre un niveau d'abstraction qui permet au end-user de se focaliser sur la politque d'ordonnancement sans se soucier la programmation du noyau bas-niveau.


Abstraction : Définie les comportements des objets de l'ordonnanceur et facilite la preuve des propriétés. Permet aux développeur de savoir ce qu'il peut et peut pas faire. Ipanema fournit 4 abstractions principales : 
* thread (définie par automate)
* core (définie par automate)
* topology (permet aux politiques d'ordonnancement de gérer les threads en conservant Simultaneous Multithreading SMT = cores of same domain share computing hardware, cache localité = last level cache et l'arichtecture Non-Uniform Memory Access NUMA)
* load balancing (evénement particulier concernant plusieurs processe voir page 49) 


![fonctionnement_ipanema](https://i.imgur.com/vlvZzKI.png).

Le langage Ipanema est un *event-based language* séparé en 2 partie : 
* core event
* thread event

Utilisation Ipanema : 
* Définie attribut thread et core
* Définie event handlers

### Politique d'ordonnancement
#### CFS-like policy 

#### ULE-like policy

### Propriété d'ordonnanceur
Work conservation est une propriété importante : Si un coeur du processeur est surchargé alors il a plus de 1 thread. Si le coeur du processeur contient aucun thread alors il est dans l'état IDLE.

## Gitlab
**ipanema-kernel** : Répo contenant linux-kernel avec ipanema intégré. On pourra retrouver les intégrations [ici](https://gitlab.inria.fr/ipanema/ipanema-kernel/-/tree/linux-4.19-ipanema/kernel/sched/ipanema).
On pourra se repérer dans le répertoire grâce au fichier *Documentation/00-Index*

**ipanema-compiler** : Répo contenant le compilateur Ipanema et le code-c généré par le compilateur. 

**ipanema-benchmarks** : Répo permettant d'effectuer des évalutaions de performance sur plusieurs politque d'ordonnoncement respectant les propriétés du scheduling. CWC (Concurret Work Conservation).

**ipanema-dbg** : Répo contenant le script *qemu-run-externKernel.sh* qui exécute **qemu-system-x86_64** avec les options et configurations corrects. 

**nas-ipanema** : Répo contenant les résultats de tests ?

