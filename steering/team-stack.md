---
inclusion: always
---

# Stack technique de référence

Ce fichier est la seule source de vérité pour les versions. Les autres steerings n'en mentionnent aucune.
Avant d'utiliser une API d'un package, vérifie la version réellement installée dans `composer.json` / `composer.lock`.

| Brique | Version | Remarque |
|---|---|---|
| PHP | 8.4 | Même version dans l'image Docker locale et dans le runtime Bref (`php-84`) |
| Laravel | 13 | Configuration de l'application dans `bootstrap/app.php` |
| Bref | 3 | Avec `bref/laravel-bridge` 3.x |
| Tests | PHPUnit (version du `composer.json`, 12 minimum) | Attributs PHP uniquement, voir `laravel-testing.md` |
| Analyse statique | PHPStan 2 + Larastan 3 | Niveau `max` |
| Style | Laravel Pint | Configuration dans `pint.json` |
| AWS | AWS SDK for PHP v3 | Voir `aws-sdk-php.md` |

## PHP 8.4

- Tu peux utiliser les nouveautés de PHP 8.4 quand elles simplifient le code : visibilité asymétrique (`public private(set)`), property hooks, `new` sans parenthèses en chaînage, `array_find()`, `array_any()`, `array_all()`.
- N'utilise aucune fonctionnalité de PHP 8.5 : opérateur pipe `|>`, `array_first()`, `array_last()`, attribut `#[\NoDiscard]`, extension `Uri`. Elles cassent à l'exécution sur nos images.

## Laravel 13

- La structure est celle de Laravel 11 et suivants : il n'y a ni `app/Http/Kernel.php` ni `app/Console/Kernel.php`. Middlewares, exceptions et routing se configurent dans `bootstrap/app.php`, et les tâches planifiées dans `routes/console.php`.
- N'utilise pas de pattern déprécié ou retiré d'une version antérieure. En cas de doute sur une API, privilégie l'approche documentée pour Laravel 13 et signale ton incertitude.
- Les attributs PHP de configuration introduits par Laravel 13 ne sont pas utilisés : on garde les propriétés de classe (voir `laravel-architecture.md`).
