

***

***

# Mini‑projet DevOps Sécurité : Exploitation et sécurisation d’une image Docker vulnérable (RCE)

## 1. Objectifs du projet

Ce mini‑projet a un double objectif :  

1. **Phase 1 – Audit / exploitation**  
   - Exécuter une application Node.js conteneurisée.  
   - Mettre en évidence une vulnérabilité de type Remote Command Execution (RCE).  
   - Analyser les risques liés au code et à l’image Docker.  

2. **Phase 2 – Correction / sécurisation**  
   - Corriger l’application pour supprimer la RCE.  
   - Durcir l’image Docker (réduire la surface d’attaque, ne plus tourner en root, etc.).  
   - Mettre à jour le `docker-compose.yml` en conséquence.  

***

## 2. Projet fourni

On vous fournit le répertoire suivant :

```text
vulnerable-app/
│
├── app.js
├── package.json
├── Dockerfile
└── docker-compose.yml
```

### 2.1. Application Node.js (version vulnérable)

```js
const express = require('express');
const app = express();

// endpoint dangereux
app.get('/exec', (req, res) => {
  const { exec } = require('child_process');
  exec(req.query.cmd, (err, stdout, stderr) => {
    res.send(stdout || stderr);
  });
});

app.get('/', (req, res) => {
  res.send('Vulnerable App 🚨');
});

app.listen(3000, () => console.log('Running on port 3000'));
```

L’endpoint `/exec` exécute directement le contenu du paramètre `cmd` dans un shell via `child_process.exec`, sans contrôle ni filtrage : c’est une RCE.

### 2.2. Fichier `package.json`

```json
{
  "name": "vulnerable-app",
  "version": "1.0.0",
  "dependencies": {
    "express": "4.17.1"
  }
}
```

### 2.3. Dockerfile (version vulnérable)

```dockerfile
FROM node:18

WORKDIR /app

COPY . .

RUN npm install

# outils inutiles + dangereux
RUN apt-get update && apt-get install -y curl nano net-tools

EXPOSE 3000

CMD ["node", "app.js"]
```

L’image :  
- utilise `node:18` complet ;  
- installe des outils supplémentaires (curl, nano, net-tools) ;  
- exécute l’application en root dans le conteneur.  

### 2.4. Fichier `docker-compose.yml` (fourni, pour lancement)

```yaml
services:
  app:
    build: .
    container_name: vulnerable_app
    ports:
      - "3000:3000"
```

Ce fichier Compose permet de construire et lancer facilement le conteneur.

***

## 3. Phase 1 – Mise en route et exploitation

### 3.1. Lancement avec Docker Compose

Dans le répertoire `vulnerable-app` :

```bash
docker compose up --build
```

Vérifiez que l’application répond sur :

```text
http://localhost:3000/
```

### 3.2. Exploitation de l’endpoint `/exec`

Objectif : démontrer que l’on peut exécuter des commandes système dans le conteneur via HTTP.

À l’aide du navigateur ou de `curl` :

1. **Test 1 – Utilisateur dans le conteneur**

   ```text
   http://localhost:3000/exec?cmd=whoami
   ```

   ou :

   ```bash
   curl "http://localhost:3000/exec?cmd=whoami"
   ```

   Notez le résultat (par exemple `root`).

2. **Test 2 – Lecture de fichier système**

   Encodez l’espace dans l’URL avec `%20` :

   ```bash
   curl "http://localhost:3000/exec?cmd=cat%20/etc/passwd"
   ```

   Notez si le contenu du fichier `/etc/passwd` est renvoyé.

3. **Test 3 – Commande d’administration**

   ```bash
   curl "http://localhost:3000/exec?cmd=apt-get%20update"
   ```

   Observez si la commande semble exécutée (sortie d’`apt-get`).

***

## 4. Phase 2 – Analyse et description de la vulnérabilité

Vous devez produire une courte analyse (dans votre compte‑rendu) :

1. Expliquer le fonctionnement du code de `/exec` et pourquoi il s’agit d’une RCE.  
2. Décrire les tests réalisés (URLs / commandes, résultats observés).  
3. Identifier les risques principaux : lecture de fichiers, installation de logiciels, scans réseau, etc.  
4. Expliquer en quoi le fait d’exécuter le processus en root avec des outils réseau installés aggrave l’impact potentiel de la vulnérabilité.  

***

