# Atelier 6 : Le cache et les Inputs

L'une de ses features majeures de Nx est le Computation Caching : Nx ne réexécute jamais deux fois la même tâche si les 
fichiers n'ont pas changé.

## 1. Essai du cache

Commençons par observer le comportement par défaut de Nx sur notre librairie partagée.

Lancez le build de la librairie pour la première fois :

```bash
nx build shared-types
```

*Notez le temps d'exécution.*

Relancez exactement la même commande immédiatement sans modifier le code :

```bash
nx build shared-types
```

*Résultat :* Vous devriez voir la mention [Match]. Nx a mis seulement quelques millisecondes car il a simplement 
ressorti le résultat du cache.

## 2. Configurer les targetDefaults

Pour aller plus loin et optimiser ce cache, nous allons prendre le contrôle dans le fichier `nx.json` à la racine du 
workspace. Modifiez la section `targetDefaults` pour configurer le cache des targets `build` :

```json
"targetDefaults": {
  "build": {
    "cache": true,
    "dependsOn": ["^build"],
    "inputs": ["production", "^production"]
  }
},
```

Décryptage :
* `cache`: On active explicitement le cache pour la target `build`.
* `inputs`: Pour calculer le hash du cache, Nx regardera un groupe 
de fichiers appelé `production` dans le projet courant, ainsi que le 
groupe `production` de ses dépendances (le `^` signifie "les dépendances de ce projet").

## 3. Visualiser les inputs via le Graph

Nx propose un outil visuel pour comprendre exactement ce qui déclenche une tâche.

Lancez le graphe : 
```
nx graph
```

1. Dans le menu de gauche, changez la vue de "Projects" à "Tasks".
2. Cliquez sur adopt-a-cat:build.
3. Cliquez sur le nœud généré pour ouvrir le panneau latéral. Vous verrez la 
section **Inputs**. On peut constater que le build de votre 
frontend dépend de ses propres fichiers ET des fichiers de `shared-types` grâce à la règle `^production`.

## 4. Test avec un README

Par défaut, Nx est prudent : si n'importe quel fichier change dans le dossier d'un projet, 
il considère que le projet a changé et invalide le cache. Faisons le test :
* Générez un cache valide : `nx build adopt-a-cat`.
* Créez ou modifiez un fichier `README.md` dans `apps/adopt-a-cat/README.md`.
* Relancez le build : `nx build adopt-a-cat`.

*Constat* : Nx relance la compilation alors que le README n'influence pas le code final.

## 5. Affiner les namedInputs
   
Corrigez les `namedInputs` afin d'exclure le README du calcul du cache. Testez 
à nouveau une modification du README et observez que le cache est désormais respecté.

## 6. Test du cache distribué

Si vous aviez activé Nx Cloud dans le repository à l'atelier 5, vous pouvez observer le cache distribué dans le CI.
Pour cela il suffit de relancer le workflow GitHub Actions. Vous verrez que les tâches sont instantanées grâce au 
cache distribué de Nx Cloud.