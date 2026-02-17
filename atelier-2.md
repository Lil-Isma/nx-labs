# Atelier 2 : Ajouter un backend (NestJS)

Dans cette partie, nous allons ajouter une application backend à notre monorepo et 
la connecter à notre application.

## 1. Ajouter le plugin NestJS

Tout comme pour React, nous devons ajouter Nest à notre workspace via le plugin officiel.

```bash
nx add @nx/nest
```

## 2. Générer l'API

Nous allons générer une application backend nommée `my-api`.

```bash
nx g @nx/nest:app apps/my-api --linter eslint --unitTestRunner jest --e2eTestRunner none
```

Une fois terminé, votre structure de fichiers devrait maintenant contenir nos deux 
applications dans le dossier `apps`.

## 3. Lancer le tout en parallèle avec `run-many`
   
Au lieu de lancer deux terminaux séparés, nous pouvons lancer plusieurs applications 
en parallèle.

```bash
npx nx run-many -t serve -p my-app,my-api
```

```
-p ... : Les projets concernés (optionnel si on veut tout lancer)
-t serve : La cible (target) à exécuter.
```

Observez le terminal, Nx lance les deux apps simultanément.

## 4. Ajouter une dépendance entre le frontend et le backend

Dans un monorepo, il est possible de créer des dépendances explicites entre les targets. Par exemple, nous pouvons
faire en sorte que le serveur frontend ne démarre que lorsque le backend est opérationnel.

Pour cela, nous allons utiliser la fonctionnalité de **dependsOn** :
```json
{
  "targets": {
    "serve": {
      "dependsOn": ["my-api:serve"]
    }
  }
}
```

Ensuite, lancez juste le frontend :
```bash
nx serve my-app
```

Vous verrez que le backend démarre automatiquement avant le frontend.