---
inclusion: manual
---

# Justification — Utilisation du SDK AWS pour PHP

Ce document explique le pourquoi des règles de `aws-sdk-php.md`. Il s'adresse à l'équipe et n'est pas chargé automatiquement par l'agent.

Ce steering est en mode `auto` : l'agent le charge seulement quand la demande concerne un service AWS. Il ne coûte donc pas de contexte sur le reste du code.

## Abstractions et clients

| Règle | Justification |
|---|---|
| Abstractions Laravel d'abord | Le disque S3, la queue SQS et le cache DynamoDB sont déjà configurés par le bridge Bref, et se remplacent facilement par des fakes dans les tests. |
| Clients en singleton dans un provider | Un client construit une fois par instance réduit la latence. La région est centralisée dans la configuration. |
| Jamais de credentials explicites | Sur Lambda, le rôle IAM fournit des credentials temporaires renouvelés automatiquement. Des credentials en dur sont un risque de fuite et cassent la rotation. |
| Classe de service derrière une interface | Le reste du code ne dépend pas du SDK : les tests utilisent un fake, et la gestion d'erreurs AWS est concentrée à un seul endroit. |

## Gestion des erreurs

| Règle | Justification |
|---|---|
| Exceptions du SDK converties en exceptions métier | Les appelants n'ont pas à connaître les codes d'erreur AWS, et le rendu d'erreur API reste cohérent. |
| Loguer le code d'erreur et le request ID AWS | Le request ID est indispensable pour une demande au support AWS. Le payload peut contenir des données sensibles. |
| Réessais confiés au SDK | Le SDK gère déjà le backoff et les erreurs réessayables. Une boucle maison s'y ajoute et multiplie les tentatives. |

## Échecs partiels

| Règle | Justification |
|---|---|
| Vérifier les champs d'échec des opérations par lot | Ces opérations renvoient un succès HTTP même quand certains éléments échouent. Sans vérification, des événements ou des messages sont perdus sans aucune erreur. C'est l'un des bugs AWS les plus fréquents en production. |

## Bonnes pratiques par service

| Règle | Justification |
|---|---|
| Jamais d'ACL publique sur S3 | Les fuites de données via des buckets publics sont un incident de sécurité classique. Les URLs présignées donnent un accès limité dans le temps. |
| `getPaginator()` | Les API AWS renvoient des résultats par pages. Un seul appel ne traite que la première page, ce qui passe inaperçu sur de petits volumes de test. |
| Secrets lus depuis la configuration | Voir les contraintes Lambda : latence, coûts et throttling d'un appel par requête. |

## Tests

| Règle | Justification |
|---|---|
| Fake de service ou `MockHandler`, jamais AWS | Tests rapides, déterministes, sans coût et sans risque d'agir sur de vraies ressources. |
