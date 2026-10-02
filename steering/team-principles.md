---
inclusion: always
---

# Principes d'équipe — API PHP / Laravel

Ces règles s'appliquent à tout code PHP que tu génères ou modifies dans ce dépôt.
Les versions sont définies dans `team-stack.md`. Les règles propres à chaque domaine sont dans les autres
fichiers de steering (design d'API, Eloquent, tests, contraintes Lambda, queues).

## Mode de travail

- Avant toute modification qui touche plus de 3 fichiers ou qui introduit une nouvelle hiérarchie de classes, propose un plan court et attends la validation.
- Garde des diffs minimaux : ne reformate pas, ne renomme pas et n'« améliore » pas du code sans rapport avec la tâche.
- N'ajoute jamais de dépendance Composer sans demander d'abord, en expliquant pourquoi la stack existante ne suffit pas.
- Ne modifie jamais une migration déjà présente sur la branche principale : crées-en une nouvelle.
- Si une exigence est ambiguë, pose la question au lieu de deviner. Si tu fais une hypothèse, énonce-la explicitement.

## Règles du langage PHP

- Chaque fichier PHP commence par `declare(strict_types=1);`.
- Le style de code est PSR-12, appliqué par Laravel Pint. Ne formate pas à la main à l'encontre de Pint.
- Type tout : propriétés, paramètres et types de retour. Évite `mixed`, et n'utilise jamais de tableau non typé là où un DTO ou une collection typée convient.
- N'utilise la PHPDoc que pour ce que PHP ne sait pas exprimer (génériques, formes de tableaux comme `array{id: int, name: string}`, `list<Order>`). Ne répète jamais les types natifs en PHPDoc.
- Les classes sont `final` par défaut. Retire `final` uniquement quand l'héritage est un besoin de conception explicite.
- Privilégie les classes et propriétés `readonly` pour les DTO, les value objects et les commandes.
- Utilise des enums natifs pour tout ensemble fermé de valeurs (statuts, types, codes d'erreur). N'utilise jamais de constantes string ou int pour cela.
- Utilise la promotion des propriétés dans le constructeur et les arguments nommés quand ils améliorent la lisibilité.

## Nommage

- Tous les identifiants, commentaires, messages de commit, messages de log et messages d'API sont en anglais.
- Classes : noms au singulier (`Order`, `CreateOrderAction`, `OrderResource`). Méthodes : verbes (`create`, `cancel`, `findByReference`).
- Les booléens se lisent comme des questions : `isPaid`, `hasExpired`, `canBeRefunded`.
- Pas d'abréviations, sauf celles très répandues (`id`, `url`, `dto`, `api`).

## Dépendances et usage du framework

- Injecte les dépendances par le constructeur dans le code applicatif (contrôleurs, actions, services, jobs).
- Les façades et les helpers globaux sont acceptés uniquement dans les routes, la config, les service providers et les tests.
- Lis la configuration via `config()` ou des valeurs de config injectées. `env()` n'est autorisé que dans `config/*.php`.

## Erreurs et exceptions

- Les échecs métier s'expriment en levant des exceptions métier typées, jamais en renvoyant `false`, `null` ou des tableaux d'erreur.
- Ne construis jamais une réponse d'erreur à la main dans un contrôleur. Le rendu des erreurs est centralisé (voir `laravel-api-design.md`).
- N'attrape jamais une exception pour l'ignorer. Si tu l'attrapes, traite-la réellement, ou relance une exception plus spécifique avec l'originale en `previous`.

## Logs et protection des données

- Utilise des logs structurés : un message court et constant, plus un tableau de contexte (`$this->logger->info('Order paid', ['order_id' => $order->id])`, avec `Psr\Log\LoggerInterface` injecté).
- Ne logue jamais de secrets, tokens, mots de passe, numéros de carte ou d'IBAN complets, ni de données personnelles au-delà des identifiants techniques.
- N'expose jamais de détails internes (SQL, noms de classes, stack traces) dans les réponses d'API.

## Interdits

- Secrets, identifiants ou URLs propres à un environnement codés en dur.
- `dd()`, `dump()`, `var_dump()`, `print_r()`, `ray()`, `die()`, `exit()` dans le code commité.
- L'opérateur de suppression d'erreur `@`.
- Du SQL brut construit par concaténation ou interpolation de chaînes. Utilise le query builder ou des paramètres liés.
- Du code commenté, et des `TODO` sans référence de ticket associée.

## Définition de « terminé »

Le code que tu livres n'est terminé que lorsque :

1. Les tests couvrant la modification sont écrits en même temps que le code (voir `laravel-testing.md`).
2. `vendor/bin/pint --test` passe.
3. `vendor/bin/phpstan analyse` passe au niveau configuré, sans nouvelle entrée dans la baseline.
4. `php artisan test` passe.

Si tu ne peux pas exécuter ces commandes, dis-le explicitement et liste ce qui reste à vérifier.
