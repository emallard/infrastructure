
# Setup de l'environnement MariaDB / Dotnet Core

## Arborescence
- ~/docker/aspnetapp : sources du projet .Net
- ~/docker/datadir : répertoire contenant la base de donnée 

## Repartir d'un environnement vide :
docker stop
```
docker stop some-mariadb
docker stop some-aspnetapp
```

ou

```
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)
```

supprimer les repertoires 

```
rm -rf ~/docker/datadir
```

# Base de donnée

- https://mariadb.com/kb/en/library/installing-and-using-mariadb-via-docker/
- https://hub.docker.com/_/mariadb/

```
docker run --name some-mariadb -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mariadb:tag
```

Se logguer sur la base: -it pour avoir un shell interactif
```
docker exec -it some-mariadb mysql -u root -p
Enter password: my-secret-pw
mysql> show databases;
```

Spécifier le fichier de base de données à utiliser : -v pour lier un répertoire, -e pour setter une variable d'environnement
```
mkdir ~/docker/datadir
docker run --name some-mariadb -v ~/docker/datadir:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mariadb:tag
```

Dump database :
```
docker exec some-mariadb sh -c 'exec mysqldump --all-databases -uroot -p"$MYSQL_ROOT_PASSWORD"' > ~/docker/datadir/all-databases.sql
```

# Développer une appli dotnetcore Entity Framework

https://docs.microsoft.com/fr-fr/ef/core/
https://docs.microsoft.com/fr-fr/ef/core/providers/

Pour SQLServer
```
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
```
Pour MariaDB :
```
dotnet add package Pomelo.EntityFrameworkCore.MySql --version 2.0.0
```



# Docker / Dotnet Core

https://docs.microsoft.com/fr-fr/dotnet/core/docker/building-net-docker-images

## Créer l'image à partir d'une version déja compilée

Publish :

```
cd ~/docker/aspnetapp
dotnet publish -c Release -o out
```

Créer une image docker d'éxécution:
```
docker build -t aspnetapp .
```

qui utilise le fichier ./Dockerfile

```
# build runtime image
FROM microsoft/aspnetcore:2.0
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "aspnetapp.dll"]
```

## Créer l'image sur un serveur d'intégration continue

```
cd ~/docker/aspnetapp
docker build -t aspnetapp .
```

qui utilise le fichier ./Dockerfile

``` docker
FROM microsoft/aspnetcore-build:2.0 AS build-env
WORKDIR /app

# copy csproj and restore as distinct layers
COPY *.csproj ./
RUN dotnet restore

# copy everything else and build
COPY . ./
RUN dotnet publish -c Release -o out

# build runtime image
FROM microsoft/aspnetcore:2.0
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "aspnetapp.dll"]
```

## Lancer l'image

```
docker run -it --rm --name some-aspnetapp aspnetapp
```

## Lancer l'image avec docker-compose

https://docs.docker.com/compose/install/#install-compose


# Composer les 2 images docker

Lancer docker sur les différents ports et faire communiquer


- Créer un dump pour rejouer la prod


- Entites Metier .Net
- Web service des communes


## Multi domaines et certificats

https://www.sheevaboite.fr/articles/traefik-reverse-proxy-https-containers-docker



# Brouillon / Liens

https://medium.com/@Likhitd/asp-net-core-and-mysql-with-docker-part-1-b7ef538ecd8e
```
docker -f my.Dockerfile
```

test listening ports
```
netstat -tulpn
```

Envoyer une image sur le serveur de production
https://stackoverflow.com/questions/23935141/how-to-copy-docker-images-from-one-host-to-another-without-via-repository
```
docker save <image> | bzip2 | \
     ssh user@host 'bunzip2 | docker load'
It's also a good idea to put pv in the middle of the pipe to see how the transfer is going:

docker save <image> | bzip2 | pv | \
     ssh user@host 'bunzip2 | docker load'

```
# Post installation de docker :

https://docs.docker.com/install/linux/linux-postinstall/#specify-dns-servers-for-docker

- Ajouter le user au groupe docker.
- Et pour le probleme CONFIG_MEMCG_SWAP_ENABLED: missing,

```
Run sudoedit /etc/default/grub in a terminal and edit the GRUB_CMDLINE_LINUX line so it looks like this:

GRUB_CMDLINE_LINUX="cgroup_enable=memory swapaccount=1" 
Save and exit and then run sudo update-grub and reboot. That should be enough.
```


