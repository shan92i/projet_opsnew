Numéros étudiants : 21110777, 21100918


## Ce projet a pour objectif de créer et de déployer une application comprenant : 
- un serveur Strapi : backend de notre application
- PostgreSQL : base de données de Strapi, pour y stocker les informations
- un projet React : frontend de notre application

# Structure du projet :

projet_opsnew \
│── ... (fichiers du projet Strapi) \
│── Dockerfile \
│── docker-compose.yml \
│── README.md (vous êtes ici) \
├── dump \
│   └── backup.sql \
├── opsci-strapi-frontend \
│   ├── ... (fichiers de l'app React) \
│   └── Dockerfile 


## Méthode 1 : avec Docker en utilisant des conteneurs individuels

### Pour lancer l'application

1. Créer un docker network (le réseau qui contient nos conteneurs, que l'on fera communiquer)

        docker network create my_network
    
2. Lancer Postgresql (les options servent à assurer la communication entre Strapi et sa base de données )

        docker run -dit -p 5432:5432 -e
        POSTGRES_PASSWORD=safepassword -e
        POSTGRES_USER=strapi --net
        my_network --name strapi-pg postgres
        
- Après avoir créé notre projet strapi via la commande : yarn create strapi-app
- Via le mode "custom", on définit l'environnement

### Création d'un Dockerfile pour le projet Strapi en se basant sur la doc. Strapi

3. On construit une image à partir du Dockerfile se situant dans le dossier courant
   
        docker build -t projet_opsnew-strapi .

    
3.(bis) Lancement de Strapi

    docker run -dp 1337:1337 --net my_network --name strapi projet_opsnew-strapi
    
4. Une fois Strapi démarré avec succès, accédez à l'interface Strapi à l'adresse suivante :

    http://localhost:1337
    
5. On crée un compte Strapi, ensuite on crée la collection `product` 

6. Configuration du frontend React (que l'on détaille davantage dans la méthode 2)
 
## Méthode 2 : avec Docker Compose

Avant de créer le fichier docker-compose.yml, nous avons : 
1. Créé le Dockerfile correspondant à notre projet Strapi

2. Cloné le projet React : git clone https://github.com/arthurescriou/opsci-strapi-frontend

3. Compilé le projet via "yarn build", puis minifié en artefacts -> cela crée alors un dossier "Dist", parfait, tout fonctionne ! 

4. On vérifie dans le fichier conf.ts que l'URL est bien défini à http://localhost:1337 ; on va ensuite chercher le token, dans strapi, pour l'ajouter à la variable TOKEN -> ainsi, l'application React, au moment d'accéder à l'API de Strapi, pourra utiliser le jeton créé

5. Ensuite, on crée le Dockerfile correspondant au projet React
6. Puis on crée finalement notre fichier docker-compose.yml 

7. A l'aide de la commande "docker-compose up", on démarre tous nos services

8. On accède finalement à l'application React à l'adresse http://localhost:5173, comme défini via le champ EXPOSE dans le Dockerfile

9. Rien ne s'affiche ? C'est normal, dans l'application Strapi, il faudra utiliser la rubrique Content Manager pour créer les instances de notre collection `Product`, précédemment créée 

10. On actualise, et le frontend est parfait ! 
Bonus. Dans Docker Compose, nous avons défini un volume  "dump" pour lier le conteneur PostgreSQL à un répertoire local sur l'hôte. Cela nous permet de stocker les sauvegardes de la base de données, et de conserver ces données même en dehors de notre environnement Docker


# Erreurs rencontrées -> comment les résoudre ? 

Une erreur de démarrage de l'application avec Docker Compose ?

"Erreur : Les ports ne sont pas disponibles lors du démarrage du conteneur Strapi" 
    
    Solution : Identifier et libérer le port 5432 utilisé par PostgreSQL, via la commande : sudo systemctl stop postgresql

-> Il s'avère qu'un autre processus utilisait déjà ce port, donc nous avons dû le libérer avant de relancer les conteneurs Docker.
    
---
Au moment de compiler le projet React à l'aide de "yarn build", nous avons fait face à une erreur étrange : 

yarn run v1.22.19
$ tsc && vite build
/bin/sh: ligne 1: tsc : commande introuvable
error Command failed with exit code 127.
info Visit https://yarnpkg.com/en/docs/cli/run for documentation about this command.

Il semblait que la commande `tsc` n'était pas installée. TypeScript, c'est ce qui permettait de compiler le projet
LA solution, ça a été d'exécuter la commande pour l'installer :
yarn add typescript --dev

problème, en exécutant la commande, on avait une erreur de compatibilité ! : 

yarn add typescript --dev
yarn add v1.22.19
[1/4] Resolving packages...
[2/4] Fetching packages...
error vite@5.0.12: The engine "node" is incompatible with this module. Expected version "^18.0.0 || >=20.0.0". Got "16.18.1"
error Found incompatible module.
info Visit https://yarnpkg.com/en/docs/cli/add for documentation about this command.

Même en installant une version plus récente de Node.js, avec "nvm install 20" et en essayant de l'utiliser "nvm use 20", l'erreur persistait...

On change alors de solution, et au lieu d'upgrader Node.js, on a installé une ancienne version de Vite, qui serait compatible avec la version de Node.js qui était dans le système : 

    yarn add vite@3.0.0 --dev

Tadam ! Le tour est joué, notre projet React est compilé ! 
    