## 5. Phase 3 – Corrections à implémenter

Dans cette phase, vous devez modifier le projet pour le rendre significativement plus sûr.

### 5.1. Correction côté application (Node.js)

Objectif : supprimer la RCE, tout en gardant un endpoint d’administration limité et sans exécution de commandes arbitraires.

Exigences minimales :

1. **Supprimer l’usage direct de `child_process.exec` avec des données utilisateur.**  
2. Redéfinir `/exec` ou le remplacer par un endpoint plus sûr, par exemple :  
   - `/exec?cmd=status` qui renvoie un simple JSON (status, date, version) **sans** passer par le shell ;  
   - toute autre commande non prévue doit être rejetée avec une erreur 400.  
3. Ajouter un endpoint `/health` qui renvoie un statut JSON simple (ex. `{ status: "up" }`).  

Exemple d’esprit attendu (à adapter librement) :

```js
app.get('/exec', (req, res) => {
  const { cmd } = req.query;

  if (cmd === 'status') {
    return res.json({
      status: 'ok',
      time: new Date().toISOString()
    });
  }

  if (cmd === 'version') {
    return res.json({
      app: 'vulnerable-app',
      version: '2.0.0'
    });
  }

  return res.status(400).json({ error: 'Commande non autorisée' });
});

app.get('/health', (req, res) => {
  res.json({ status: 'up' });
});
```

Aucune commande système ne doit plus être exécutée à partir d’entrées utilisateur.

### 5.2. Durcissement de l’image Docker

Objectif : réduire la surface d’attaque et éviter d’exécuter l’application en root.

Modifiez le `Dockerfile` pour :

1. Partir d’une image plus légère, par exemple `node:18-slim`.  
2. Ne plus installer d’outils inutiles (`curl`, `nano`, `net-tools`, etc.).  
3. Créer un utilisateur applicatif non‑root (ex. `appuser`) et lancer l’application avec cet utilisateur (`USER appuser`).  
4. Optionnel : limiter les fichiers copiés (copier d’abord `package*.json`, installer, puis copier le reste du code).

Exemple d’esprit attendu :

```dockerfile
FROM node:18-slim

WORKDIR /app

COPY package*.json ./
RUN npm install --only=production

COPY . .

RUN useradd -m appuser
USER appuser

EXPOSE 3000
CMD ["node", "app.js"]
```

### 5.3. Mise à jour de `docker-compose.yml`

Vous pouvez conserver la structure simple :

```yaml
version: '3.8'

services:
  app:
    build: .
    container_name: secure_app
    ports:
      - "3000:3000"
```

Attendu : après modification, le conteneur doit toujours démarrer et répondre sur `/` et `/health`, mais les anciennes URLs d’exploitation (`/exec?cmd=whoami`, etc.) ne doivent plus permettre d’exécuter des commandes système.

***

## 6. Phase 4 – Validation de la sécurisation

Après vos modifications :

1. Reconstruisez et relancez les conteneurs :

   ```bash
   docker compose up --build
   ```

2. Vérifiez :  
   - que `/` et `/health` répondent correctement ;  
   - que les anciennes URLs d’attaque renvoient des erreurs contrôlées (par exemple 400, ou un message “commande non autorisée”).  

3. Confirmez que :  
   - il n’est plus possible d’exécuter `whoami`, `cat /etc/passwd`, `apt-get update` via `/exec` ;  
   - l’utilisateur actif dans le conteneur n’est plus `root` (par exemple en testant `whoami` à l’intérieur du conteneur via `docker exec`, dans un contexte de validation en local, pas via l’API).

***

## 7. Travail à rendre

Votre rendu doit contenir :

1. **Compte‑rendu d’exploitation** (Phase 1 et 2)  
   - description de la vulnérabilité ;  
   - tests réalisés et résultats ;  
   - analyse des risques.  

2. **Description des corrections implémentées**  
   - modifications apportées au code Node.js (suppression de la RCE, nouveaux endpoints) ;  
   - modifications apportées au Dockerfile (image, utilisateur, outils, etc.) ;  
   - mention de la mise à jour de `docker-compose.yml` si nécessaire.  

3. **Preuve de bon fonctionnement après correction**  
   - exemples de requêtes vers `/`, `/health` et `/exec` (nouvelle version) ;  
   - confirmation que les anciennes commandes d’attaque ne sont plus exécutées.  

