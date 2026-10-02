---
inclusion: manual
---

# Justification — Recette de création d'endpoint

Ce document explique le pourquoi de `new-endpoint.md`. Il s'adresse à l'équipe et n'est pas chargé automatiquement par l'agent.

Ce steering est en mode `manual` : on l'appelle avec `#new-endpoint` ou `/new-endpoint` quand on crée un endpoint. Il ne coûte pas de contexte le reste du temps.

| Étape | Justification |
|---|---|
| Clarifier avant d'écrire | Les questions d'autorisation et d'erreurs métier sont celles que l'agent ignore le plus souvent. Les poser d'emblée évite de réécrire l'endpoint. |
| Proposer un plan | Un endpoint complet touche une dizaine de fichiers. Valider la liste évite un mauvais découpage. |
| Ordre d'implémentation imposé | On part des briques sans dépendances (enums, exceptions, DTO) vers celles qui les utilisent (Action, Request, contrôleur). Chaque fichier écrit ne dépend que de fichiers déjà écrits. |
| Policy avant le `FormRequest` | Garantit que `authorize()` délègue à une vraie Policy plutôt que de renvoyer `true`. |
| Tests de l'Action puis de l'endpoint | Les règles métier sont testées finement au niveau de l'Action ; le test Feature vérifie le contrat HTTP. |
| Vérifications Pint, PHPStan, tests | Mêmes contrôles que la CI : la merge request arrive verte. |
| Conclusion avec fichiers, exemple et hypothèses | Le relecteur sait quoi relire, voit le contrat en un coup d'œil, et repère les hypothèses à valider. |
