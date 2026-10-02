---
inclusion: manual
---

# Justification — Revue de sécurité

Ce document explique le pourquoi de `security-review.md`. Il s'adresse à l'équipe et n'est pas chargé automatiquement par l'agent.

Ce steering est en mode `manual` : on l'appelle avec `#security-review` ou `/security-review` sur un diff avant la merge request.

| Choix | Justification |
|---|---|
| Base OWASP API Security Top 10 | Référence reconnue, centrée sur les failles spécifiques aux API, notamment les autorisations, plutôt que sur les failles web classiques. |
| Points traduits en vérifications Laravel concrètes | Une checklist générique produit des remarques vagues. Lier chaque point à un mécanisme Laravel (Policy, `validated()`, `throttle`) rend la revue vérifiable. |
| Autorisation au niveau objet en premier | C'est la faille API la plus fréquente : un utilisateur authentifié accède aux ressources d'un autre en changeant un identifiant. |
| Points ajoutés : injection, secrets, données sensibles | Hors du Top 10 API, mais essentiels en contexte financier et dans du code généré. |
| Aucune modification pendant la revue | Une revue qui corrige directement mélange constat et correction, et rend la relecture humaine difficile. Les corrections viennent ensuite, une par une. |
| Rapport avec sévérité, emplacement, scénario, correction | Permet de prioriser, de retrouver le code et de juger du risque réel. Le scénario d'exploitation évite les faux positifs théoriques. |
| Lister ce qui n'a pas pu être vérifié | Une revue silencieuse sur un point ne veut pas dire que le point est sûr. |
