---
share: true
revision: false
MOC:
  - "[[MOC_Perso_Informatique|MOC_Perso_Informatique]]"
tags:
  - Container
  - Docker
  - Kubernetes
---
---
share: true
revision: false
MOC: 
- "[[MOC_Perso_Informatique|MOC_Perso_Informatique]]"
tags:
- Container
- Docker
- Kubernetes
---
# Docker

## Question et à faire

- Voir la fin du schéma que je ne comprends pas

---

Objectif : Monter directement, facilement et rapidement des applications et leurs versions. Chaque service est lancé dans un container autonome et indépendant des autres (en faisant tout de même les liens nécessaires).

## Fonctionnement général

Les conteneurs peuvent être basés sur différentes plateformes ou systèmes d'exploitation (OS). Dans la majorité des cas, les conteneurs les plus utilisés sont de type Linux. Cependant, il existe également des conteneurs Windows qui sont basés sur la virtualisation de l'OS à travers Hyper-V.

Sous Windows, l'installation et l'utilisation de Docker Desktop impliquent l'installation d'une image Linux, ou plus simplement de l'application WSL (Windows Subsystem for Linux), afin de permettre l'exécution des conteneurs de type Linux.

![[Pasted image 20240418180513.png|Pasted image 20240418180513.png]]
![[docker.png|docker.png]]

## Les images et les conteneurs

Qu'est-ce qu'une image par rapport à un conteneur ?

![[Pasted image 20240321183907.png|Pasted image 20240321183907.png]]

### L'image
Une image est un fichier binaire qui contient tous les éléments nécessaires pour exécuter une application, y compris le code source, les bibliothèques, les dépendances, les variables d'environnement et les fichiers de configuration.

Elle est généralement créée à partir d'un fichier Dockerfile (sans extension) ou d'un autre format de description d'image et peut être construite à l'aide de Docker.

Une image est statique et ne peut pas être modifiée une fois créée. Toutefois, elle peut servir de base pour créer de nouveaux conteneurs.

Une image est juste un état d'une configuration Docker à un moment. On le voit très bien dans le Dockerfile de PHP, ils sont partis d'une installation vierge de Debian pour installer et configurer PHP. De notre côté, on aura juste à lancer cette image dans un container et ce sera tout bon. Docker va ajouter des couches de configuration sur l'image à chaque changement de configuration.

Bien entendu, nous pouvons créer notre propre image pour la partager ou s'en servir personnellement,

Beaucoup de services plus ou moins connus proposent des images pré-configurées pour que vous n'aillez plus qu'à les lancer en un clin d'oeil. La liste est disponible sur le [Docker Hub](https://hub.docker.com/).
Il s'agit d'un référentiel centralisé qui permet aux utilisateurs de Docker de stocker, partager et gérer des images de conteneurs Docker. Ceci peut être fait de manière publique ou avec un compte payant pour un stockage privé.

### Le conteneur

Un conteneur est une instance exécutable d'une image. Il s'agit d'un processus isolé qui s'exécute sur un système hôte.  
Les conteneurs utilisent les fonctionnalités de virtualisation au niveau du système d'exploitation pour isoler les processus, les systèmes de fichiers et les ressources.

C'est l'exécution d'une image, nous pouvons exécuter en même temps plusieurs fois la même image.

## Layers
Une couche Docker est un ensemble de fichiers ou de changements appliqués à un système de fichiers. Chaque instruction dans un Dockerfile (telles que `RUN`, `COPY`, `ADD`, etc.) crée une nouvelle couche dans l'image Docker résultante.
Ces couches sont empilées les unes sur les autres pour former l'image finale. Les couches sont immuables et en lecture seule, ce qui signifie qu'une fois qu'elles sont créées, elles ne peuvent pas être modifiées.
Les couches permettent une **reproductibilité et une efficacité élevée** dans Docker. Chaque couche est basée sur la couche précédente.
**Les couches partagées entre différentes images ne sont stockées qu'une seule fois, ce qui économise de l'espace disque.**
En réutilisant les couches existantes, la construction d'images est **plus rapide**.  Inconvénient : 
**Toute modification dans une couche entraîne la récréation de toutes les couches supérieures**, ce qui peut ralentir le processus de construction.

Les couches (layers) sont associées aux images Docker, pas aux conteneurs. La suppression de l'image supprime les layers concernées en laissant les layers utilisés dans d'autres images.

![[Pasted image 20240418172924.png|Pasted image 20240418172924.png]]

