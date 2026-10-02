---
inclusion: manual
---

# Revue de sécurité d'un changement

Analyse le diff ou les fichiers indiqués avec la checklist ci-dessous, inspirée de l'OWASP API Security Top 10.
Ne modifie aucun fichier pendant la revue : produis uniquement le rapport.

## Checklist

| # | Point de contrôle | Ce qu'il faut vérifier dans le code |
|---|---|---|
| 1 | Autorisation au niveau objet | Chaque accès à une ressource par identifiant vérifie que l'appelant y a droit (Policy appelée avec le modèle), pas seulement qu'il est authentifié |
| 2 | Authentification | Toutes les routes non publiques sont sous le middleware d'authentification ; aucune route sensible n'en est exclue |
| 3 | Autorisation au niveau propriété | Pas de `$guarded = []`, uniquement `validated()` ou `toData()`. Les `JsonResource` n'exposent aucun champ interne ou sensible |
| 4 | Consommation de ressources | Listes paginées avec un maximum, limite de débit (`throttle`) sur les routes coûteuses, tailles d'entrée bornées dans la validation |
| 5 | Autorisation au niveau fonction | Les routes d'administration ou internes ont un contrôle de rôle explicite |
| 6 | Flux métier sensibles | Opérations financières protégées contre le rejeu (idempotence) et les accès concurrents (verrou, transaction) |
| 7 | SSRF | Aucune URL fournie par l'utilisateur n'est appelée sans liste blanche de domaines |
| 8 | Configuration | Pas de mode debug, de CORS ouvert ou de route de diagnostic exposée |
| 9 | Inventaire | Pas de route non versionnée ou oubliée ; les routes dépréciées sont identifiées |
| 10 | Consommation d'API tierces | Les réponses des services externes sont validées avant usage, avec des timeouts définis |
| 11 | Injection | Pas de SQL ni de commande système construits par concaténation |
| 12 | Secrets et données | Pas de secret en dur. Pas de donnée sensible (token, IBAN, carte, donnée personnelle) dans les logs ni dans les messages d'erreur |

## Format du rapport

Pour chaque problème trouvé :

- **Sévérité** : critique, haute, moyenne ou basse ;
- **Emplacement** : fichier et ligne ;
- **Problème** : ce qui ne va pas et le scénario d'exploitation ;
- **Correction** : la modification recommandée.

Termine par la liste des points de la checklist vérifiés sans problème, et ceux que tu n'as pas pu vérifier faute d'accès au code concerné.
