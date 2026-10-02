---
inclusion: fileMatch
fileMatchPattern: ["app/**/*.php", "config/**/*.php", "bootstrap/**/*.php"]
---

# Contraintes d'exécution sur AWS Lambda (Bref)

Le code tourne sur AWS Lambda via Bref. Ce qui marche en local dans Docker peut casser sur Lambda : applique ces règles même si le code fonctionne sur ta machine.

## Système de fichiers

- Le système de fichiers est en lecture seule, sauf `/tmp`. Le dossier `storage/` est déjà redirigé vers `/tmp` par le bridge Laravel.
- `/tmp` est éphémère et limité en taille : n'y écris que des fichiers temporaires, et supprime-les après usage.
- Tout fichier qui doit persister ou être partagé va sur S3 via le disque `s3`, injecté sous forme de `Illuminate\Contracts\Filesystem\Filesystem`.
- N'utilise jamais les drivers `file` pour le cache, les sessions ou les logs.

## Configuration

- La configuration est mise en cache au démarrage de la Lambda. `env()` ne fonctionne que dans `config/*.php` ; partout ailleurs, utilise `config()`.
- Les secrets arrivent par des variables d'environnement résolues depuis SSM au démarrage. Lis-les via la config, jamais en appelant SSM ou Secrets Manager à chaque requête.

## État et cycle de vie

- L'API est sans état : pas de session, pas de cookie d'état, authentification par token à chaque requête.
- Le process PHP peut être réutilisé entre plusieurs invocations (workers de queue, handlers d'événements). Ne stocke jamais de données propres à une requête ou à un message dans une propriété statique ou dans un singleton du container.
- Pas de processus longs : ni `queue:work`, ni `schedule:work`, ni boucle d'attente, ni `sleep()`. Les queues sont consommées par des Lambdas déclenchées par SQS, et le planificateur est déclenché par EventBridge.

## Durée et taille

- Une requête HTTP doit répondre en quelques secondes ; la limite côté API Gateway est d'environ 30 secondes. Tout traitement long part dans un job en queue, et l'endpoint renvoie 202.
- La réponse d'une Lambda est limitée à 6 Mo. Pour un téléchargement volumineux, renvoie une URL S3 présignée plutôt que le fichier.
- Pour un upload volumineux, le client envoie le fichier directement sur S3 via une URL présignée ; l'API ne reçoit que la référence.
- Traite les gros volumes en flux (`lazyById()`, streams S3), jamais en chargeant tout en mémoire.

## Démarrage à froid

- Les service providers ne font aucun appel réseau et aucune requête SQL au démarrage (`register()`, `boot()`).
- Les clients coûteux à construire (SDK AWS, clients HTTP) sont enregistrés en singleton et construits à la première utilisation.

## Logs

- Les logs partent sur `stderr`, au format CloudWatch configuré par Bref. N'ajoute pas de handler de log vers un fichier.
- Le `requestId` de la requête est présent dans le contexte de chaque log.
