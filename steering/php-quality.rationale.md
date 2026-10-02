---
inclusion: manual
---

# Justification — Qualité du code PHP

Ce document explique le pourquoi des règles de `php-quality.md`. Il s'adresse à l'équipe et n'est pas chargé automatiquement par l'agent.

## Analyse statique

| Règle | Justification |
|---|---|
| Pas de nouvelle entrée dans la baseline | La baseline sert à absorber l'existant, pas à masquer les nouvelles erreurs. Sur un projet neuf, elle doit rester vide. |
| `@phpstan-ignore` avec identifiant et commentaire | Un ignore sans identifiant masque toutes les erreurs de la ligne, y compris celles qui apparaîtront plus tard. Le commentaire force à justifier le contournement en revue. |
| Génériques sur collections et relations | Sans génériques, PHPStan ne connaît pas le type des éléments, et tout le code qui les manipule échappe à l'analyse. Larastan 3 s'appuie sur ces annotations. |
| Array shapes, puis DTO au-delà de 3 clés | Une array shape documente la structure. Au-delà de quelques clés, un DTO est plus lisible, réutilisable et vérifié par PHP lui-même. |

## Valeurs nulles et comparaisons

| Règle | Justification |
|---|---|
| Exception plutôt que `null` en cas d'échec | Un `null` renvoyé se propage silencieusement jusqu'à une erreur loin de sa cause. Une exception échoue à l'endroit exact du problème. |
| Comparaisons strictes | Les comparaisons lâches de PHP réservent des surprises (`"abc" == 0` selon les versions, `in_array("1e1", ["10"])` vrai). En contexte financier, c'est inacceptable. |
| `match` exhaustif sans `default` sur un enum | Quand on ajoute une valeur à l'enum, PHPStan signale tous les `match` qui ne la traitent pas. Un `default` désactiverait cette protection. |
| Pas d'`assert()` pour valider | `assert()` est désactivé en production par configuration (`zend.assertions`). Une validation qui en dépend n'existe pas en production. |

## Données sensibles au type

| Règle | Justification |
|---|---|
| Jamais de `float` pour un montant | Les flottants ne représentent pas exactement les décimales (`0.1 + 0.2 !== 0.3`). Les erreurs d'arrondi s'accumulent sur des opérations financières. |
| Dates immuables | Modifier une date mutable reçue en paramètre change aussi la valeur chez l'appelant. C'est une source classique de bugs difficiles à tracer. |
| Ne pas convertir en `int` ce qui n'est pas un nombre | Un code postal ou une référence perd ses zéros initiaux une fois converti en entier. |

## Lisibilité

| Règle | Justification |
|---|---|
| Retours anticipés, 2 niveaux d'imbrication maximum | Le code généré a tendance à imbriquer les conditions. Les retours anticipés le rendent plus lisible et plus simple à relire en revue. |
| Méthodes courtes, une seule responsabilité | Une méthode courte se teste et se relit facilement. **[parti pris]** Le seuil de 30 lignes est indicatif. |
| Pas de magie (`__get`, `__call`, `compact`…) | Ces mécanismes échappent à PHPStan et à l'IDE. Le code devient impossible à analyser ou à refactorer avec certitude. |
| Pas de paramètre booléen qui change le comportement | À l'appel, `process($order, true)` ne dit pas ce que fait `true`. Deux méthodes nommées ou un enum rendent l'intention explicite. |
