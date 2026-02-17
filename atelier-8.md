# Atelier 8 : Module Boundaries

> [!NOTE]
> Vous pouvez pull la branche `atelier-7` pour récupérer le code de départ de cet atelier.
> ```bash
> git remote add Lil-Isma git@github.com:Lil-Isma/felitech.git
> git fetch Lil-Isma
> git checkout -b atelier-8 Lil-Isma/atelier-7
> ```

Dans l'atelier précédent, nous avons découpé notre application en plusieurs librairies (`feature`, `ui`, `data-access`, etc.). 
C'est un bon début, mais rien n'empêche physiquement une librairie "UI" d'importer une librairie "Feature", ce 
qui créerait des dépendances circulaires et une architecture "spaghetti" qu'on aimerait éviter.

## 1. Créer et assigner des tags

Pour imposer des règles, nous devons d'abord classifier nos projets. Nx utilise deux dimensions principales pour les tags :
* `type:*` : Quelle est la nature du projet ? (`feature`, `ui`, `data-access`, `util`, `app`)
* `scope:*` : À quel domaine métier appartient-il ? (`shared`, `cats`, `admin`, etc.)

Commencez donc par ajouter des tags à vos projets en éditant les fichiers `project.json`.

Par exemple pour la library UI:
```json
{
  "name": "ui-design-system",
  // ...
  "tags": ["type:ui", "scope:shared"]
}
```

## 2. Définir les règles de dépendances

Dans une architecture propre, la dépendance doit couler dans un seul sens : App -> Feature -> Data Access -> UI -> Util.
1. `app` peut tout importer.
2. `feature` peut tout importer
3. `ui` peut importer `ui` et `util`
4. `data-access` peut importer `data-access` et `util`
5. `util` ne peut importer que d'autres `util`

A l'aide de la [documentation officielle](https://nx.dev/docs/features/enforce-module-boundaries#configure-boundary-rules),
configurez les règles de dépendances dans le fichier `eslint.config.mjs` pour faire respecter cette architecture.

Pour valider que les règles sont bien appliquées, ne pas hésiter à faire des imports
interdits dans votre code et vérifier que ESLint remonte une erreur
```
nx affected -t lint
```

## 3. Mettre à jour les dossiers scannés par Tailwind

Si vous utilisez Tailwind CSS, il faut lui indiquer où se trouvent les fichiers à scanner pour générer les classes CSS.

Il ne faut donc pas oublier de mettre à jour le fichier `apps/adopt-a-cat/src/styles.css` pour y ajouter les chemins 
vers les autres librairies qui contiennent des composants utilisant Tailwind.

Par exemple :

```css
@import 'tailwindcss';

@source '../../../libs/design-system/ui-design-system/src/';
```

## 4. (Bonus) Automatiser la mise à jour des source avec un sync generator

Nx fournit un **sync generator** pour synchroniser les sources Tailwind entre les différentes librairies.

[Automating @source with Nx Sync Generators](https://nx.dev/blog/setup-tailwind-4-angular-nx-workspace#automating-source-with-nx-sync-generators)