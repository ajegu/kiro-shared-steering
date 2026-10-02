---
inclusion: fileMatch
fileMatchPattern: ["routes/api.php", "app/Http/**/*.php", "app/Exceptions/**/*.php", "bootstrap/app.php"]
---

# Design des API REST

## URLs et méthodes

- Préfixe de version obligatoire : `/v1/...`.
- Ressources au pluriel, en kebab-case : `/v1/payment-orders`, `/v1/payment-orders/{paymentOrder}`.
- Une action métier hors CRUD est un `POST` sur une sous-route verbale : `POST /v1/orders/{order}/cancel`.
- Chaque route a un nom : `->name('v1.orders.cancel')`.

| Opération | Méthode | Statut de succès |
|---|---|---|
| Lister | `GET` | 200 |
| Lire | `GET` | 200 |
| Créer | `POST` | 201 |
| Modifier | `PATCH` | 200 |
| Supprimer | `DELETE` | 204, sans corps |
| Traitement asynchrone | `POST` | 202 |

## Requêtes

- Toute donnée d'entrée est validée par un `FormRequest` dédié, jamais par `$request->validate()` dans le contrôleur.
- `authorize()` délègue à une Policy. Ne renvoie jamais `true` en dur, sauf pour un endpoint public, avec un commentaire qui le justifie.
- Le `FormRequest` expose une méthode `toData()` qui construit le DTO de l'Action.
- N'utilise jamais `$request->all()` ni `$request->input()` hors du `FormRequest` : seulement `$request->validated()` ou `toData()`.

## Réponses en succès

- Toute réponse passe par une `JsonResource` ou une collection de ressources. Ne renvoie jamais un modèle, un tableau ou `response()->json()` directement.
- Clés JSON en camelCase : `createdAt`, `walletId`.
- Dates au format ISO 8601 en UTC : `2026-10-02T16:00:00Z`.
- Montants en entiers, en unité mineure, toujours accompagnés de la devise : `{"amount": 1250, "currency": "EUR"}`.
- N'expose que les champs nécessaires au consommateur. Ajouter un champ à une ressource est un choix explicite, pas un `parent::toArray()`.
- Toute liste est paginée (taille par défaut 20, maximum 100), au format de pagination standard des ressources Laravel.

## Erreurs

Toutes les erreurs ont la même forme : un tableau `errors`, même pour une seule erreur.

```json
{
    "errors": [
        {
            "type": "invalid_request",
            "code": "authorization_error",
            "message": "You cannot read operations for this wallet.",
            "docUrl": "https://docs.example.com/errors/authorization_error",
            "requestId": "fa433c8c-1232-4c14-9f9e-95eddad19764"
        }
    ]
}
```

| Champ | Règle |
|---|---|
| `type` | Valeur de l'enum `App\Enums\ErrorType` |
| `code` | Valeur de l'enum `App\Enums\ErrorCode`, en snake_case |
| `message` | En anglais, compréhensible par le consommateur, sans détail interne |
| `docUrl` | Construite depuis la configuration (`config('api.doc_url')`), jamais en dur |
| `requestId` | L'identifiant de requête, identique à celui présent dans les logs |

### Mise en œuvre

- Toutes les erreurs sont rendues par une seule ressource, `App\Http\Resources\ErrorResource`.
- Le rendu est centralisé dans `withExceptions()` de `bootstrap/app.php`. Aucun contrôleur, middleware ou Action ne construit une réponse d'erreur.
- Une erreur métier est une exception de `app/Exceptions/` qui porte son `ErrorType`, son `ErrorCode` et son code HTTP.
- Pour un nouveau cas d'erreur, ajoute une valeur à l'enum `ErrorCode`. N'invente jamais un code sous forme de chaîne libre.
- Les exceptions du framework (validation, authentification, autorisation, modèle introuvable, route inconnue, limite de débit) sont converties vers ce même format.
- Une erreur inattendue renvoie une 500 avec un message générique. Le message réel et la trace ne vont que dans les logs.

### Correspondance type / code HTTP — PROVISOIRE, À VALIDER PAR L'ÉQUIPE

| `type` | Statut HTTP | Exemples de `code` |
|---|---|---|
| `invalid_request` | 400, 403, 404, 409, 422 selon le `code` | `validation_error`, `authorization_error`, `resource_not_found`, `conflict` |
| `authentication_error` | 401 | `missing_token`, `invalid_token` |
| `rate_limit_error` | 429 | `too_many_requests` |
| `server_error` | 500, 503 | `internal_error`, `service_unavailable` |

### Erreurs de validation — PROVISOIRE, À VALIDER PAR L'ÉQUIPE

Une entrée par champ invalide, avec un champ `field` qui contient le chemin du champ en notation pointée :

```json
{
    "errors": [
        {
            "type": "invalid_request",
            "code": "validation_error",
            "field": "lines.0.amount",
            "message": "The amount must be greater than 0.",
            "docUrl": "https://docs.example.com/errors/validation_error",
            "requestId": "fa433c8c-1232-4c14-9f9e-95eddad19764"
        }
    ]
}
```
