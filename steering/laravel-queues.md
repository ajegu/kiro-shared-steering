---
inclusion: fileMatch
fileMatchPattern: ["app/Jobs/**/*.php", "app/Listeners/**/*.php", "config/queue.php"]
---

# Jobs et queues (SQS sur Lambda)

Les jobs sont stockés dans SQS et exécutés par une Lambda via le `QueueHandler` du bridge Bref. Le comportement des Laravel Queues s'applique (`$tries`, `$backoff`, `failed()`).

## Conception d'un job

- Un job est une coquille fine : il récupère ses données et délègue le travail à une Action. Pas de logique métier dans `handle()`.
- Le constructeur ne reçoit que des identifiants et des scalaires (`string $orderId`), jamais un modèle complet ou un objet volumineux. Le job recharge les données fraîches au début de `handle()`.
- Le payload sérialisé reste petit : la limite d'un message SQS est de 256 Ko.

## Idempotence

- SQS livre chaque message au moins une fois : un job peut s'exécuter deux fois. Le résultat doit être identique.
- Vérifie l'état avant d'agir (« déjà payé ? alors ne rien faire »), et utilise des clés d'idempotence pour les appels externes qui le permettent.
- Une opération qui ne peut pas être rejouée sans risque (envoi d'argent, notification client) est protégée par un verrou ou une contrainte d'unicité en base.

## Réessais et échecs

- Déclare toujours `$tries` et `$backoff` (tableau progressif, par exemple `[10, 60, 300]`).
- Déclare `$timeout`, strictement inférieur au timeout de la Lambda. Le timeout de la Lambda doit lui-même rester inférieur au visibility timeout de la queue SQS.
- Implémente `failed(Throwable $exception)` : log structuré avec l'identifiant concerné, et remise de l'objet métier dans un état cohérent si nécessaire.
- Laisse remonter les exceptions pour déclencher un réessai. N'attrape une exception que pour la transformer, ou pour échouer définitivement avec `$this->fail()` quand un réessai ne changera rien (donnée invalide).
- Pour différer, utilise `$this->release($secondes)`, jamais `sleep()`. Le délai maximal de SQS est de 15 minutes.

## Dispatch

- Un job dispatché pendant une transaction ne doit partir qu'après le commit : `->afterCommit()`, ou `after_commit` activé dans la configuration de la connexion.
- N'utilise `ShouldBeUnique` que si le store de cache configuré supporte les verrous atomiques (DynamoDB ou Redis).
- Listeners en queue : implémentent `ShouldQueue` et suivent les mêmes règles que les jobs.
