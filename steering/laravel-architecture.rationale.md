---
inclusion: manual
---

# Justification — Architecture applicative Laravel

Ce document explique le pourquoi des règles de `laravel-architecture.md`. Il s'adresse à l'équipe et n'est pas chargé automatiquement par l'agent.

| Règle | Justification |
|---|---|
| Arborescence imposée, pas de nouveau dossier sans proposition | Sur 15 à 30 applications, une arborescence identique permet à chacun de s'y retrouver. Sans cadre, l'agent invente des dossiers (`Helpers`, `Managers`, `Utils`) différents d'un projet à l'autre. |
| Flux Request → Controller → Action → Resource | Chaque couche a une seule responsabilité : validation, orchestration HTTP, logique métier, présentation. On sait où chercher et où ajouter du code. |
| Contrôleurs sans logique ni requête Eloquent | La logique métier dans un contrôleur n'est réutilisable ni dans un job ni dans une commande, et se teste uniquement en passant par HTTP. |
| Méthodes de ressource standard, contrôleur invocable hors CRUD | Évite les contrôleurs fourre-tout avec des dizaines de méthodes. Une action métier a un nom explicite et un fichier dédié. **[parti pris]** |
| Une Action = un cas d'usage, une méthode `handle()` | Le nom de la classe décrit ce que fait l'application. Une convention de méthode unique rend tous les appels prévisibles. **[parti pris]** sur le nom `handle()`, choisi par cohérence avec les jobs Laravel. |
| Une Action ignore HTTP | C'est ce qui la rend réutilisable depuis un contrôleur, un job, un listener ou une commande, et testable sans requête HTTP. |
| Transactions via `ConnectionInterface` injectée | Cohérent avec la règle d'injection de dépendances de `team-principles.md`, et explicite sur le fait que l'Action écrit en base. |
| DTO `final readonly` suffixés `Data` | Un DTO typé remplace les tableaux associatifs entre couches : PHPStan vérifie chaque propriété, et l'objet ne peut pas être modifié en route. |
| Services externes derrière une interface | Les Actions deviennent testables en remplaçant l'interface par un fake, sans réseau. Changer de fournisseur ne touche qu'une classe. |
| Effets de bord via événements en queue | L'Action reste centrée sur son cas d'usage. Un échec de notification ne fait pas échouer un paiement, et chaque effet de bord peut être réessayé séparément. |
| `ShouldDispatchAfterCommit` | Sans ce marqueur, un listener peut traiter un événement dont la transaction a ensuite été annulée, et agir sur des données qui n'existent pas. |
| Propriétés plutôt qu'attributs Laravel 13 | **[parti pris]** Les deux sont valides. Mélanger les styles dans un même code est ce qu'on veut éviter. |
