# Atelier 11 : Créer un executor custom

> [!NOTE]
> Vous pouvez pull la branche `atelier-10` pour récupérer le code de départ de cet atelier.
> ```bash
> git remote add Lil-Isma git@github.com:Lil-Isma/felitech.git
> git fetch Lil-Isma
> git checkout -b atelier-11 Lil-Isma/atelier-10
> ```

Dans un vrai monorepo d'entreprise, l'un des plus gros risques est l'inflation de la taille des applications. 
Puisque le code est partagé, un développeur peut facilement introduire une librairie très lourde 
(ex: `lodash` entier, dinosaure comme `moment.js`) dans un composant UI partagé, impactant silencieusement
toutes les applications front qui l'utilisent.

**Todo :** Créer un executor `bundle-checker`.

Cet executor devra analyser le dossier généré par le build de votre application (ex: dist/apps/adopt-a-cat), calculer 
la taille totale des fichiers générés, et faire planter la CI (retourner une erreur) si la taille dépasse une limite 
définie dans le `project.json`.

> [!TIP]
> Lien vers la documentation Nx: [Write a simple executor](https://nx.dev/docs/extending-nx/local-executors)

## 1. Créer un executor custom

Nous allons ajouter cet exécuteur dans notre plugin local internal-tools.

Générez la coquille vide de l'exécuteur :
```bash
nx g @nx/plugin:executor tools/internal-tools/src/executors/bundle-checker
```

## 2. Définir les options

Ouvrez le fichier `schema.json` de votre exécuteur et modifiez le schéma pour qu'il accepte deux options :
* `buildPath` (`string`) : Le chemin vers le dossier à analyser.
* maxSizeKb (`number`) : La taille maximale autorisée.

Mettez à jour les types dans `schema.d.ts` en conséquence.

## 3. Implémenter la logique métier

Ouvrez le fichier executor.ts. La fonction par défaut renvoie simplement `{ success: true }`. Dans Nx, si un executor 
renvoie `false` (ou lève une exception), la tâche échoue et la CI s'arrête.

**TODO :**
* Lisez les options `buildPath` et `maxSizeKb` passées à la fonction.
* Vérifiez si le dossier `buildPath` existe. Si non, renvoyez une erreur expliquant qu'il faut lancer le build d'abord.
* Parcourez récursivement les fichiers du dossier cible avec le module natif fs de Node.js.
* Additionnez la taille de chaque fichier (via `fs.statSync(file).size`).
* Convertissez le total en Kb. Si la taille dépasse `maxSizeKb`, loggez un message d'erreur clair et retournez 
`{ success: false }`. Sinon, retournez `{ success: true }`.

## 4. Configurer la target et la tester

Pour tester votre outil, il faut le configurer sur votre application React (adopt-a-cat).

Ouvrez `apps/adopt-a-cat/project.json` et ajoutez une nouvelle target :
```json
"targets": {
  "check-size": {
    "executor": "@felitech/internal-tools:bundle-checker",
    "dependsOn": ["build"],
    "options": {
      "buildPath": "dist/apps/adopt-a-cat",
      "maxSizeKb": 500
    }
  }
}
```

Lancez la commande :
```bash
nx run adopt-a-cat:check-size
```
