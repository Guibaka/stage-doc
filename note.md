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


![automate_thread](https://i.imgur.com/17BadbB.png)

![automate_core](https://i.imgur.com/FmKcLoi.png)



Le langage Ipanema est un *event-based language* séparé en 2 partie : 
* core event
* thread event

Utilisation Ipanema : 
* Définit attribut thread et core
* Définit les event handlers

### Politique d'ordonnancement
#### CFS-like policy 
Utilise un arbre bicolore (arbre binaire de recherche équilibré). Chaque noeud de l'arbre représente un processus. Chaque noeud a un *temps d'exécution* (nanoseconde) et un *temps max d'exécution* (= temps_attente/nb_processus). 

1. noeud gauche est choisi (temps d'exécution plus faible)
2. si processus terminé alors supprimé le noeud
3. si processus atteind le *temps max d'exécution* interruption alors lui associ un *new temps exécution passé* et on l'insére dans l'abre
4. sélectionne le nouveau noeud gauche

#### ULE-like policy


### Propriété d'ordonnanceur
Work conservation est une propriété importante : Si un coeur du processeur est surchargé alors il a plus de 1 thread. Si le coeur du processeur contient aucun thread alors il est dans l'état IDLE.

## Polique implémenté en Bossa

* **Round-Robin** 
* **BVT** (Borrowed Virtual Time) : considére le temps de latence 
* **Earliest Deadline First** : Attribue une priorité à chaque requête en fonction de l'échéance de cette dernière, les tâches dont l’échéance est proche recevant la priorité la plus élevée. 
* **Rate Monotonic** :  attribut la priorité la plus forte à la tâche qui possède la plus petite période. Priorité avec une constanten statique.

Lien vers la documentation de Bossa [ici](http://bossa.lip6.fr/)

## Gitlab
**ipanema-kernel** : Répo contenant linux-kernel avec ipanema intégré. On pourra retrouver les intégrations [ici](https://gitlab.inria.fr/ipanema/ipanema-kernel/-/tree/linux-4.19-ipanema/kernel/sched/ipanema).
On pourra se repérer dans le répertoire grâce au fichier *Documentation/00-Index*

**ipanema-compiler** : Répo contenant le compilateur Ipanema et le code-c généré par le compilateur. 

**ipanema-benchmarks** : Répo permettant d'effectuer des évalutaions de performance sur plusieurs politque d'ordonnoncement respectant les propriétés du scheduling. CWC (Concurret Work Conservation).

**ipanema-dbg** : Répo contenant le script *qemu-run-externKernel.sh* qui exécute **qemu-system-x86_64** avec les options et configurations corrects. 

**nas-ipanema** : Répo contenant les résultats de tests ?

### Compilateur
Pour comprendre mieux comment le Compilateur Ipanema fonctionne. Voici la compréhension du main.ml : 

* Définit les fonctions permettants de vérifier l'AST renvoyé lors du parsing 
* Renvoie l'ast parsé par le parseur. Utilise aussi les parseurs de Bossa (de ce que j'ai compris)
* Utilise ast (scheduler) et ast2 (Handlers, Interface, Attack, Detach) 
* Génére du *Leon code* ? + *C code* + *WhyML code*
* Génére le Makefile du code C généré
* Fonction **pipeline build_ast** permet de réaliser les étapes mentionnés et retournent les ensembles d'ast.
* Utilise la notion du voyageur (ast se complète au fur et à mesure de chaque appel de parsing)


#### Ast
* L'ast de l'ordonnanceur est déclaré [ici](https://gitlab.inria.fr/ipanema/compiler/-/blob/master/compiler/types/ast.ml). 
* L'ast des *events handlers*, *interface*, *attach*, *detach* [ici](https://gitlab.inria.fr/ipanema/compiler/-/blob/master/compiler/generator_new/ast2c.ml)

#### Parser 
Exemple qui suit est tiré [ici](https://caml.inria.fr/pub/docs/oreilly-book/html/book-ora107.html)
##### parser.mly 
*Header* : Importe les types nécessaire pour la syntaxe : 
```ocaml=
%{
open Basic_types ;;

let phrase_of_cmd c =
 match c with
   "RUN" -> Run
 | "LIST" -> List
 | "END" -> End
 | _ -> failwith "line : unexpected command"
;;

let bin_op_of_rel r =
 match r with
   "=" -> EQUAL
 | "<" -> INF
 | "<=" -> INFEQ
 | ">" -> SUP
 | ">=" -> SUPEQ
 | "<>" -> DIFF
 | _ -> failwith "line : unexpected relation symbol"
;;

%}
```

*Declaration* contient 3 sections : 
* lexeme declaration 
* precedente declaration 
* declaration of the start symbol 

```ocaml=
(*lexical units*)
 %token <int> Lint
 %token <string> Lident
 %token <string> Lstring
 %token <string> Lcmd
 %token Lplus Lminus Lmult Ldiv Lmod
 %token <string> Lrel
 %token Land Lor Lneg
 %token Lpar Rpar
 %token <string> Lrem 
 %token Lrem Llet Lprint Linput Lif Lthen Lgoto
 %token Lequal
 %token Leol
 
 (*operator rules*)
 %right Lneg
 %left Land Lor
 %left Lequal Lrel
 %left Lmod
 %left Lplus Lminus
 %left Lmult Ldiv
 %nonassoc Lop
 
 (*parsing of command line*)
 %start line
 %type <Basic_types.phrase> line
```
Dans le parser Bossa nous avons : 
```ocaml=
%start bossa_spec
%type <Ast.scheduler * Objects.sched> bossa_spec
```

*Grammar rules* contient 3 sections : 
```ocaml=
 %%
(*Rule for a line*)
line : 
   Lint inst Leol               { Line {num=$1; inst=$2} }
 | Lcmd Leol                    { phrase_of_cmd $1 }
 ;

(*Rule for an instruction*)
inst :
   Lrem                         { Rem $1 }
 | Lgoto Lint                   { Goto $2 }
 | Lprint exp                   { Print $2 }
 | Linput Lident                { Input $2 }
 | Lif exp Lthen Lint           { If ($2, $4) }
 | Llet Lident Lequal exp        { Let ($2, $4) }
 ;

(*Rule for an expression*)
exp :
   Lint                         { ExpInt $1 }
 | Lident                       { ExpVar $1 }
 | Lstring                      { ExpStr $1 }
 | Lneg exp                     { ExpUnr (NOT, $2) }
 | exp Lplus exp                { ExpBin ($1, PLUS, $3) }
 | exp Lminus exp               { ExpBin ($1, MINUS, $3) }
 | exp Lmult exp                { ExpBin ($1, MULT, $3) }
 | exp Ldiv exp                 { ExpBin ($1, DIV, $3) }
 | exp Lmod exp                 { ExpBin ($1, MOD, $3) }
 | exp Lequal exp                { ExpBin ($1, EQUAL, $3) }
 | exp Lrel exp                 { ExpBin ($1, (bin_op_of_rel $2), $3) }
 | exp Land exp                 { ExpBin ($1, AND, $3) }
 | exp Lor exp                  { ExpBin ($1, OR, $3) }
 (*prec : use of unary with Lop, unary minus*)
 | Lminus exp %prec Lop        { ExpUnr(OPPOSITE, $2) }
 | Lpar exp Rpar                { $2 }
 ;
 %%
```

##### lexer.mll
Reconnait les lexical unit définit : 
```ocaml=
rule lexer = parse
   [' ' '\t']            { lexer lexbuf }

 | '\n'                  { Leol }

 | '!'                   { Lneg }
 | '&'                   { Land }
 | '|'                   { Lor }
 | '='                   { Lequal }
 | '%'                   { Lmod }
 | '+'                   { Lplus }
 | '-'                   { Lminus }
 | '*'                   { Lmult }
 | '/'                   { Ldiv }

 | ['<' '>']             { Lrel (Lexing.lexeme lexbuf) }
 | "<="                  { Lrel (Lexing.lexeme lexbuf) }
 | ">="                  { Lrel (Lexing.lexeme lexbuf) }

 | "REM" [^ '\n']*       { Lrem (Lexing.lexeme lexbuf) }
 | "LET"                 { Llet }
 | "PRINT"               { Lprint }
 | "INPUT"               { Linput }
 | "IF"                  { Lif }
 | "THEN"                { Lthen }
 | "GOTO"                { Lgoto }

 | "RUN"                { Lcmd (Lexing.lexeme lexbuf) }
 | "LIST"               { Lcmd (Lexing.lexeme lexbuf) }
 | "END"                { Lcmd (Lexing.lexeme lexbuf) }
 
 | ['0'-'9']+           { Lint (int_of_string (Lexing.lexeme lexbuf)) }
 | ['A'-'z']+           { Lident (Lexing.lexeme lexbuf) }
 | '"' [^ '"']* '"'     { Lstring (string_chars (Lexing.lexeme lexbuf)) }
```

#### Building
Permet de faire des vérifications sur le contenu parsé : 
* Combine module admission criteria into a single admission criteria. 
* Renaming as may be useful in the module combining process
* type check 
* variable check

## Ipanema-Kernel
Pour que les fonctions générées par le compilateur d'ipanema soient intégrées dans le noyau, il est définit dans le répertoire include :  [inclule/linux/ipanema.h](https://gitlab.inria.fr/ipanema/ipanema-kernel/-/blob/linux-4.19-ipanema/include/linux/ipanema.h), les structures généré en C par le compilateur Ipanema ainsi que l'architecture des fonctions qui seront générées. On notera que ce sont des **pointeur de fonction** permettant à l'ordonnanceur de choisir la politique d'ordonnancement implémenté par Ipanema selon la strucutre *struct ipanema_policy*. 
Remarque : le fichier [sched/ipanema.h](https://gitlab.inria.fr/ipanema/ipanema-kernel/-/blob/linux-4.19-ipanema/kernel/sched/ipanema.h) fait un include au fichier include/linux/ipanema.h.

Pour la structure, il est important de savoir que *struct ipanema_policy* contient un champs *routines* qui est de type *struct ipanema_module_routines* où on retrouve tous les pointeurs de fonctions.

Concernant l'implémentation des fonctions du ipenema.h se trouvent dans [sched/ipanema.c](https://gitlab.inria.fr/ipanema/ipanema-kernel/-/blob/linux-4.19-ipanema/kernel/sched/ipanema.c). (**Important de comprendre les fonctions définient dans ce fichier**). 

### Installation 
Pour l'installation à partir d'une image debian10-x64-nfs : 
```shell=
cp /boot/kernel_config ~/ipanema-kernel/.config
make olddefconfig
make menuconfig #choisir les bonnes options
make -j 
make modules_install
make install
less /boot/grub/grub.cfg #check si on reboot sur le bon kernel
grub-reboot '1>2' #choisir le bon kernel
reboot
```

## Grid5000
Connexion :
```shell=
ssh grenoble.g5k
```

Réserver un node : 
```shell=
oarsub -I
```

Exporter la valeur du Job_ID : 
```shell=
export OAR_JOB_ID
```

Déployer l'image : 
```shell=
kadeploy3 -k -f ~/nodes -e $IMAGE
```

Connexion en root : 
```shell=
ssh root@dahu-xx
```

**Sauvegarde l'image** très important : 
```shell=
tgz-g5k
```
ou
```shell=
tgz-g5k -f gvacherias@fgrenoble:/home/gvacherias/ipanema/ipanema-image.tgz
```
ou 
```shell=
frontend : tgz-g5k -m node -f ~/path_to_myimage.tgz
```

Déploie un environnement créé : 
```shell=
kadeploy3 -f $OAR_NODEFILE -a mydebian10-x64-nfs.env
```

Cf script *deploy*

## Benchmark 
**forever** : 
```shell=
#!/bin/bash
set -x
while :; do $@; done
```
Affiche les commandes exécuté et leurs arguments
**steve** : 
```shell=
#!/bin/bash
set -x -u
find "$@" -type f -executable -name '*.sh' | sort -R |
    while read job
    do
	${job}
    done
```
Execute tous les .sh dans le répertoire benchmark
