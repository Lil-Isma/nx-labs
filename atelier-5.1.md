# Atelier 5.1 (bonus) : Se connecter à Nx Cloud

Nx Cloud est une plateforme de Nx qui offre des fonctionnalités de caching, d'exécution distribuée et de 
visualisation des tâches.

## 1. Créer un compte Nx Cloud

Rendez-vous sur https://cloud.nx.app/ et créez un compte. 

Connectez votre repository GitHub à Nx Cloud : https://nx.dev/docs/features/ci-features/github-integration#connect-to-github.

Une fois l'intégration terminée, Nx devrait automatiquement créer une Pull Request dans votre repository pour ajouter la configuration nécessaire à Nx Cloud.
Mergez cette PR.

## 2. Mettre à jour le workflow GitHub

Maintenant que Nx Cloud est configuré, on peut mettre à jour notre workflow GitHub Actions pour utiliser les fonctionnalités de Nx Cloud.

Pour les commandes `nx affected`, aucune modification n'est nécessaire, car Nx Cloud s'intègre automatiquement. 

Cependant, pour les commandes `nx format` et `nx synx` (et toute autre commande), il faut penser à précéder la commande par `npx nx-cloud`
pour que les résultats soient envoyés à Nx Cloud.

> [NOTE]
> Voir (https://nx.dev/docs/guides/nx-cloud/record-commands)[la documentation officielle] pour plus de détails.

```yml
- run: npx nx-cloud record -- npx nx sync:check
- run: npx nx-cloud record -- npx nx format:check
```

