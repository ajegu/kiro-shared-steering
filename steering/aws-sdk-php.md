---
inclusion: auto
name: aws-sdk-php
description: Utilisation du SDK AWS pour PHP dans le code Laravel (S3, SQS, DynamoDB, EventBridge, SSM, Secrets Manager). À charger quand le code appelle un service AWS ou manipule des fichiers, messages ou événements AWS.
---

# Utilisation du SDK AWS pour PHP

## Abstractions Laravel d'abord

- Fichiers : disque `s3` de Laravel. Messages applicatifs : queue `sqs` de Laravel. Cache : store `dynamodb`.
- N'utilise le SDK directement que pour ce que Laravel ne couvre pas : URLs présignées avancées, EventBridge, opérations DynamoDB métier, etc.

## Clients

- Les clients du SDK (`S3Client`, `EventBridgeClient`…) sont enregistrés en singleton dans un service provider, avec la région lue depuis la configuration.
- Ne fais jamais de `new S3Client(...)` dans une Action, un job ou un contrôleur.
- Ne passe jamais de credentials explicites : sur Lambda, les droits viennent du rôle IAM de la fonction ; en local, de l'environnement.
- Chaque usage passe par une classe de `app/Services/Aws/` qui implémente une interface métier (`EventPublisher`, `DocumentStorage`). Le reste du code ne dépend que de l'interface.

## Gestion des erreurs

- Attrape les exceptions spécifiques du SDK (`S3Exception`, `EventBridgeException`, ou `AwsException` à défaut) dans la classe de service, et convertis-les en exceptions métier.
- Logue `getAwsErrorCode()` et `getAwsRequestId()` avec le contexte métier, jamais le payload complet.
- Les réessais sont gérés par le SDK : configure le mode `standard` dans le client et n'écris pas de boucle de réessai maison.

## Échecs partiels des opérations par lot

Ces appels réussissent au niveau HTTP même quand une partie des éléments échoue. Vérifie toujours le résultat :

| Opération | Champ à vérifier |
|---|---|
| EventBridge `putEvents` | `FailedEntryCount` et `Entries[*].ErrorCode` |
| SQS `sendMessageBatch` / `deleteMessageBatch` | `Failed` |
| DynamoDB `batchWriteItem` | `UnprocessedItems` |
| DynamoDB `batchGetItem` | `UnprocessedKeys` |

## Bonnes pratiques par service

- S3 : jamais d'ACL publique. Les échanges de fichiers avec les clients passent par des URLs présignées à durée courte.
- Résultats paginés : utilise `getPaginator()`, jamais un seul appel en supposant que tout est revenu.
- Secrets : lus depuis la configuration (résolus au démarrage par Bref depuis SSM). Si un secret doit vraiment être lu à l'exécution, mets-le en cache.

## Tests

- Dans les tests, remplace la classe de service par un fake dans le container. Pour tester la classe de service elle-même, utilise `Aws\MockHandler` avec des `Aws\Result`. Un test n'appelle jamais AWS.
