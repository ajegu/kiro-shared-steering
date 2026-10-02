---
inclusion: manual
---

# Justification — Contraintes d'exécution sur AWS Lambda

Ce document explique le pourquoi des règles de `laravel-lambda-constraints.md`. Il s'adresse à l'équipe et n'est pas chargé automatiquement par l'agent.

La plupart de ces règles existent parce que l'environnement local (Docker, serveur classique) ne reproduit pas les contraintes de Lambda. Le code généré fonctionne alors en local et échoue en production.

## Système de fichiers

| Règle | Justification |
|---|---|
| Lecture seule sauf `/tmp` | Contrainte de Lambda. Le bridge Bref redirige `storage/` vers `/tmp`, mais un chemin écrit en dur ailleurs échoue. |
| `/tmp` éphémère, nettoyé après usage | `/tmp` disparaît quand l'instance est recyclée, et il est partagé entre les invocations d'une même instance : des fichiers oubliés finissent par le remplir. |
| Fichiers persistants sur S3 | Seul stockage partagé entre toutes les instances de la Lambda. |
| Jamais de driver `file` | Cache, sessions ou logs en fichier sont perdus et propres à une instance : un utilisateur ne retrouve pas son cache sur l'instance suivante. |

## Configuration

| Règle | Justification |
|---|---|
| `env()` uniquement dans `config/` | Bref met la configuration en cache au démarrage. Ensuite, `env()` renvoie `null` hors des fichiers de config. |
| Secrets résolus au démarrage | Appeler SSM ou Secrets Manager à chaque requête ajoute de la latence, des coûts et un risque de throttling. |

## État et cycle de vie

| Règle | Justification |
|---|---|
| API sans état | Plusieurs instances traitent les requêtes en parallèle ; aucune ne peut supposer avoir vu la requête précédente. |
| Pas d'état de requête en statique ou en singleton | Un process réutilisé garde ses propriétés statiques et ses singletons. Les données d'un message peuvent alors fuiter dans le traitement du suivant, voire entre clients différents. |
| Pas de processus longs | Une Lambda s'arrête à la fin de son timeout. Les consommateurs de queue et le planificateur sont pris en charge par SQS et EventBridge. |

## Durée et taille

| Règle | Justification |
|---|---|
| Traitement long en queue, réponse 202 | API Gateway coupe la requête après environ 30 secondes, même si la Lambda continue. Le client reçoit une erreur alors que le traitement se poursuit. |
| Réponse limitée à 6 Mo | Limite de Lambda pour une invocation synchrone. Au-delà, la réponse échoue. |
| Uploads via URL présignée | Évite les limites de taille d'API Gateway et de Lambda, et ne facture pas de temps de calcul pour un simple transfert. |
| Traitement en flux | La mémoire d'une Lambda est fixée à la configuration ; un gros fichier chargé en entier la dépasse. |

## Démarrage à froid

| Règle | Justification |
|---|---|
| Rien de réseau ni de SQL dans les providers | Le démarrage à froid s'ajoute au temps de réponse de la première requête. Un appel réseau au boot ralentit aussi toutes les commandes et tous les tests. |
| Clients coûteux en singleton paresseux | Construits une seule fois par instance, et uniquement si la requête en a besoin. |

## Logs

| Règle | Justification |
|---|---|
| `stderr` au format CloudWatch | Seule sortie collectée par Lambda. Un log écrit dans un fichier n'est jamais lu. |
| `requestId` dans le contexte | Permet de relier l'erreur renvoyée au client aux logs de la requête. |
