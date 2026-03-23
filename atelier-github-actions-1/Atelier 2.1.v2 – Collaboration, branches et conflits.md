
***

# Atelier Git / DevOps – Collaboration, branches et conflits

## 1. Objectifs de l’atelier

À l’issue de ce TP, vous devrez être capables de :  
- initialiser et versionner un projet avec Git ;  
- publier un dépôt sur GitHub ou GitLab ;  
- travailler sur des branches de fonctionnalités (feature branches) ;  
- créer et faire valider une Pull Request / Merge Request ;  
- provoquer puis résoudre un conflit Git simple.  

L’accent est mis sur la **collaboration** et le **workflow**, pas sur la complexité du code.

***

## 2. Contexte

Vous formez une petite équipe DevOps travaillant sur un même projet.  
Le projet est volontairement simple : un fichier texte `app.txt` qui va évoluer au fil des contributions.  

Votre mission : mettre en place un **flux de travail Git propre** pour collaborer sans écraser le travail des autres.

***

## 3. Partie 1 – Initialisation du projet

### Étapes à réaliser

1. Créez un dossier de projet local :

```bash
mkdir devops-git-project
cd devops-git-project
```

2. Initialisez un dépôt Git :

```bash
git init
```

3. Créez un premier fichier d’application :

```bash
echo "Projet DevOps M1" > app.txt
```

4. Effectuez un premier commit :

```bash
git add .
git commit -m "Initial commit"
```

5. Créez un dépôt distant sur GitHub ou GitLab (vide), puis reliez‑le :

```bash
git remote add origin <URL_DU_REPO>
git branch -M main
git push -u origin main
```

***

## 4. Partie 2 – Travail en branches

### Objectif

Chaque étudiant crée une **branche de fonctionnalité** et y effectue des modifications isolées, sans travailler directement sur `main`.

### Étapes

1. Créez une branche de fonctionnalité (par exemple `feature/login`) :

```bash
git checkout -b feature/login
```

2. Modifiez le fichier :

```bash
echo "Ajout fonctionnalité login" >> app.txt
```

3. Enregistrez vos modifications :

```bash
git add app.txt
git commit -m "Ajout fonctionnalité login"
```

4. Poussez votre branche sur le dépôt distant :

```bash
git push origin feature/login
```

Chaque étudiant doit avoir **au moins une branche** `feature/*` avec au moins un commit.

***

## 5. Partie 3 – Pull Request / Merge Request

### Objectif

Proposer vos modifications pour intégration dans la branche `main` via une **PR/MR** (code review).

### Étapes

1. Sur GitHub ou GitLab, créez une Pull Request (PR) ou Merge Request (MR) de votre branche `feature/...` vers `main`.  
2. Renseignez :  
   - un titre clair ;  
   - une description courte de la modification.  
3. Demandez à un autre étudiant de **relire et valider** votre PR/MR.  
4. Une fois la revue effectuée, mergez la PR/MR dans `main` via l’interface.  

Chaque étudiant doit :  
- créer au moins une PR/MR ;  
- relire et commenter au moins une PR/MR d’un camarade.

***

## 6. Partie 4 – Mise en situation de conflit

### Objectif

Comprendre comment un conflit se produit et comment le résoudre proprement.

### Scénario (en binôme)

**Étudiant A :**

```bash
git checkout main
echo "Version A" > app.txt
git commit -am "Modification A"
git push
```

**Étudiant B :**

```bash
git checkout main
echo "Version B" > app.txt
git commit -am "Modification B"
git push   # → doit échouer (divergence)
```

Étudiant B récupère alors les changements et rencontre un conflit :

```bash
git pull origin main
```

Le fichier `app.txt` contient alors quelque chose comme :

```text
<<<<<<< HEAD
Version B
=======
Version A
>>>>>>> origin/main
```

### À faire

1. Modifier `app.txt` manuellement pour obtenir une version cohérente, par exemple :

```text
Version A
Version B
```

2. Marquer la résolution du conflit :

```bash
git add app.txt
git commit -m "Résolution de conflit sur app.txt"
git push
```

***

## 7. Partie 5 – Workflow Git simplifié

### Modèle attendu

- Branche `main` : code stable.  
- Branches `feature/*` : développement des nouvelles fonctionnalités.  

### Cycle de travail recommandé

1. Créer une branche à partir de `main`.  
2. Développer et committer localement.  
3. Pousser la branche sur le dépôt distant.  
4. Ouvrir une PR/MR.  
5. Faire relire (review) et corriger si nécessaire.  
6. Merger dans `main`.  

***

## 8. Livrables attendus

Pour chaque étudiant, on attend :

- un dépôt distant fonctionnel (GitHub ou GitLab) ;  
- au moins **2 commits** ;  
- au moins **1 branche feature** créée et poussée ;  
- au moins **1 PR/MR** créée et mergée ;  
- une **résolution de conflit** réalisée (en binôme ou petit groupe).  

***

## 9. Questions de synthèse

Répondez brièvement aux questions suivantes :

1. Pourquoi travaille‑t‑on dans des branches plutôt que directement sur `main` ?  
2. Qu’est‑ce qu’une Pull Request / Merge Request et à quoi sert‑elle ?  
3. Dans quelles situations les conflits apparaissent‑ils ? Donnez un exemple.  
4. Quels risques si toute l’équipe travaille directement sur `main` sans PR/MR ?  