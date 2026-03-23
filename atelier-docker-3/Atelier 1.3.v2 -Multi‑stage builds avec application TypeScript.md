

***
# Mini‑projet DevOps Docker : Multi‑stage builds avec application TypeScript

## 1. Objectif du projet

L’objectif de ce mini‑projet est de vous faire pratiquer la conteneurisation d’une application Node.js écrite en TypeScript en utilisant les **multi‑stage builds** Docker.  

À l’issue du projet, vous devrez être capables de :  
- conteneuriser une application Node.js/TypeScript ;  
- utiliser un multi‑stage build Docker pour compiler le code TypeScript vers JavaScript ;  
- construire une image finale qui ne contient que le code compilé (`dist/`), les fichiers statiques (`public/`) et les dépendances de production ;  
- comparer une image “naïve” et une image multi‑stage en termes de taille et de contenu.  

L’objectif global est de **réduire la taille de l’image** et de **séparer clairement** la phase de build de la phase d’exécution.

***

## 2. Projet fourni

On vous fournit le squelette suivant :

```text
ts-app/
│
├── src/
│   └── index.ts
├── public/
│   └── index.html
├── package.json
├── tsconfig.json
└── Dockerfile.single
```

### 2.1. `src/index.ts`

```ts
import express from 'express';
import path from 'path';

const app = express();
const port = 3000;

// Servir les fichiers statiques (public)
app.use(express.static(path.join(__dirname, '..', 'public')));

app.get('/api/hello', (_req, res) => {
  res.json({ message: 'Hello from TypeScript + Docker multi-stage!' });
});

app.listen(port, () => {
  console.log(`Server listening on port ${port}`);
});
```

### 2.2. `public/index.html`

Contenu minimal, par exemple :

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>TS Docker Multi-stage Demo</title>
  </head>
  <body>
    <h1>TypeScript + Docker multi-stage</h1>
    <p>Static file served from the public folder.</p>
  </body>
</html>
```

### 2.3. `package.json`

```json
{
  "name": "ts-app",
  "version": "1.0.0",
  "scripts": {
    "build": "tsc",
    "start": "node dist/index.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  },
  "devDependencies": {
    "@types/express": "^4.17.21",
    "@types/node": "^20.0.0",
    "typescript": "^5.6.0"
  }
}
```

### 2.4. `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "rootDir": "src",
    "outDir": "dist",
    "strict": true,
    "esModuleInterop": true
  }
}
```

### 2.5. `Dockerfile.single` (version naïve)

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY . .

RUN npm install
RUN npm run build

EXPOSE 3000

CMD ["npm", "start"]
```

Ce Dockerfile :  
- mélange build et exécution dans un seul stage ;  
- embarque les sources TypeScript (`src/`) et les dépendances de développement dans l’image finale.

### 2.6. `docker-compose.yml` (pour faciliter l’exécution)

Ajoutez à la racine du projet (`ts-app/`) un fichier `docker-compose.yml` pour lancer facilement l’application :

```yaml

services:
  ts-app-single:
    build:
      context: .
      dockerfile: Dockerfile.single
    container_name: ts_app_single
    ports:
      - "3000:3000"

  ts-app-multi:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: ts_app_multi
    ports:
      - "3001:3000"
```

Ce fichier vous permettra de lancer à la fois l’image single‑stage (port 3000) et l’image multi‑stage (port 3001) pour comparaison.

***

## 3. Partie 1 – Image single‑stage (référence)

### 3.1. Construction de l’image naïve

Dans le répertoire `ts-app` :

```bash
docker build -t ts-app-single -f Dockerfile.single .
```

ou, avec Docker Compose :

```bash
docker compose build ts-app-single
```

### 3.2. Exécution

Sans Compose :

```bash
docker run -p 3000:3000 ts-app-single
```

Avec Compose :

```bash
docker compose up ts-app-single
```

### 3.3. Vérifications

- `http://localhost:3000/` doit servir la page HTML du dossier `public/`.  
- `http://localhost:3000/api/hello` doit renvoyer un JSON.  

Notez la taille de l’image :

```bash
docker images ts-app-single
```

Cette taille servira de référence pour comparaison.

***

## 4. Partie 2 – Dockerfile multi‑stage à écrire

Vous devez créer un **nouveau fichier** `Dockerfile` (sans extension) utilisant un multi‑stage build avec au moins deux stages.

### 4.1. Stage 1 – Build TypeScript

Exigences :

- Basé sur l’image `node:20-alpine`.  
- Tâches attendues :  
  - copier `package.json` (et éventuellement `package-lock.json` s’il existe) ;  
  - installer les dépendances nécessaires à la compilation (`npm install`) ;  
  - copier le dossier `src/` et les fichiers nécessaires au build (`tsconfig.json`, éventuellement `public/` si besoin) ;  
  - exécuter `npm run build` pour générer le dossier `dist/`.  

L’objectif de ce stage est uniquement de produire les artefacts compilés (`dist/`).

### 4.2. Stage 2 – Runtime (image finale)

Exigences :

- Basé sur `node:20-alpine` (ou une autre image légère équivalente).  
- Tâches attendues :  
  - copier `package.json` ;  
  - installer uniquement les dépendances de **production** (`npm ci --omit=dev` ou `npm install --only=production`) ;  
  - copier depuis le stage de build :  
    - le dossier compilé `dist/` ;  
    - le dossier `public/` (pour les fichiers statiques) ;  
  - définir la commande de démarrage :  
    ```dockerfile
    CMD ["node", "dist/index.js"]
    ```  

Contraintes importantes :  

- Les sources TypeScript (`src/`) ne doivent **pas** être présentes dans l’image finale.  
- Les dépendances de développement (TypeScript, `@types`, etc.) ne doivent **pas** être installées dans l’image finale.  

***

## 5. Partie 3 – Comparaison et conclusion

### 5.1. Construction de l’image multi‑stage

Dans le répertoire `ts-app` :

```bash
docker build -t ts-app-multi .
```

ou, avec Docker Compose :

```bash
docker compose build ts-app-multi
```

### 5.2. Comparaison des tailles

```bash
docker images ts-app-single
docker images ts-app-multi
```

Relevez la taille de chaque image et calculez la différence.

### 5.3. Tests de fonctionnement

Lancez l’image multi‑stage :

Sans Compose :

```bash
docker run -p 3001:3000 ts-app-multi
```

Avec Compose (lance les deux services pour comparaison) :

```bash
docker compose up
```

Vérifiez :

- `http://localhost:3001/` doit servir la même page HTML que l’image single‑stage.  
- `http://localhost:3001/api/hello` doit répondre comme avant.  

### 5.4. Analyse écrite

Dans un court texte en Markdown, expliquez :  

- quels fichiers se trouvent dans l’image finale multi‑stage (ex. `dist/`, `public/`, dépendances de production) ;  
- ce qui a été éliminé par rapport à l’image single‑stage (`src` TypeScript, devDependencies, outils de build) ;  
- en quoi cela réduit la taille de l’image et améliore la sécurité et la portabilité (moins de code inutile, pas d’outils de build en production, surface d’attaque réduite).  