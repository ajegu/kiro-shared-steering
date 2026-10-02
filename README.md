# kiro-shared-steering

Steerings Kiro partagés par l'équipe pour la génération de code PHP : API Laravel exécutées sur AWS Lambda avec Bref.

## Installation dans un projet

Copier le contenu de `steering/` dans le dossier `.kiro/steering/` du projet.
Les fichiers `*.rationale.md` peuvent être copiés aussi : ils sont en `inclusion: manual` et ne sont jamais chargés automatiquement.

Un projet peut surcharger un fichier : la version du workspace l'emporte sur la version globale.

## Contenu

| Fichier | Inclusion | Rôle |
|---|---|---|
| `team-principles.md` | always | Règles communes à tout le code PHP |
| `team-stack.md` | always | Versions de référence de la stack |
| `php-quality.md` | fileMatch `**/*.php` | Typage, PHPStan niveau max, lisibilité |
| `laravel-architecture.md` | fileMatch `app/**/*.php` | Organisation en couches : Request, Controller, Action, Resource |
| `laravel-api-design.md` | fileMatch routes, `app/Http`, exceptions | Conventions REST et format d'erreur |
| `laravel-eloquent.md` | fileMatch modèles et `database/` | Modèles, requêtes, migrations, factories |
| `laravel-testing.md` | fileMatch `tests/**` | Tests PHPUnit |
| `laravel-lambda-constraints.md` | fileMatch `app/`, `config/`, `bootstrap/` | Contraintes d'exécution sur Lambda |
| `laravel-queues.md` | fileMatch jobs et listeners | Jobs SQS : idempotence, réessais, timeouts |
| `aws-sdk-php.md` | auto | Utilisation du SDK AWS pour PHP |
| `new-endpoint.md` | manual (`#new-endpoint`) | Recette de création d'un endpoint complet |
| `security-review.md` | manual (`#security-review`) | Revue de sécurité d'un diff |

Chaque steering a un fichier `<nom>.rationale.md` qui justifie ses règles.

## Contribuer

Toute modification passe par une pull request. Une nouvelle règle s'accompagne de sa justification dans le fichier `.rationale.md` correspondant.
