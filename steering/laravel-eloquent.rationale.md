---
inclusion: manual
---

# Justification — Eloquent et base de données

Ce document explique le pourquoi des règles de `laravel-eloquent.md`. Il s'adresse à l'équipe et n'est pas chargé automatiquement par l'agent.

## Modèles

| Règle | Justification |
|---|---|
| `$fillable` explicite, jamais `$guarded = []` | `$guarded = []` rend toutes les colonnes assignables en masse. Un champ ajouté plus tard (rôle, solde, statut) devient modifiable depuis une requête. |
| Casts dans `casts()` | Les enums, dates et booléens sont typés dès la lecture en base, et PHPStan connaît leur type. |
| Relations typées avec génériques | Larastan peut vérifier tout le code qui parcourt les relations. |
| Modèle limité à l'état et aux relations | Un modèle qui contient la logique métier devient un « god object » difficile à tester. Les méthodes de lecture d'état restent dans le modèle, car elles en sont le vocabulaire naturel. |
| UUID exposés, jamais l'auto-incrément | Un identifiant séquentiel révèle le volume d'activité et facilite l'énumération des ressources des autres clients. **[parti pris]** |

## Requêtes

| Règle | Justification |
|---|---|
| Pas de requête dans une boucle, `with()` et `withCount()` | Le problème N+1 est le défaut de performance le plus courant dans le code Eloquent généré. Sur Lambda, chaque requête SQL en plus allonge la durée facturée. |
| Lazy loading interdit hors production | Le N+1 devient une erreur visible dans les tests au lieu d'une lenteur découverte en production. |
| Sélection des colonnes sur les gros volumes | Moins de données transférées et moins de mémoire, qui est limitée sur Lambda. |
| `lazyById()` / `chunkById()` | `all()` charge toute la table en mémoire. `chunkById()` reste correct même si les données changent pendant le parcours, contrairement à `chunk()`. |
| Requêtes complexes dans des scopes | Une requête nommée se lit, se réutilise et se teste. |
| SQL brut seulement avec paramètres liés | Prévention de l'injection SQL. |
| `lockForUpdate()` sur lecture puis écriture concurrente | Sans verrou, deux requêtes simultanées lisent le même solde et l'écrasent l'une l'autre. C'est critique sur des données financières. |

## Migrations

| Règle | Justification |
|---|---|
| Classes anonymes avec `down()` | Les classes anonymes évitent les conflits de noms. `down()` permet un rollback rapide en cas de problème au déploiement. |
| Jamais modifier une migration mergée | Elle a déjà été jouée ailleurs et ne le sera plus : la modification crée un écart silencieux entre environnements. |
| Clés étrangères contraintes, `onDelete` explicite | La base garantit l'intégrité. Le comportement à la suppression est une décision métier, pas un défaut implicite. |
| Index sur les filtres et tris fréquents | Sans index, la requête parcourt toute la table : acceptable en développement, catastrophique en production. |
| Changements sans interruption | Pendant un déploiement, l'ancien et le nouveau code tournent en même temps sur la même base. Une colonne renommée ou obligatoire casse l'ancien code. |
| Montants en `bigInteger`, devise en `char(3)` | Même raison que l'interdiction des `float` : aucune perte de précision. `char(3)` correspond aux codes ISO 4217. |

## Factories

| Règle | Justification |
|---|---|
| Une factory par modèle, avec des states métier | Les tests restent lisibles (`Order::factory()->paid()`) et ne dupliquent pas la construction des données. |
| Données valides par défaut | Un test ne doit préciser que ce qui compte pour lui. |
