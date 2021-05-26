# Kameleon

## Installation 

On installe les outils nécessaire : 
```shell=
export PATH="/home/gvacherias/.gem/ruby/2.5.0/bin:$PATH"
gem install --user kameleon-builder
```

Les packages nécessaire : 
```shell=
apt-get install libguestfs-tools
apt-get install socat
apt-get install tightvncserver
```

Pour les fichiers tarball il faut les placer dans le répertoire public de notre home. Avec un path : 
```shell=
http://public.site.grid5000.fr/~username/file
```

Ajoute les recettes d'environement de grid5000 : 
```shell=
$ kameleon repo add grid5000 https://gitlab.inria.fr/grid5000/environments-recipes
```

