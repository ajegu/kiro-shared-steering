---
inclusion: manual
---

# Justification — Design des API REST

Ce document explique le pourquoi des règles de `laravel-api-design.md`. Il s'adresse à l'équipe et n'est pas chargé automatiquement par l'agent.

## URLs et méthodes

| Règle | Justification |
|---|---|
| Préfixe `/v1` obligatoire | Permet de faire évoluer un contrat de façon incompatible sans casser les consommateurs existants. Ajouter un versioning après coup est coûteux. |
| Ressources au pluriel en kebab-case | Convention REST la plus répandue. Une règle unique évite que l'agent alterne entre `paymentOrders`, `payment_orders` et `payment-orders`. |
| Action métier = `POST` sur une sous-route verbale | Une opération comme « annuler » n'est pas une simple modification de champ : elle a ses propres règles et effets de bord. Un `PATCH status=cancelled` les cacherait. |
| Routes nommées | Les tests et les redirections ne dépendent pas des URLs, qui peuvent évoluer. |
| Table des statuts HTTP | Le choix entre 200, 201, 202 et 204 varie d'un développeur à l'autre. Une table fixe rend les API prévisibles pour les consommateurs. |

## Requêtes

| Règle | Justification |
|---|---|
| `FormRequest` dédié | Validation et autorisation regroupées, testables, et hors du contrôleur. |
| `authorize()` via Policy, jamais `true` en dur | L'agent génère souvent `return true;` par défaut, ce qui ouvre l'endpoint à tout utilisateur authentifié. C'est la faille API la plus fréquente (accès aux objets d'un autre utilisateur). |
| Méthode `toData()` | Une conversion unique et typée entre la requête validée et le DTO de l'Action. |
| Jamais `all()` ni `input()` hors du `FormRequest` | Ces méthodes renvoient aussi les champs non validés. Combiné à une assignation de masse, cela permet de modifier des champs protégés. |

## Réponses en succès

| Règle | Justification |
|---|---|
| Toujours une `JsonResource` | La forme de la réponse est découplée du schéma de base : renommer une colonne ne casse pas le contrat d'API. |
| Clés en camelCase | Cohérence avec le format d'erreur existant (`docUrl`, `requestId`). **[parti pris]** |
| Dates ISO 8601 en UTC | Format standard, sans ambiguïté de fuseau. Le client gère l'affichage local. |
| Montants en entiers avec devise | Aucune perte de précision dans la sérialisation JSON, et aucun montant ambigu sans devise. |
| Champs exposés explicitement | `parent::toArray()` expose toutes les colonnes, y compris celles ajoutées plus tard, qui peuvent être sensibles. |
| Listes toujours paginées, maximum 100 | Une liste non paginée grossit avec les données, jusqu'à dépasser la limite de 6 Mo de réponse Lambda ou le timeout. **[parti pris]** sur les valeurs 20 et 100. |

## Erreurs

| Règle | Justification |
|---|---|
| Un format unique, un tableau `errors` | Format déjà utilisé par l'équipe. Un tableau permet de renvoyer plusieurs erreurs (validation) avec la même structure qu'une erreur unique. |
| `type` et `code` issus d'enums | Les consommateurs écrivent du code sur ces valeurs : elles doivent être stables et ne jamais être inventées à la volée. |
| `docUrl` depuis la configuration | L'URL change selon l'environnement et peut évoluer ; elle ne doit pas être dupliquée dans le code. |
| `requestId` identique à celui des logs | Le support peut retrouver en quelques secondes les logs correspondant à une erreur signalée par un client. |
| Rendu centralisé dans `bootstrap/app.php` | C'est la seule garantie que toutes les erreurs, y compris celles du framework, ont la même forme. |
| Erreurs du framework converties | Sans conversion, une erreur de validation ou une 404 renvoie le format par défaut de Laravel, différent du vôtre. |
| 500 avec message générique | Un message d'exception peut contenir du SQL, des noms de tables ou des données. |

## Points en attente de décision

- La correspondance entre `type` et code HTTP est une proposition. Dans l'exemple fourni, une erreur d'autorisation porte le type `invalid_request`, ce qui laisse supposer que le `type` est une catégorie large.
- Le champ `field` des erreurs de validation est une proposition : il n'apparaît pas dans le format existant.
