# Atelier 5 : Optimiser le CI avec Nx Affected

Dans l'atelier précédent, nous avons mis en place un CI qui exécute les tests et le build sur tous les projets à chaque modification.

Cependant, dans un monorepo qui grossit, cela devient inefficace. Si on modifie uniquement le frontend, pourquoi tester le backend ?

Nx propose la commande affected qui utilise le "Dependency Graph" (vu dans l'atelier 4) pour déterminer quels projets sont impactés par vos modifications.

## 1. Comprendre le concept en local
   
Pour tester cette fonctionnalité, nous allons simuler une modification.

Ouvrez la définition de `Cat` et ajoutez un simple commentaire ou un espace :

```typescript
// Modification de test
export type Cat = {
// ...
}
```

Comme nous l'avons vu dans le graphe de l'atelier 4, le Frontend et le Backend dépendent tous les deux de cette librairie.

Lancez la commande suivante pour visualiser ce qui est impacté :

```bash
nx graph --affected
```

Vous devriez voir que tous les nœuds sont rouges, car modifier la librairie partagée affecte tout le monde.

Maintenant, annulez cette modification (ou commitez-la), puis modifiez un fichier uniquement dans l'application React. 
Relancez la commande. Vous verrez que le backend ne sera pas sélectionné.

## 2. Exécuter les tâches affectées
   
Au lieu d'utiliser `nx run-many`, nous utilisons `nx affected`. Nx compare l'état actuel de vos fichiers (HEAD) avec 
la branche principale (main) pour déduire ce qu'il doit exécuter.

Essayez cette commande (après avoir modifié un fichier) :

```bash
npx nx affected -t lint build
```

## 3. Mettre à jour le Workflow GitHub
   
Nous allons maintenant optimiser notre fichier `.github/workflows/ci.yml` créé lors de l'atelier 5.

L'objectif est de remplacer `run-many` par `affected`.

Modifiez le fichier .github/workflows/ci.yml :

```yml
name: CI

on:
  push:
    branches:
      - main

env:
  BEFORE_SHA: ${{ github.event.before }}

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
          node-version: 22
          cache: 'npm'

      # Installation des dépendances
      - run: npm ci

      - run: npx nx format:check --base=$BEFORE_SHA --head=HEAD

      - run: npx nx affected -t lint test build --base=$BEFORE_SHA --head=HEAD
```

## 4. Pousser le code et visualiser le CI

Poussez votre code sur GitHub. Allez dans l'onglet Actions et regardez 
l'étape `run: npx nx affected ...`.

## 5. Utiliser l'action officielle Nx pour la CI

Au lieu de configurer nous mêmes `$BEFORE_SHA` et `$HEAD`, nous pouvons utiliser l'action officielle Nx pour la CI : [nx-set-shas](https://github.com/marketplace/actions/nx-set-shas).
Cette action va automatiquement détecter les SHAs de base et de head pour les commandes `nx affected` en fonction du contexte de la pull request ou du push.

Juste après l'étape `actions/checkout`, ajoutez cette étape :

```yaml
- name: Derive appropriate SHAs for base and head for `nx affected` commands
  uses: nrwl/nx-set-shas@v4

- run: |
      echo "BASE: ${{ env.NX_BASE }}"
      echo "HEAD: ${{ env.NX_HEAD }}"
```

Ensuite, il suffit juste de lancer les commandes `affected` sans préciser les options `--base` et `--head`.
Nx utilisera automatiquement les variables d'environnement `NX_BASE` et `NX_HEAD` pour déterminer les commits à comparer.

```yaml
- run: npx nx affected -t lint,test,build
```