---
inclusion: manual
---

# Justification — Tests avec PHPUnit

Ce document explique le pourquoi des règles de `laravel-testing.md`. Il s'adresse à l'équipe et n'est pas chargé automatiquement par l'agent.

## Syntaxe

| Règle | Justification |
|---|---|
| Attributs uniquement, jamais d'annotations | Les annotations en docblock ne sont plus prises en charge depuis PHPUnit 12 : un test annoté `@test` n'est plus exécuté, sans erreur. Les modèles en génèrent encore beaucoup. |
| Classes `final`, `strict_types` | Mêmes règles que le code applicatif, vérifiées par les mêmes outils. |
| Méthodes en snake_case descriptives | Le nom du test décrit le comportement attendu et se lit dans le rapport d'échec. **[parti pris]** sur le snake_case. |
| Data providers statiques avec clés descriptives | Obligatoire en PHPUnit récent pour les providers statiques. Les clés indiquent quel cas a échoué. |
| Arrange / Act / Assert | Structure qui rend un test lisible en quelques secondes en revue. |

## Quoi tester

| Règle | Justification |
|---|---|
| Liste minimale par endpoint | Sans liste, l'agent ne teste que le cas nominal. Les failles d'autorisation (401, 403) et les erreurs de validation sont précisément ce qui passe en production sans être vu. |
| Format d'erreur vérifié par endpoint | Le format d'erreur est un contrat avec les consommateurs ; une régression doit être détectée. |
| Test de chaque Action | Les règles métier se testent plus finement au niveau de l'Action qu'à travers HTTP. |

## Assertions

| Règle | Justification |
|---|---|
| Vérifier les valeurs, pas seulement la structure | Un test qui vérifie seulement la présence d'une clé passe même si la valeur est fausse. |
| Vérifier l'état en base | Une réponse 201 ne prouve pas que les données ont été écrites correctement. |

## Isolation

| Règle | Justification |
|---|---|
| `RefreshDatabase` et factories | Chaque test part d'une base connue. Les identifiants en dur rendent les tests dépendants de l'ordre d'exécution. |
| Aucun appel réseau, `preventStrayRequests()` | Un test qui appelle un service réel est lent, instable, et peut avoir des effets réels. `preventStrayRequests()` fait échouer le test au lieu de laisser passer l'appel. |
| Jamais AWS dans un test | Coût, lenteur, et risque d'agir sur de vraies ressources. |
| Temps figé, pas de `sleep()` | Les tests liés aux dates deviennent déterministes, et `sleep()` ralentit toute la suite. |
| Tests indépendants | Condition nécessaire pour l'exécution parallèle et pour déboguer un test isolément. |

## Exécution

| Règle | Justification |
|---|---|
| Doit passer en `--parallel` | Révèle les dépendances cachées entre tests et accélère la CI. |
| Ne jamais supprimer ou affaiblir un test qui échoue | Face à un test rouge, un agent tend à modifier le test plutôt que le code. Cette règle l'oblige à signaler le changement de comportement. |
