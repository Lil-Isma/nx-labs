# Atelier 9 : Nx Release

Dans cet atelier nous allons créer deux libraries et les versionner avec Nx Release:
* `@felitech/is-odd`: une librairie qui exporte une fonction `isOdd` pour vérifier si un nombre est impair.
* `@felitech/is-even`: une librairie qui exporte une fonction `isEven` pour vérifier si un nombre est pair.

Parce qu'on est DRY (Don't Repeat Yourself) à l'extrême, `is-even` va dépendre de `is-odd` pour faire son travail,
sinon ça serait trop facile.

## 1. Créer les deux librairies

Créez les deux libraries sans oublier de spécifier `--publishable` et `--importPath` pour qu'elles soient prêtes 
à être publiées.

Exemple :
```bash
nx g @nx/js:lib libs/packages/is-odd --publishable --importPath=@felitech/is-odd
```

Pour être au plus simple, utiliser `tsc` (typescript compiler) comme builder pour les deux librairies, mais
n'importe quel builder peut faire l'affaire.

## 2. Configuration de Nx Release

Maintenant nous allons configurer Nx Release. Toute la configuration se fait dans `nx.json`. Normalement, Nx a déjà
ajouté cette partie:
```json
"release": {
    "version": {
      "preVersionCommand": "npx nx run-many -t build"
    }
}
```

Nx va donc build toutes les libraries avant de calculer la nouvelle version. Activons maintenant conventionalCommits
pour que Nx puisse calculer la nouvelle version à partir de nos messages de commit.
```json
"release": {
    "version": {
      "preVersionCommand": "npx nx run-many -t build",
      "conventionalCommits": true
    },
    "projects": ["is-odd", "is-even"]
}
```

Faites un commit de type feature (ex: `git commit -m "feat: introduce is-odd and is-even libraries"`) puis lancez
la commande de release initiale en mode dry-run pour voir ce qui se passerait :
```bash
nx release --first-release --dry-run
```

Si c'est tout bon, on peut lancer la commande de release réelle :
```bash
nx release --first-release
```

Vérifiez qu'on a bien un nouveau commit `chore(release): publish 0.1.0` et qu'un tag `v0.1.0` a été créé.

## 3. Faire un breaking change

Modifions la fonction `isOdd` pour qu'elle throw une erreur si l'input n'est pas un nombre. 
```typescript
export function isOdd(x: number): boolean {
  if (typeof x !== 'number') {
    throw new Error('Input must be a number');
  }
  return x % 2 === 1;
}
```
  
Faites un commit de type breaking change 
```
git commit -m "feat: add input validation to is-odd" -m "BREAKING CHANGE: is-odd now throws an error if the input is not a number"
```

Puis lancer une nouvelle release
```bash
nx release
```