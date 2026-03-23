

***
# Atelier DevOps – Pipeline CI/CD complet avec GitHub Actions et Docker

## 1. Objectifs

À l’issue de ce TP, vous devrez être capables de :  
- configurer un workflow GitHub Actions qui :  
  - teste le code à chaque push (CI) ;  
  - construit une image Docker de l’application ;  
  - pousse automatiquement cette image vers un registre (Docker Hub ou GHCR). [docs.github](https://docs.github.com/en/actions/publishing-packages/publishing-docker-images)
- comprendre la logique d’un pipeline **CI → build → push** dans un seul workflow. [github](https://github.com/docker/build-push-action)

***

## 2. Contexte

Vous disposez déjà d’un dépôt GitHub contenant :  
- une petite application (par exemple Node.js) ;  
- un `Dockerfile` fonctionnel à la racine du projet ;  
- idéalement, un workflow CI de base (tests) réalisé dans l’atelier précédent. [docs.github](https://docs.github.com/actions/guides/building-and-testing-nodejs)

L’objectif de cet atelier est de mettre en place un workflow **unique** qui :  
1. Vérifie le code (tests).  
2. Si tout est OK, construit une image Docker.  
3. Pousse cette image vers un registre (Docker Hub ou GitHub Container Registry). [oneuptime](https://oneuptime.com/blog/post/2026-02-20-github-actions-docker-build-push/view)

***

## 3. Pré-requis

1. Un `Dockerfile` valide (testé localement si possible).  
2. Un compte sur Docker Hub **ou** l’utilisation de GitHub Container Registry (GHCR). [docs.github](https://docs.github.com/actions/guides/publishing-docker-images)
3. Des secrets configurés dans le dépôt GitHub :  
   - Pour Docker Hub : `DOCKERHUB_USERNAME`, `DOCKERHUB_TOKEN`. [pradumnasaraf](https://pradumnasaraf.dev/blog/dockerhub-githubactions)
   - Pour GHCR : le `GITHUB_TOKEN` intégré suffit dans la plupart des cas. [docs.github](https://docs.github.com/en/actions/publishing-packages/publishing-docker-images)

***

## 4. Partie 1 – Rappel : Dockerfile de l’application

Assurez-vous d’avoir un `Dockerfile` par exemple :

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install --only=production

COPY . .

EXPOSE 3000

CMD ["node", "app.js"]
```

Test local (optionnel) :

```bash
docker build -t mon-app-local .
docker run -p 3000:3000 mon-app-local
```

***

## 5. Partie 2 – Création du workflow CI/CD

Dans votre dépôt GitHub, créez (ou modifiez) le fichier :

```text
.github/workflows/ci-cd.yml
```

### 5.1. Spécifications du workflow

Le workflow doit :

- se déclencher sur :  
  - `push` sur la branche `main`,  
  - et `push` de tags versionnés (par ex. `v1.0.0`). [github](https://github.blog/enterprise-software/ci-cd/build-ci-cd-pipeline-github-actions-four-steps/)
- contenir un **job CI** qui :  
  - récupère le code,  
  - installe les dépendances,  
  - lance les tests (ou un script de vérification). [docs.github](https://docs.github.com/en/actions/tutorials/build-and-test-code)
- contenir un **job CD** qui :  
  - dépend du job CI (il ne s’exécute que si la CI est OK),  
  - construit l’image Docker,  
  - pousse l’image vers un registre (Docker Hub ou GHCR) avec un tag explicite (par exemple `latest` + tag basé sur le commit ou le tag Git). [github](https://github.com/docker/build-push-action)

Vous pouvez utiliser l’action officielle `docker/build-push-action` pour le build + push. [github](https://github.com/marketplace/actions/build-and-push-docker-images)

***

## 6. Partie 3 – Déclenchement et validation

1. Commitez et poussez `ci-cd.yml` :  
   ```bash
   git add .github/workflows/ci-cd.yml
   git commit -m "Ajout pipeline CI/CD (tests + Docker push)"
   git push
   ```

2. Vérifiez dans l’onglet **Actions** :  
   - le job de tests s’exécute ; [docs.github](https://docs.github.com/actions/quickstart)
   - si la CI passe, le job de build/push de l’image se lance. [notes.kodekloud](https://notes.kodekloud.com/docs/GitHub-Actions/Continuous-Integration-with-GitHub-Actions/Workflow-Docker-Push/page)

3. Créez un tag pour simuler une release :  
   ```bash
   git tag v1.0.0
   git push origin v1.0.0
   ```
   Vérifiez que le workflow se déclenche également sur ce tag.

4. Contrôlez dans votre registre (Docker Hub ou GHCR) que :  
   - une image a bien été poussée ;  
   - les tags attendus sont présents (`latest`, `v1.0.0`, ou autre selon votre configuration). [docs.github](https://docs.github.com/actions/guides/publishing-docker-images)

***

## 7. Partie 4 – Analyse et questions

Dans un court texte, expliquez :

1. Quels événements déclenchent votre pipeline (section `on:`). [github](https://github.blog/enterprise-software/ci-cd/build-ci-cd-pipeline-github-actions-four-steps/)
2. Ce que fait le job de **CI** (liste des steps : checkout, install, tests). [docs.github](https://docs.github.com/actions/guides/building-and-testing-nodejs)
3. Ce que fait le job de **CD** (login registre, build, push, tags). [github](https://github.com/docker/build-push-action)
4. Pourquoi il est important que le job CD dépende du job CI (ne pas déployer si les tests échouent). [blog.devops](https://blog.devops.dev/implementing-ci-cd-with-github-actions-and-docker-2da457c832be)

***

## 8. Livrables attendus

Pour chaque étudiant / binôme, on attend :

- un dépôt GitHub avec :  
  - un `Dockerfile` fonctionnel ;  
  - un workflow `ci-cd.yml` ;  
- un pipeline CI/CD qui :  
  - teste le projet à chaque `push` ;  
  - construit et pousse une image Docker lorsque les tests réussissent ; [oneuptime](https://oneuptime.com/blog/post/2026-02-20-github-actions-docker-build-push/view)
- une capture ou preuve de l’image existante dans le registre (Docker Hub ou GHCR).  

***