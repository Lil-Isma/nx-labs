# Atelier 10 : Créer un générateur custom

Dans un monorepo avec de nombreux développeurs, utiliser la commande standard `nx g @nx/react:lib` peut poser problème.
Les développeurs oublient souvent d'ajouter les tags (`type:*`, `scope:*`) indispensables pour faire respecter nos 
règles d'architecture (cf. Atelier 9).

La solution : créer notre propre générateur (un "wrapper") qui va encapsuler le générateur officiel et imposer nos standards.

> [!TIP]
> La documentation de Nx: [Local Generators](https://nx.dev/docs/extending-nx/local-generators).

## 1. Créer un plugin local

Dans Nx, les générateurs personnalisés vivent dans un plugin local. Créons ce plugin que nous appellerons internal-tools :
```bash
nx add @nx/plugin
nx g @nx/plugin:plugin tools/internal-tools 
```

## 2. Initialiser le générateur

Générez votre générateur, nommé corporate-lib :
```
nx g @nx/plugin:generator tools/internal-tools/src/generators/corporate-lib
```

Ouvrez le dossier libs/internal-tools/src/generators/corporate-lib. Vous pouvez supprimer le dossier `files` qui s'y 
trouve, nous n'en aurons pas besoin.

## 3. Définir les paramètres d'entrée

Nous voulons que notre générateur demande obligatoirement un `type` et un `scope` à l'utilisateur lors de son exécution.

**Todo** : Éditez le fichier `schema.json` pour y ajouter ces deux propriétés et les rendre requises. Éditez
`schema.d.ts` pour que les types soient corrects.

> [!TIP]
> Voici un exemple de [schema.json](https://github.com/nrwl/nx/blob/master/packages/plugin/src/generators/create-package/schema.json)
> dans le repository officiel de Nx pour vous inspirer.

## 4. Implémenter la logique

La logique du générateur s'implémente dans le fichier `corporate-lib.ts`

L'objectif est d'invoquer le générateur officiel de React en arrière-plan avec une configuration prédéfinie, puis de modifier la
configuration générée (le project.json) pour y injecter nos tags.

### Exemple: créer une library avec le generator officiel

```typescript
import { libraryGenerator } from '@nx/react';

await libraryGenerator(tree, {
  name: 'my-lib',
  directory: 'cats',
  style: 'scss',
  linter: 'eslint',
  unitTestRunner: 'jest',
  bundler: 'vite',
});
```

## Exemple: modifier les tags d'un projet

```typescript
import {
  readProjectConfiguration,
  updateProjectConfiguration,
} from '@nx/devkit';

const projectName = 'my-lib';
const projectConfig = readProjectConfiguration(tree, projectName);
projectConfig.tags = ['type:ui'];
updateProjectConfiguration(tree, projectName, projectConfig);
```

## 5. Tester le générateur

Testez le générateur en local :
```bash
nx g @felitech/internal-tools:corporate-lib --name=my-new-lib --scope=cats --type=ui --directory=libs/cats/my-new-lib
```
