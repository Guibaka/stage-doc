# Kameleon

## Installation 

On installe les outils nécessaire : 
```shell=
$ export PATH="/home/gvacherias/.gem/ruby/2.5.0/bin:$PATH"
$ gem install --user kameleon-builder
```

Les packages nécessaire : 
```shell=
$ apt-get install libguestfs-tools socat tightvncserver qemu-system libvirt-clients libvirt-daemon-system cpu-checker
```

Pour les fichiers tarball il faut les placer dans le répertoire public de notre home. Avec un path : 
```shell=
$ http://public.site.grid5000.fr/~username/file
```

Ajoute les recettes d'environement de grid5000 : 
```shell=
$ kameleon repo add grid5000 https://gitlab.inria.fr/grid5000/environments-recipes
```

Crée notre propre recette : 
```shell=
$ kameleon new kameleon-recipe-name grid5000/from_grid5000_environment/base
```

## Configuration 
Dans le fichier kameleon-recipe-name.yaml modifier : 
```yaml=
## Environment to build from
grid5000_environment_import_name: "ipanema"
grid5000_environment_import_user: "username"
```


## Problème 
* Suspecté que l'erreur provient du manque des *kvm acceleration* (ce qui n'était pas le cas)
* Peut-être que d'étendre une image ipanema (nfs) sur une image debian (min) poserai un soucis ?
* Lors de l'étape du bootstrap, *start_qemu.yaml* la commande : 
```shell=
$ ssh -q -F /home/gvacherias/kameleon/debianipanema-test/build/debianipanema-test/ssh_config debianipanema-test -t /bin/bash
```
Option : 
* *-q* : quiet 
* *-F* : configFile 
* *-t* : Force pseudo-terminal allocation.  This can be used to execute arbitrary screen-based programs on a remote machine, which can be very useful, e.g. when implementing menu services

Voici le fichier de configuration généré lors de l'étape précédente : 
```typescript=Host debianipanema-test
HostName 127.0.0.1
Port 50000
User root
UserKnownHostsFile /dev/null
StrictHostKeyChecking no
PasswordAuthentication no
IdentitiesOnly yes
LogLevel FATAL
ForwardAgent yes
Compression yes
Protocol 2
IdentityFile /home/gvacherias/kameleon/debianipanema-test/build/debianipanema-test/insecure_ssh_key
```
* Cette étape ne pose pas de problème avec les environnement existant tel que debian10-x64-nfs avec *environment_user* : "deploy".
