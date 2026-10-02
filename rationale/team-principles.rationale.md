---
inclusion: manual
---

# Justification — Principes d'équipe

Ce document explique le pourquoi de chaque règle de `team-principles.md`.
Il s'adresse à l'équipe (revues de code, onboarding, discussions sur l'évolution des règles), pas à l'agent :
il est en `inclusion: manual` pour ne pas consommer de contexte à chaque requête. On peut le charger
ponctuellement dans le chat via `#team-principles.rationale` quand une règle est contestée ou ambiguë.

Les règles marquées **[parti pris]** relèvent d'un choix d'équipe plutôt que d'une contrainte technique,
et peuvent être rediscutées.

## Mode de travail

| Règle | Justification |
|---|---|
| Plan avant toute modification de plus de 3 fichiers | Un agent qui part dans la mauvaise direction sur un gros changement produit beaucoup de code à jeter. Valider un plan coûte quelques secondes ; relire 15 fichiers faux coûte bien plus. **[parti pris]** Le seuil de 3 est arbitraire et peut être ajusté. |
| Diffs minimaux | Les modèles ont tendance à « améliorer » le code autour de la tâche (renommages, reformatage, refactoring opportuniste). Cela noie la vraie modification dans la revue et introduit des régressions hors périmètre. |
| Pas de dépendance Composer sans demander | L'agent ajoute facilement un package pour un besoin déjà couvert par Laravel. Chaque dépendance pèse sur la maintenance, la sécurité et la taille du package Lambda. Il arrive aussi qu'il propose un package abandonné, voire inexistant. |
| Ne jamais modifier une migration existante | Une migration déjà jouée en staging ou en production ne sera pas rejouée. La modifier crée un écart silencieux entre les environnements. Les modèles le font souvent pour « corriger » un schéma. |
| Demander plutôt que deviner, expliciter les hypothèses | Face à une ambiguïté, un modèle choisit une interprétation sans le signaler. Rendre les hypothèses visibles permet de les corriger en revue. |

## Règles du langage PHP

| Règle | Justification |
|---|---|
| `declare(strict_types=1)` | Sans mode strict, PHP convertit silencieusement les types (`"12abc"` devient `12`). En mode strict, une erreur de type échoue immédiatement au lieu de corrompre une donnée, ce qui est essentiel quand on manipule des montants. |
| PSR-12 via Pint | Un style unique sur l'ensemble des applications. Le préciser à l'agent évite qu'il produise un code qui fait échouer `pint --test` en CI. |
| Tout typer, éviter `mixed` | Le typage est ce qui permet à PHPStan de détecter les erreurs. Un code généré sans types peut sembler correct tout en cachant des bugs que l'analyse statique aurait trouvés. |
| PHPDoc uniquement pour ce que PHP n'exprime pas | Une PHPDoc qui répète les types natifs finit par diverger du code au fil des modifications. Les formes de tableaux et les génériques, eux, apportent une information réelle à PHPStan. |
| Classes `final` par défaut | L'héritage est un couplage fort, souvent accidentel. `final` pousse vers la composition et sécurise les refactorings : aucune sous-classe cachée ne dépend du comportement interne. **[parti pris]** |
| `readonly` pour DTO et value objects | Un objet immuable ne peut pas être modifié en cours de route par un autre composant. Cela supprime une famille entière de bugs, en particulier quand un objet traverse plusieurs couches. |
| Enums natifs pour les ensembles fermés | Le typage rend une valeur invalide impossible. Un `match` sur un enum est vérifié exhaustivement par PHPStan, et l'enum regroupe au même endroit le comportement lié à chaque valeur. |
| Promotion des propriétés et arguments nommés | Moins de code répétitif. Les arguments nommés rendent lisibles les appels avec plusieurs booléens ou paramètres optionnels. |

## Nommage

| Règle | Justification |
|---|---|
| Tout en anglais | Cohérence avec le framework et l'écosystème. Les modèles produisent aussi un code de meilleure qualité quand le domaine est nommé en anglais. Cela évite les mélanges du type `getFacture()`. |
| Classes au singulier, méthodes en verbes | Une convention prévisible permet à l'agent comme aux développeurs de deviner un nom sans chercher. |
| Booléens sous forme de question | `if ($order->isPaid())` se lit naturellement ; `$order->paid` est ambigu (booléen, date, montant ?). |
| Pas d'abréviations | Les abréviations varient d'une personne à l'autre (`ord`, `odr`, `order`). Une recherche dans le code doit trouver toutes les occurrences. |

## Dépendances et usage du framework

| Règle | Justification |
|---|---|
| Injection par le constructeur | Les dépendances d'une classe sont visibles dans sa signature, testables sans `Facade::fake()` et analysables par PHPStan. **[parti pris]** De nombreuses équipes Laravel utilisent les façades partout. |
| Façades acceptées dans routes, config, providers et tests | C'est leur usage naturel, là où l'injection n'apporte rien. |
| `env()` uniquement dans `config/` | **Règle critique sur Lambda.** Bref met la configuration en cache au cold start ; une fois la config en cache, `env()` renvoie `null` hors des fichiers de config. Le code fonctionne en local et casse en production. |

## Erreurs et exceptions

| Règle | Justification |
|---|---|
| Exceptions métier typées plutôt que `false` / `null` | Une valeur de retour peut être ignorée par l'appelant, une exception non. Le type de l'exception porte la cause, ce qui permet le rendu centralisé vers le format d'erreur de l'équipe. |
| Pas de réponse d'erreur construite dans un contrôleur | Condition pour que toutes les erreurs aient le même format. Sans cette règle, l'agent génère des `response()->json([...])` différents d'un endpoint à l'autre. |
| Ne pas avaler les exceptions | Un `catch` vide transforme un incident visible en donnée corrompue invisible. Conserver l'exception originale en `previous` garde la cause racine dans les logs. |

## Logs et protection des données

| Règle | Justification |
|---|---|
| Logs structurés : message constant + contexte | Dans CloudWatch, on peut filtrer et agréger sur un message fixe et des champs. Un message interpolé (`"Order 42 paid"`) rend chaque ligne unique et impossible à agréger. |
| Pas de secrets ni de données personnelles dans les logs | Obligation RGPD et exigence de conformité en contexte financier (PCI-DSS pour les cartes). Les logs ont une rétention et des droits d'accès bien plus larges que la base de données. |
| Pas de détails internes dans les réponses | Noms de tables, SQL ou stack traces renseignent un attaquant sur l'architecture. |

## Interdits

| Règle | Justification |
|---|---|
| Pas de secrets en dur | Un secret commité reste dans l'historique Git, même après suppression. Les modèles insèrent volontiers des valeurs d'exemple « temporaires » qui finissent en production. |
| Pas de `dd()`, `dump()`, `die()`, `exit()`… | Sur Lambda, `die()` ou `exit()` tue le process FPM en pleine requête. Les fonctions de debug exposent en outre des données dans la réponse. |
| Pas d'opérateur `@` | Il masque les erreurs au lieu de les traiter et rend le diagnostic impossible. |
| Pas de SQL concaténé | Risque d'injection SQL. Les modèles en génèrent encore régulièrement pour les requêtes dynamiques (filtres, tris). |
| Pas de code commenté ni de `TODO` sans ticket | Git conserve l'historique ; le code commenté n'est que du bruit. Un `TODO` sans ticket n'est jamais traité. |

## Définition de « terminé »

| Règle | Justification |
|---|---|
| Tests écrits avec le code | Sans cette consigne, l'agent livre un code « fonctionnel » non vérifié. C'est aussi au moment de l'écriture qu'il a la meilleure compréhension du comportement attendu. |
| Pint, PHPStan et PHPUnit doivent passer | Ce sont les contrôles exécutés en CI. Les exiger de l'agent évite les allers-retours sur des merge requests en échec. |
| Pas de nouvelle entrée dans la baseline PHPStan | Sinon l'agent « corrige » une erreur en l'ajoutant à la baseline, ce qui revient à la masquer. |
| Signaler explicitement ce qui n'a pas été vérifié | Un agent qui n'a pas pu lancer les contrôles a tendance à présenter son travail comme terminé. Cette règle rend la limite visible. |

## Vérification automatique

Une partie de ces règles est vérifiée en CI, pour ne pas dépendre uniquement du respect du steering par l'agent :

| Règle | Outil |
|---|---|
| `declare(strict_types=1)` | Pint, règle `declare_strict_types` |
| Classes `final` | Pint, règle `final_class` |
| Fonctions de debug interdites | PHPStan, extension `ekino/phpstan-banned-code` |
| `env()` hors de `config/` | Larastan, règle dédiée à activer |
| Règles d'architecture | PHPStan, extension `phpat` |
