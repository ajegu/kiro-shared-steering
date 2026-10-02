---
inclusion: fileMatch
fileMatchPattern: "**/*.php"
---

# Qualité du code PHP

Complète `team-principles.md` avec les règles qui permettent de passer PHPStan au niveau `max` sans contournement.

## Analyse statique

- N'ajoute jamais d'entrée dans la baseline PHPStan.
- N'utilise `@phpstan-ignore` qu'en dernier recours, toujours avec l'identifiant de l'erreur et un commentaire qui explique pourquoi : `// @phpstan-ignore argument.type (le SDK AWS déclare un type trop large)`. Jamais `@phpstan-ignore-line` ni `@phpstan-ignore-next-line` sans identifiant.
- Type les collections avec leurs génériques : `@return Collection<int, Order>`, `@param list<OrderLine> $lines`.
- Type les relations Eloquent avec leurs génériques : `@return HasMany<OrderLine, $this>`.
- Décris les tableaux structurés avec des array shapes. Si une array shape dépasse 3 clés ou circule entre plusieurs classes, remplace-la par un DTO.

## Valeurs nulles et comparaisons

- Ne renvoie pas `null` pour signaler un échec : lève une exception. Réserve les types nullables aux valeurs réellement optionnelles.
- Comparaisons strictes uniquement : `===`, `!==`, `in_array($value, $list, true)`, `array_search($value, $list, true)`.
- Utilise `match` plutôt que `switch`. Sur un enum, couvre tous les cas sans branche `default`, pour que PHPStan signale un cas oublié.
- N'utilise pas `assert()` pour valider des données à l'exécution.

## Données sensibles au type

- Montants : jamais de `float`. Utilise des entiers en unité mineure (centimes) ou un value object `Money`, toujours accompagnés de la devise.
- Dates : utilise des dates immuables (`CarbonImmutable`, `DateTimeImmutable`). Ne modifie jamais une date reçue en paramètre.
- Identifiants et codes : ne convertis pas en `int` une valeur qui n'est pas un nombre (IBAN, référence, code postal) ; garde-la en `string`.

## Lisibilité

- Retours anticipés plutôt que conditions imbriquées. Pas plus de 2 niveaux d'imbrication dans une méthode.
- Une méthode fait une seule chose. Au-delà d'environ 30 lignes, découpe-la.
- Pas de magie dans notre code : ni `__get`, `__set`, `__call` dans nos classes, ni propriétés dynamiques, ni `compact()` ou `extract()`.
- Pas de paramètres booléens qui changent le comportement d'une méthode (`process($order, true)`) : crée deux méthodes, ou utilise un enum.
