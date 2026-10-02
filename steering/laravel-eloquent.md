---
inclusion: fileMatch
fileMatchPattern: ["app/Models/**/*.php", "database/**/*.php"]
---

# Eloquent et base de données

## Modèles

- Déclare explicitement `$fillable`. N'utilise jamais `$guarded = []`.
- Déclare les casts dans la méthode `casts()` : enums, dates immuables (`immutable_datetime`), booléens, JSON.
- Type chaque relation avec son type de retour et ses génériques : `public function lines(): HasMany` avec `@return HasMany<OrderLine, $this>`.
- Un modèle ne contient que : relations, casts, accessors/mutators simples, scopes locaux et méthodes de lecture d'état (`isPaid()`, `canBeCancelled()`). La logique métier va dans les Actions.
- Les identifiants exposés dans l'API sont des UUID (trait `HasUuids` sur une colonne dédiée). Ne jamais exposer l'identifiant auto-incrémenté.

## Requêtes

- Pas de requête dans une boucle. Charge les relations à l'avance avec `with()`, et les comptes avec `withCount()`.
- Le lazy loading est interdit hors production (`Model::preventLazyLoading()` est activé). Si un test échoue pour lazy loading, corrige le chargement, ne désactive pas la protection.
- Sélectionne les colonnes nécessaires sur les requêtes volumineuses.
- Pour parcourir un grand volume, utilise `lazyById()` ou `chunkById()`, jamais `all()` ou `get()` sans limite.
- Une requête qui dépasse quelques conditions va dans un scope nommé ou une méthode dédiée, pas inline dans une Action.
- SQL brut uniquement via `DB::raw()`, `whereRaw()` ou `selectRaw()` avec des paramètres liés, et seulement si le query builder ne suffit pas.
- Utilise `lockForUpdate()` dans une transaction quand une lecture précède une écriture concurrente (solde, compteur, statut).

## Migrations

- Classes anonymes, avec une méthode `down()` qui annule réellement `up()`.
- Ne modifie jamais une migration déjà mergée : crée-en une nouvelle.
- Clés étrangères déclarées avec `foreignId()->constrained()` (ou `foreignUuid()`), avec un comportement `onDelete` explicite.
- Index sur chaque colonne utilisée en filtre ou en tri fréquent.
- Changements sans interruption de service : une nouvelle colonne est d'abord ajoutée nullable ou avec une valeur par défaut. Ne renomme ni ne supprime une colonne dans la même release que le code qui cesse de l'utiliser.
- Montants en `bigInteger` (unité mineure), devise en `char(3)`. Jamais de `float` ni de `double`.

## Factories

- Chaque modèle a une factory, avec des states nommés pour les cas métier (`->paid()`, `->cancelled()`).
- Les factories produisent des données valides par défaut.
