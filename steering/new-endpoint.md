---
inclusion: manual
---

# Recette : créer un endpoint d'API complet

Suis ces étapes dans l'ordre. Respecte `laravel-architecture.md`, `laravel-api-design.md` et `laravel-testing.md`.

## 1. Clarifier

Avant d'écrire du code, vérifie que tu connais :

- la ressource, la méthode HTTP et l'URL ;
- qui a le droit d'appeler l'endpoint (rôle, propriétaire de la ressource) ;
- les champs d'entrée, leurs règles de validation et les champs de sortie ;
- les erreurs métier possibles et leur `code`.

S'il manque une information, pose la question. Sinon, énonce tes hypothèses.

## 2. Proposer un plan

Liste les fichiers que tu vas créer ou modifier, et attends la validation.

## 3. Implémenter, dans cet ordre

1. Les valeurs à ajouter aux enums (`ErrorCode`, statuts…).
2. Les exceptions métier dans `app/Exceptions/`.
3. Le DTO dans `app/Data/`.
4. L'Action dans `app/Actions/`.
5. La Policy, ou la méthode de Policy.
6. Le `FormRequest`, avec `rules()`, `authorize()` et `toData()`.
7. La `JsonResource`.
8. Le contrôleur (méthode de ressource, ou contrôleur invocable pour une action hors CRUD).
9. La route nommée dans `routes/api.php`, sous le préfixe de version.

## 4. Tester

1. Test de l'Action : règles métier et chaque exception.
2. Test Feature de l'endpoint : cas nominal, validation (data provider), 401, 403, 404, format d'erreur.

## 5. Vérifier

Lance `vendor/bin/pint --test`, `vendor/bin/phpstan analyse` et `php artisan test`.

## 6. Conclure

Termine par :

- la liste des fichiers créés ou modifiés ;
- un exemple de requête et de réponse ;
- les hypothèses faites et ce qui n'a pas pu être vérifié.
