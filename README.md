# Résumé du Challenge

Pour ce challenge, d'après ma compréhension du problème, il ne fallait pas toucher 
au code source de l'application (ce qui en soit aurait simplifier le processus de containerisation 
au niveau du composant frontend qui pose probleme avec le js qui va requeter l'url en dur : http://localhost:4200 ).

Afin de préciser mes choix techniques, je suis parti sur quelques hypothèses:
1. Il ne faut pas modifier le code source => je pars sur une solution de "repainting"
2. Le serveur de base de donnée doit tourner avant de démarrer l'api
3. Le serveur API doit tourner avant de démarrer le frontend

> Pour mettre en place rapidement l'environnement : 
>> mkdir challenge\
cd challenge\
git clone https://github.com/govpf/devops-challenge.git \
git clone https://github.com/atouboulic/challenge-results.git \
cd challenge-results \
sh copyfiles_to_git_challenge_folder.sh \
cd ../devops-challenge 


_____________________________________

# Pour faire fonctionner l'application avec docker, docker-compose ou minikube:

## Pre-requis

1. Compiler l'api
```
$ cd api
$ mvn package -DskipTests
```

* Le -DskipTests évite de devoir démarrer un serveur postgreSQL qui est nécessaire au Tests Unitaires. 
* Si besoin des tests unitaires: faire docker-compose up -d db au préalable

2. Compiler le frontend
```
$ cd frontend
$ yarn install --frozen-lockfile 
$ yarn build
$ cd ..
```

3. S'assurer que les port 5432, 8080, 80 et 4200 ne sont pas utilisés en local

## Démarrer les containers avec docker-compose

Appliquer les pré-requis puis :

```
docker-compose up 
```

* Note : les containers vont démarrer de manière séquentielle. (PostgreSQL > API > FRONTEND)

## Pour faire fonctionner l'application avec manuellement avec Docker :

> Appliquer les pré-requis 

### 1. Démarrer la DB (postgres) 

```shell 
docker-compose up -d db 
```

### 2. Build api image
```
cd api
docker build -t challenge-api .
```

### 3. Démarrer l'api dans un docker container
```
docker run -p 8080:8080 --network=host challenge-api  << For local usage
ou
docker run --net=container:db challenge-api << in order to be in same network as db
ou 
docker run -e "SPRING_DATASOURCE_URL=jdbc:postgresql://<PUBLIC_IP_ADDRESS>:5432/mystuff" challenge-api  << if public url    
```

### 4. Build frontend image
```
cd ../frontend
docker build -t challenge-frontend .
```

### 5. Démarrer le frontend dans un docker container
```
docker run -it  --name frontend -p 80:80 --network=host challenge-frontend
```

## Pour faire fonctionner l'application avec manuellement avec Minikube :

pré-requis : 
* installer minikube, kubectl
* avoir compilé l'api et le frontend auparavant

1. Démarrer minikube 

```
minikube start --driver=docker
   (ou minikube start --driver=local)
   
eval $(minikube docker-env)
```

> La commande eval permet d'utiliser le docker registry "de minikube"

2. Générer les images docker dans l'environnement minikube

```shell
docker-compose up --no-start --build
```


3. Déployer les différent fichier yml

```shell
kubectl apply -f kube
```

>Les services suivants seront créés :
>* postgres (5432:5432)
>* api (8080:8080)
>* frontend (80:80 / 4200:4200) (avec NodePort)
> 
>
>Le persistent volume sera créé pour stocker les données postgreSQL
>* postgres-pv (100M)
>
>Le déploiement sera créé pour Postgres :
>* postgres
>
>2 replicas set seront créés avec un scaling à 1 :
>* api
>* frontend

4. identifier l'ip de Minikube et se connecter au frontend
```
minikube ip
```
Se connecter sur son navigateur a l'url : <ip_minikube>:30000

> Note : le port est ouvert via un NodePort

> Remarque : l'appli web fait des requetes sur localhost:4200 => ca plante

