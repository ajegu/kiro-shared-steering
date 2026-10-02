---
inclusion: manual
---

# Justification — Stack technique de référence

Ce document explique le pourquoi des règles de `team-stack.md`. Il s'adresse à l'équipe et n'est pas chargé automatiquement par l'agent.

| Règle | Justification |
|---|---|
| Un seul fichier pour toutes les versions | Une montée de version ne modifie qu'un fichier. Des versions dispersées dans plusieurs steerings finissent par se contredire, et l'agent applique alors la mauvaise. |
| Vérifier `composer.lock` avant d'utiliser une API | Les modèles mélangent les API de plusieurs versions d'un même package. La version réellement installée est la seule référence fiable. |
| PHP 8.4, identique en local et dans le runtime Bref | Nos images Docker ne proposent pas encore PHP 8.5. Un écart de version entre local et Lambda produit du code qui passe les tests et casse en production. |
| Nouveautés 8.4 autorisées | Visibilité asymétrique, property hooks et nouvelles fonctions de tableau simplifient le code. Les citer explicitement aide l'agent, qui a vu peu d'exemples de ces fonctionnalités récentes. |
| Fonctionnalités 8.5 listées comme interdites | Les modèles récents connaissent PHP 8.5 et peuvent utiliser l'opérateur pipe ou `array_first()`. Ces appels ne lèvent une erreur qu'à l'exécution, d'où la liste explicite. |
| Structure Laravel 11+ (pas de `Kernel.php`) | Une grande partie du code Laravel vu à l'entraînement date d'avant Laravel 11. Sans cette règle, l'agent recrée des fichiers `Kernel.php` qui ne sont plus chargés. |
| Pas de pattern déprécié, signaler l'incertitude | L'agent ne sait pas toujours quand une API a changé. Lui demander de signaler son doute permet de vérifier en revue plutôt qu'en production. |
| Propriétés de classe plutôt qu'attributs Laravel 13 | **[parti pris]** Les deux styles fonctionnent. Les propriétés sont le style le plus documenté et le plus présent dans le code existant. L'essentiel est de ne pas mélanger les deux. |
