

# Mini‑projet DevOps : Déploiement d’une application web conteneurisée

## 1. Objectif général

L’objectif de ce mini‑projet est de mettre en pratique les bases du déploiement et de l’orchestration de conteneurs avec Docker.  
Vous allez construire et exécuter une application web complète qui communique avec une base de données relationnelle.  

À l’issue du projet, vous devrez être capables de :  
- conteneuriser une application Node.js simple ( ou python); 
- connecter cette application à une base de données MySQL ; 
- orchestrer l’ensemble à l’aide de Docker Compose. 

***

## 2. Contexte et architecture cible

Vous travaillez dans une jeune startup qui souhaite disposer rapidement d’une petite application web fonctionnelle.  
On vous demande de mettre en place une architecture composée :  
- d’un backend Node.js (framework Express) exposant une API HTTP ;
- d’une base de données MySQL pour stocker des informations ; 
- le tout exécuté sous forme de conteneurs Docker, pilotés par Docker Compose. 

Architecture logique visée :

- Navigateur (client HTTP)  
  ↓  
- Application Node.js (API)  
  ↓  
- Base de données MySQL  

***

## 3. Structure de projet attendue

Le projet devra respecter la structure minimale suivante :

```text
mini-devops-project/
│
├── app/
│   ├── app.js
│   ├── package.json
│   └── Dockerfile
│
└── docker-compose.yml
```

- Le répertoire `app/` contient le code de l’application Node.js et le Dockerfile associé.  
- Le fichier `docker-compose.yml` décrit les différents services (application, base de données) et leurs relations. 

***

## 4. Étapes du projet

### Étape 1 – Création de l’application Node.js

Vous devez mettre en place une petite API HTTP qui :  
- expose au minimum une route `GET /` ;  
- à chaque requête, interroge la base de données MySQL ;  
- retourne par exemple la date et l’heure actuelles renvoyées par la base.  

L’objectif est de vérifier que l’application est effectivement capable de dialoguer avec MySQL via le réseau Docker. 

### Étape 2 – Construction de l’image Docker de l’application

Vous devez écrire un fichier `Dockerfile` dans le répertoire `app/` permettant de :  
- partir d’une image officielle Node.js ;  
- copier les fichiers nécessaires (`package.json`, code source) ;  
- installer les dépendances ;  
- exposer le port utilisé par l’application (3000 par exemple) ;  
- définir la commande de démarrage (par exemple `node app.js`).

Le but est d’obtenir une image Docker autonome de votre application.

### Étape 3 – Orchestration avec Docker Compose

Dans le fichier `docker-compose.yml`, vous devez définir au minimum deux services :  
- un service `web` correspondant à l’application Node.js, construit à partir du Dockerfile du répertoire `app/` ;  
- un service `db` correspondant à la base MySQL, basé sur une image officielle MySQL. 

Un exemple minimal de fichier `docker-compose.yml` est donné ci‑dessous, à adapter et à compléter :

```yaml
services:
  web:
    build: ./app
    container_name: devops_web
    ports:
      - "3000:3000"
    environment:
      # Exemple de variables d'environnement utilisées par l'app Node.js
      DB_HOST: db
      DB_USER: root
      DB_PASSWORD: root
      DB_NAME: testdb
    depends_on:
      - db

  db:
    image: mysql:5.7
    container_name: devops_db
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: testdb
      MYSQL_USER: root
      MYSQL_PASSWORD: root
    ports:
      - "3306:3306"
    # À améliorer au niveau 1 avec un volume pour la persistance
```

Points à observer dans ce fichier :  
- le service `web` est construit à partir du répertoire `app` et dépend du service `db` ; 
- l’application Node.js utilise le nom de service `db` comme hôte MySQL (`DB_HOST=db`) grâce au DNS interne de Docker ; 
- les paramètres de la base sont fournis via des variables d’environnement.  

### Étape 4 – Lancement et tests

Une fois les fichiers prêts, vous lancerez l’ensemble des services avec :

```bash
docker-compose up --build
```

Vous devez ensuite vérifier, via un navigateur ou `curl`, que l’application répond sur :

```text
http://localhost:3000/
```

et qu’elle est bien capable de communiquer avec la base de données (par exemple en affichant l’heure renvoyée par MySQL). 

***

## 5. Points pédagogiques à observer

Ce mini‑projet met en évidence plusieurs situations typiques rencontrées en DevOps :

1. **Problème de disponibilité de la base**  
   Au démarrage, la base MySQL peut ne pas être immédiatement prête alors que l’application tente déjà de s’y connecter. Il est alors nécessaire d’introduire un mécanisme d’attente ou de répétition de la connexion (retry) côté application.

2. **Persistance des données**  
   Par défaut, les données MySQL stockées dans un conteneur sont perdues si celui‑ci est recréé. Il faut donc utiliser des volumes Docker pour assurer la persistance des données (à ajouter dans la définition du service `db`).

3. **Gestion des identifiants et paramètres**  
   Plutôt que coder en dur les mots de passe et paramètres de connexion dans le code, il convient d’utiliser des variables d’environnement (dans le code et dans `docker-compose.yml`). 

***

## 6. Améliorations demandées (évaluation)

### Niveau 1

- Ajouter un volume Docker pour la base MySQL, afin de rendre les données persistantes (section `volumes` du service `db`). 
- Utiliser des variables d’environnement dans `docker-compose.yml` pour tous les paramètres sensibles (mot de passe MySQL, nom de base, etc.) et les consommer dans l’application. 

### Niveau 2

- Ajouter un endpoint `/health` dans l’application, retournant un statut simple indiquant si le service est opérationnel.
- Améliorer la gestion des erreurs de connexion à la base (messages explicites, gestion des échecs).  

### Niveau 3

- Optimiser la construction de l’image (par exemple en réduisant la taille de l’image, en limitant les fichiers copiés, etc.).
- Ajouter un fichier `.dockerignore` pour exclure les fichiers inutiles du contexte de build (par exemple `node_modules`, fichiers temporaires).  

***

## 7. Intérêt du projet

Ce mini‑projet vous permet de manipuler concrètement :  
- les notions de réseau et de résolution de noms au sein d’un environnement Docker (utilisation des noms de services comme DNS) ; 
- la gestion des volumes et des données persistantes dans un environnement conteneurisé ; 
- les bonnes pratiques de base du DevOps : séparation des rôles (application / base), configuration via variables d’environnement, observation des problèmes de démarrage et de dépendances entre services. 