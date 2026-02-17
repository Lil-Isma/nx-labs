# Atelier 4 : Automatisation (GitHub Actions)

> [!NOTE]
> Vous pouvez pull la branche `atelier-3` pour récupérer le code de départ de cet atelier.
> ```bash
> git remote add Lil-Isma git@github.com:Lil-Isma/felitech.git
> git fetch Lil-Isma
> git checkout -b atelier-4 Lil-Isma/atelier-3
> ```

L'objectif de cet atelier est d'automatiser la validation de notre code. À chaque modification, nous voulons nous assurer que toutes les applications et librairies fonctionnent correctement.

Pour l'instant, nous allons configurer une CI "standard" qui vérifie l'intégralité du monorepo.

## 1. Préparer le Workflow GitHub

Nx a potentiellement déjà généré un dossier `.github/workflows` à la racine.
* Si oui : Ouvrez `.github/workflows/ci.yml`
* Si non : Créez le fichier `.github/workflows/ci.yml`.

Nous allons nous assurer que la CI exécute les tâches lint, test et build sur tous les projets (sans optimisation pour le moment).

```yaml
name: CI

on:
  push:
    branches:
      - main
  pull_request:

permissions:
  actions: read
  contents: read

jobs:
  main:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          filter: tree:0
          fetch-depth: 0

      # Installation de Node.js
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      # Installation des dépendances
      - run: npm ci

      # Vérifier le formatage du code (Prettier)
      - run: npx nx format:check

      # Lancer les tâches sur TOUS les projets
      - run: npx nx run-many -t lint,test,build
```      
     
## 2. Pousser le code et visualiser le CI

Rendez-vous sur votre repository, et cliquez sur l'onglet Actions en haut. Cliquez
sur le workflow en cours d'exécution. Vous pourrez observer les différentes étapes 
se lancer :
* La vérification du formatage (`format:check`)
* La vérification de la configuration (`sync:check`)
* L'exécution de `run-many` : Vous verrez que Nx lance en parallèle `lint`, `test` et build pour votre app React,
l'API NestJS et la librairie `shared-types`