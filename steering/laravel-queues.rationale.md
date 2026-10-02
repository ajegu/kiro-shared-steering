---
inclusion: manual
---

# Justification — Jobs et queues

Ce document explique le pourquoi des règles de `laravel-queues.md`. Il s'adresse à l'équipe et n'est pas chargé automatiquement par l'agent.

## Conception d'un job

| Règle | Justification |
|---|---|
| Job fin qui délègue à une Action | La logique métier reste testable et réutilisable hors queue. Le job ne gère que l'aspect asynchrone. |
| Seulement des identifiants dans le constructeur | Un modèle sérialisé peut être obsolète au moment de l'exécution, plusieurs minutes plus tard. Recharger garantit des données fraîches. |
| Payload petit | SQS refuse les messages de plus de 256 Ko. Le dispatch échoue alors en production avec des données réelles, alors qu'il passait avec les données de test. |

## Idempotence

| Règle | Justification |
|---|---|
| Un job peut s'exécuter deux fois | SQS garantit une livraison « au moins une fois ». Un timeout ou un redémarrage peut faire rejouer un message déjà traité. |
| Vérifier l'état avant d'agir | La façon la plus simple de rendre un job idempotent. |
| Verrou ou contrainte d'unicité sur les opérations irréversibles | Un double virement ou une double notification client ne se rattrape pas. La garantie doit venir de la base, pas d'un simple test applicatif. |

## Réessais et échecs

| Règle | Justification |
|---|---|
| `$tries` et `$backoff` progressif | Sans backoff, les réessais frappent immédiatement un service déjà en difficulté. |
| Timeouts emboîtés : job < Lambda < visibility timeout | Si le message redevient visible avant la fin du traitement, une seconde Lambda le traite en parallèle. Si la Lambda s'arrête avant le job, l'échec n'est pas géré proprement. |
| `failed()` implémenté | Sans lui, un job en échec définitif disparaît sans trace ni remise en état de l'objet métier. |
| Laisser remonter les exceptions, `fail()` si inutile de réessayer | Une exception avalée est vue comme un succès : le message est supprimé. À l'inverse, réessayer une donnée invalide ne fait que retarder l'échec. |
| `release()` plutôt que `sleep()` | `sleep()` consomme du temps Lambda facturé et rapproche du timeout. `release()` remet le message en file avec un délai. |

## Dispatch

| Règle | Justification |
|---|---|
| Dispatch après commit | Sans cela, le job peut s'exécuter avant la fin de la transaction, et ne pas trouver les données, ou traiter des données annulées ensuite. |
| `ShouldBeUnique` seulement avec des verrous atomiques | Avec un store de cache sans verrou atomique, l'unicité n'est pas garantie, sans aucune erreur visible. |
| Listeners en queue soumis aux mêmes règles | Ce sont des jobs, avec les mêmes risques. |
