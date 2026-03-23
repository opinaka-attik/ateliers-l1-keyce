

***
# Mini‑projet DevOps – GitHub Actions (CI simple sur un projet Node.js)

## 1. Objectifs du mini‑projet

À l’issue de ce mini‑projet, vous devrez être capables de :  
- comprendre la structure d’un workflow GitHub Actions (événements, jobs, steps) ;  
- écrire un fichier `ci.yml` pour une application Node.js ;  
- déclencher automatiquement `npm install` puis `npm test` à chaque `git push` ;  
- interpréter les résultats d’exécution dans l’onglet **Actions** d’un dépôt GitHub.  

***

## 2. Mise en place du projet Node.js

Vous allez d’abord créer un petit projet Node.js qui servira de support à la CI.

### 2.1. Création du projet

Dans un terminal :

```bash
mkdir gha-node-ci
cd gha-node-ci
npm init -y
```

### 2.2. Fichier `app.js`

Créer un fichier `app.js` à la racine du projet :

```js
// app.js
function sum(a, b) {
  return a + b;
}

// Exécution directe pour vérification manuelle
if (require.main === module) {
  console.log("2 + 3 =", sum(2, 3));
}

module.exports = { sum };
```

### 2.3. Fichier de test `app.test.js`

Créer un fichier `app.test.js` :

```js
// app.test.js
const { sum } = require('./app');

function runTests() {
  if (sum(2, 3) !== 5) {
    throw new Error('Test échoué : sum(2, 3) devrait valoir 5');
  }

  console.log('Tous les tests sont passés');
}

runTests();
```

### 2.4. Mise à jour de `package.json`

Ouvrir `package.json` et définir le script `test` :

```json
{
  "name": "gha-node-ci",
  "version": "1.0.0",
  "main": "app.js",
  "scripts": {
    "test": "node app.test.js"
  },
  "dependencies": {}
}
```

Vérification locale :

```bash
npm test
```

Le message attendu est : `Tous les tests sont passés`.

### 2.5. Initialisation Git et dépôt GitHub

Initialiser Git et pousser le projet sur GitHub :

```bash
git init
git add .
git commit -m "Initial commit - projet Node.js simple"
git branch -M main
git remote add origin <URL_DU_REPO_GITHUB>
git push -u origin main
```

***

## 3. Création du workflow GitHub Actions

L’objectif est maintenant de créer un workflow CI qui exécute `npm install` et `npm test` à chaque push sur `main` et sur les Pull Requests vers `main`.

### 3.1. Dossier des workflows

Dans le dossier du projet :

```bash
mkdir -p .github/workflows
```

Créer le fichier `.github/workflows/ci.yml`.

### 3.2. Contenu du fichier `ci.yml`

Recopier et adapter le contenu suivant :

```yaml
name: CI Node.js

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
      - name: Récupérer le code du dépôt
        uses: actions/checkout@v4

      - name: Installer Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Installer les dépendances npm
        run: npm install

      - name: Exécuter les tests
        run: npm test
```

Ce workflow :  
- se déclenche à chaque `push` sur `main` et à chaque Pull Request vers `main` ;  
- récupère le code ;  
- installe Node.js 20 ;  
- installe les dépendances ;  
- lance les tests avec `npm test`.  

***

## 4. Activation et observation de la CI

### 4.1. Commit et push du workflow

Exécuter :

```bash
git add .github/workflows/ci.yml
git commit -m "Ajout workflow CI Node.js"
git push
```

### 4.2. Vérification dans GitHub

1. Ouvrir le dépôt sur GitHub.  
2. Aller dans l’onglet **Actions**.  
3. Vérifier qu’un workflow nommé « CI Node.js » apparaît.  
4. Ouvrir la dernière exécution et consulter :  
   - le statut global (succès / échec) ;  
   - les logs des différentes étapes (checkout, installation, tests).  

En cas d’échec, identifier l’étape en erreur et la cause dans les logs, corriger le problème dans le projet, puis recommitter et pousser pour relancer la CI.

***

## 5. Améliorations à réaliser

Les améliorations suivantes sont demandées pour valider le mini‑projet :

1. **Affichage d’information supplémentaire**  
   Ajouter une étape dans le workflow qui affiche la version de Node.js utilisée :

   ```yaml
      - name: Afficher la version de Node
        run: node -v
   ```

2. **Limitation des déclencheurs**  
   Vérifier que la section `on:` ne déclenche la CI que :  
   - sur `push` vers `main` ;  
   - sur `pull_request` vers `main`.  

3. **Lisibilité des steps**  
   Vérifier que chaque étape possède un `name` explicite (par exemple « Récupérer le code du dépôt », « Installer Node.js », « Exécuter les tests »).

***

## 6. Livrables attendus

Pour chaque apprenant, le mini‑projet sera considéré comme terminé si les éléments suivants sont présents :

1. **Code du projet**  
   - fichiers `app.js`, `app.test.js`, `package.json` conformes à l’énoncé ;  
   - projet fonctionnel en local (`npm test` doit passer).

2. **Configuration GitHub Actions**  
   - présence du dossier `.github/workflows/` ;  
   - présence d’un fichier `ci.yml` valide contenant :  
     - la section `on:` configurée pour `push` et `pull_request` sur `main` ;  
     - un job `build-and-test` avec les étapes décrites (checkout, installation de Node, `npm install`, `npm test`, affichage de la version de Node).

3. **Exécution de la CI**  
   - au moins une exécution réussie du workflow visible dans l’onglet **Actions** du dépôt GitHub.

1. **Brève description écrite** en Markdown
   - déclencheurs du workflow (ce qui apparaît dans `on:`) ;  
   - description des étapes principales du job ;  
   - comportement attendu si les tests échouent ;  
   - intérêt de GitHub Actions pour ce projet (détection automatique des régressions, feedback immédiat, etc.).  