## Orchestration de containers
### Docker-compose
[Docker compose](https://github.com/docker/compose) est un outil simple d'orchestration de containers pour lancer un ou plusieurs containers, définis par un fichier nommé `docker-compose.yml` à la racine de votre projet, avec une seule ligne de commande.
Avec ce fichier, on peut monter tous les containers dont on a besoin pour notre application avec une simple ligne de commande.

Exemple :
```yml
# docker-compose.yml

version: '2'

services:
  blog-server:
    build: .
    image: laravel-blog
    links:
      - mysql
    volumes:
      - ./:/application

  queue-server:
    build: .
    image: laravel-blog
    command: php artisan queue:work
    links:
      - mysql
    volumes:
      - ./:/application

  mysql:
    image: mysql
    ports:
      - '3306:3306'
    environment:
      - MYSQL_ROOT_PASSWORD=secret
      - MYSQL_DATABASE=laravel-blog
    volumes:
      - ./tmp/db:/var/lib/mysql

  nginx:
    image: nginx
    ports:
      - '80:80'
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
      - ./:/application
    links:
      - blog-server
```
Deuxième exemple :
```yml
version: '3.8'  
  
services:  
  my-running-app:  
    image: httpd:2.4  
    container_name: my-running-app  
    ports:  
      - "8080:80"  
    volumes:  
      - ./site/:/usr/local/apache2/htdocs/
```
La ligne de commande permettant de monter tous les containers :
```bash
docker-compose up -d
```

Si vous souhaitez arrêter les containers et les supprimer :

```bash
docker-compose down
```

A noter que toutes les machines d'un même docker-compose font partie du même réseau.

Faire le docker compose est équivalent à faire plusieurs docker run avec tous les paramètres comme le port, etc.

#### Docker build
Dans un fichier `docker-compose.yml`, la directive `build` est utilisée pour indiquer à Docker Compose de construire une image Docker à partir d'un Dockerfile local ou à partir d'un répertoire local contenant le contexte de construction. Cette directive est utilisée pour construire des images personnalisées ou des images spécifiques à votre environnement, par opposition à l'utilisation d'images pré-construites provenant de Docker Hub ou d'un autre registre.

Exemple :
```yaml
version: '3'
services:   
	web:     build: ./chemin/vers/votre/Dockerfile
	ports:   - "8080:80"
```

### Outils avancés (Kubernetes, Rancher, Swarm)
![[kubernet.png|kubernet.png]]

Permet de mettre des règles sur des containers permettant par exemple de démarrer plusieurs containers selon la charge, etc. 

Il existe plusieurs orchestrateurs comme kubernetes, rancher ou celui de docker (Swarm).

Les masters et les nodes sont des serveurs physiques différents. Par exemple kubernetes a besoin de 2 masters, etc. Kubernetes va choisir s'il doit déployer sur un nouveau node ou sur le même, il choisi tout seul.

L'orchestrateur fait aussi d'autres choses comme le déploiement des mises à jour.

#### Différence avec docker compose
**Docker Compose** est un outil **simple et léger** conçu pour gérer des applications multi-conteneurs sur un seul hôte. Il est facile à configurer et à utiliser, ce qui le rend idéal pour le développement local et les tests. Docker Compose utilise un fichier de configuration YAML pour définir les conteneurs à exécuter, les réseaux et les volumes à créer, ainsi que les dépendances entre les conteneurs.

**Kubernetes**, en revanche, est un orchestrateur de conteneurs de niveau production conçu pour gérer des **applications distribuées à grande échelle** sur plusieurs hôtes. Il offre un ensemble plus riche de fonctionnalités que Docker Compose, notamment :

- **Découverte de service et équilibrage de charge** : Kubernetes permet aux conteneurs de se découvrir et de se connecter les uns aux autres, même si leurs adresses IP changent. Il peut également équilibrer la charge du trafic entre les instances d'une application.
- **Auto-guérison** : Kubernetes peut automatiquement redémarrer les conteneurs défaillants ou mettre à l'échelle les applications en fonction de la demande.
- **Gestion des ressources** : Kubernetes peut gérer les ressources CPU, mémoire et stockage pour les conteneurs.
- **Déploiements progressifs** : Kubernetes peut déployer de nouvelles versions d'applications de manière progressive et sûre.

## Volumes
Comme on l'a vu avant, les containers sont isolés du système hôte. Ils ont un accès en lecture seule pour exister mais c'est tout. La problématique, c'est que dans beaucoup de cas, on va avoir besoin d'utiliser des fichiers qui ne sont pas dans un container.

Ils devront soit, être partagés par plusieurs containers, soit ils devront être sauvegardés mêmes après la suppression d'un container. En effet, lorsqu'un container est supprimé, l'ensemble des fichiers présents à l'intérieur le sont également. Cela peut être gênant pour ces raisons ou pour un upload de fichier par un utilisateur sur une application Web entre autres.

Pour remédier à ce problème, Docker a mis en place un système appelé [Volume](https://docs.docker.com/glossary/?term=volume).

Les volumes ont pour objectifs d'utiliser des données directement sur le système hôte. Pour faire simple, on va dire à Docker : "Voilà ce dossier (ou ce fichier) en particulier ne le cherche pas dans le container mais à cet emplacement sur l'host. Merci !". Les chemins vers ces dossiers et fichiers peuvent être mis en place dans la configuration du container que nous verrons en détail plus tard.

A partir du moment où le volume est mis en place, le container a un accès direct au dossier ciblé dans l'host. Ce n'est pas une copie du container vers l'host mais bien un accès direct du container vers le File System de l'host. Donc on va s'en servir pour partager du code source entre plusieurs containers, mais aussi sauvegarder une base de données MySQL et plus encore. On peut, bien entendu, définir autant de volumes que l'on souhaite avec les chemins de notre choix, il n'y a pas de limitation à ce niveau-là.

![[Pasted image 20240418193839.png|Pasted image 20240418193839.png]]

Pour mapper un dossier, il faut utiliser la propriété -v :
```bash
docker run -v ${PWD}:/usr/local/apache2/htdocs/ httpd:2.4
```
_${PWD}_ permet de récupérer le chemin absolu du répertoire de travail

## Ports
De la même façon que le reste, les ports réseaux à l'intérieur d'un container sont isolés du reste du système. C'est une notion importante car, on va devoir faire de la redirection de port. 

Lorsqu'un service expose un port à l'intérieur du container, il n'est, par conséquent, pas accessible depuis l'**extérieur** par défaut. Il faut donc définir le port lié de notre propre machine :

![[Pasted image 20240418184214.png|Pasted image 20240418184214.png]]

### Entre containers

Il existe également un autre aspect important avec les ports, c'est la notion d'[EXPOSE](https://docs.docker.com/engine/reference/builder/#expose).

`EXPOSE` va permettre d'**ouvrir un port uniquement pour les autres containers**. En pratique, on va s'en servir pour faire interagir des containers entre eux. Typiquement, un container MySQL n'aura pas besoin d'être accessible via l'host, on va donc exposer son port par défaut `3306` aux autres container pour qu'ils y aient accès.

Docker utilise aussi cette notion lors de l'utilisation de l'opérateur `--link` que nous allons détailler.

## Réseaux
### Bridge
Sur Docker il existe un network par défaut. Son nom est **bridge** et si vous ne changez pas la configuration, tous les containers seront associés à ce réseau. Qu'est-ce que cela change pour nous ? Si vous n'avez pas besoin d'isoler vos containers entre eux, ils pourront tous communiquer ensemble sur ce réseau sans avoir à utiliser **les links** par exemple.

Ce sont tous les containers d'un projet (un projet est un groupe de container lancés via le même compose up ou dans le même dossier) qui peuvent communiquer entre eux.

De plus certains containers communiquent entre eux sans préciser les ports car par défaut, dans l'image, certains ports sont exposés entre les machines (par exemple le 3306 de mariadb).

### Utiliser le même réseau dans des projets différents

J'ai repris deux docker-compose mais séparé mariadb et phpmyadmin dans des docker-compose différents (un docker compose est dans un un sous-dossier src).

En lançant mes deux docker compose up, j'ai maintenant mes containers lancés dans deux projets différents :

![[Pasted image 20240503100042.png|Pasted image 20240503100042.png]]

Bien qu'ils utilisent le driver bridge, les deux projets sont dans des réseaux différents.

**C'est donc tous les containers dans un même projet (issus du même docker compose ou dans le même dossier) qui sont dans un même réseau**.

Dans mon cas, nous avons donc deux réseaux :

  ![[Pasted image 20240503103610.png|Pasted image 20240503103610.png]]

  ![[Pasted image 20240503103640.png|Pasted image 20240503103640.png]]
 
Je n'arrive donc pas à me connecter à mon mariadb :

![[Pasted image 20240503103708.png|Pasted image 20240503103708.png]]

Pour qu'ils puissent communiquer, j'ai ajouter le réseau dans les docker-compose :

![[Pasted image 20240503103723.png|Pasted image 20240503103723.png]]
![[Pasted image 20240503103736.png|Pasted image 20240503103736.png]]

En faisant ainsi, j'ai bien un seul et même réseau :

 ![[Pasted image 20240503103759.png|Pasted image 20240503103759.png]] 

Et la connexion fonctionne bien :
![[Pasted image 20240503103838.png|Pasted image 20240503103838.png]]

## Les links

Une des philosophies de Docker est d'avoir un container par service. Les containers ont besoin de communiquer entre eux. On a pris plus tôt le cas de MySQL mais les exemples ne manquent pas.

L'opérateur `link` permet, comme son nom l'indique, de lier des containers entre eux automatiquement. Il permet au container d'avoir accès directement au service d'un autre container via un port sur un **réseau privé**. Pour définir ce port, il regarde la valeur du `EXPOSE` sur le container cible.

Docker va également modifier le fichier `/etc/hosts` pour nous. Ainsi, on pourra accéder à ce container via son nom et non son ip plus facilement. L'avantage c'est qu'on ne va pas exposer ici un container sur l'host pour rien. C'est un réseau totalement privé entre deux containers.

![[Pasted image 20240418184531.png|Pasted image 20240418184531.png]]

## Gestion d'environnement Docker - Portainer
Portainer est une interface utilisateur graphique open source pour gérer des environnements Docker. Il permet de visualiser, de gérer et de déployer des conteneurs Docker, des réseaux et des volumes, ainsi que de surveiller les ressources système et les performances des conteneurs.

En résumé, il est possible d'installer docker sur plusieurs serveurs et gérer ces serveurs ainsi que les dockers installés directement depuis portainer.

## Exemple architecture logiciel
![[Pasted image 20240418201637.png|Pasted image 20240418201637.png]]

## Commandes docker simple

### Ouvrir une instance

Ouvrir une instance d'une image (un conteneur) :
```bash
docker run -p 80:80 --detach httpd
```
**docker run** ouvre une instance d'une image, si l'image n'existe pas, il la télécharge directement sur https://hub.docker.com/
**--port ou -p** indique le **port** de l'instance : Dans l'option `-p 81:80`, cela signifie que le port 81 de votre système hôte est mapé sur le port 80 du conteneur Docker.
**--detach ou -d** lance le processus en **arrière-plan**

### Version de docker

```bash
docker version
```

- Cette commande nous permet d’afficher la version du client et du serveur (aussi appelé Engine) de Docker.
- Elle permet notamment de vérifier le bon fonctionnement de Docker

### Information générale

```bash
docker info
```

- Cette commande nous permet d’afficher plus de détails concernant le serveur Docker (aussi appelé le docker Engine).
- Le nombre de conteneurs actuellement en fonctionnement
- Le nombre d’images stockées dans Docker
- Autres...

### Récupérer une image déjà existante
Retrouver l'image souhaité sur dockerhub si besoin.
```bash
docker pull nomImage
Exemple : docker pull alpine
```

### Liste des conteneurs (en cours)

```bash
docker container ls
## Ou
docker ps
```

- Liste les conteneurs qui sont actuellement en cours d'exécution

### Liste des conteneurs (tous)

```bash
docker container ls -a
## Ou
docker ps -a
```

### Liste des images (tous)

```bash
docker image ls
```

- Liste tous les conteneurs, y compris ceux qui ont été arrêtés

### Liste des réseaux (tous)

```bash
docker network ls
```

### Démarrer un conteneur

```bash
docker container run -p 80:80 -d --name myHTTPD httpd
```

- L'option name permet de spécifier un nom et de l'associer à ce conteneur, ce qui est beaucoup plus simple par la suite pour pouvoir interagir avec lui et lui envoyer des commandes (plutôt que d’utiliser son id)

### Arrêter un conteneur

```bash
docker container stop myHTTPD
```

- Permet d'arrêter un container (il ne sera donc visible que via la commande docker container ls –a)

### Supprimer un conteneur

```bash
docker container rm myHTTPD
```

- Supprimer le conteneur. Les données modifier seront perdu.

### Naviguer dans le conteneur

```bash
docker exec -ti myHTTPD bash

## OU

docker exec -ti <CONTAINER_ID> bash
```

### Effacer le cache, les images, containers et volumes qui ne sont pas utilisés

```bash
## Efface uniquement le cache et les images non utlisées
docker prune

## Efface le cache, les images, les containers et les volumes non utlisées
docker prune -a
```

## DevOps - Infomaniak Jelastic
![[Pasted image 20240524094718.png|Pasted image 20240524094718.png]]

Voir [[schéma devops docker|schéma devops docker]]

## Sources

### Cours EMT par Christophe Chevalier
https://gitlab.indc.dev/formations/docker-pratique/-/tree/main/01%20-%20Les%20conteneurs?ref_type=heads

gitlab.indc.dev
**User : EMT_FORMATION**
**Pass : q@M0gkl3J!7vUbpER**

### Tuto en ligne :
- https://guillaumebriday.fr/comprendre-et-mettre-en-place-docker
- https://guillaumebriday.fr/docker-les-reseaux-personnalises
- https://devopssec.fr/article/fonctionnement-manipulation-reseau-docker