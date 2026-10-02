---
inclusion: fileMatch
fileMatchPattern: "app/**/*.php"
---

# Architecture applicative Laravel

## Organisation des dossiers

```
app/
├── Actions/            # Cas d'usage métier, une classe par action
├── Data/               # DTO readonly échangés entre les couches
├── Enums/              # Enums natifs (statuts, types, codes d'erreur)
├── Events/  Listeners/ # Effets de bord découplés
├── Exceptions/         # Exceptions métier typées
├── Http/
│   ├── Controllers/
│   ├── Middleware/
│   ├── Requests/       # FormRequest : validation + autorisation
│   └── Resources/      # JsonResource : forme des réponses
├── Jobs/
├── Models/
├── Policies/
└── Services/           # Intégrations externes (AWS, API tierces), derrière une interface
```

Ne crée pas d'autre dossier de premier niveau sans le proposer d'abord.

## Flux d'une requête

`Route` → `FormRequest` (validation, autorisation) → `Controller` → `Action` → `Model` / `Service` → `JsonResource`

- Le contrôleur transforme la requête validée en DTO (`$request->toData()`), appelle une Action et renvoie une `JsonResource`. Il ne contient aucune logique métier et aucune requête Eloquent, hormis le route model binding.
- Un contrôleur de ressource n'utilise que les méthodes standard (`index`, `show`, `store`, `update`, `destroy`). Une action métier hors CRUD (annuler, valider, rembourser) a son propre contrôleur invocable (`__invoke`).

## Actions

- Une Action = un cas d'usage, nommé par un verbe : `CreateOrderAction`, `CancelOrderAction`.
- Une seule méthode publique, `handle()`, qui reçoit un DTO ou des modèles, et renvoie un modèle, un DTO ou `void`.
- Une Action ne connaît pas HTTP : jamais de `Request`, de `Response`, de code HTTP, ni de `JsonResource`. Elle doit pouvoir être appelée depuis un contrôleur, un job ou une commande.
- Une Action qui écrit plusieurs enregistrements le fait dans une transaction, via `ConnectionInterface` injectée : `$this->db->transaction(fn () => ...)`.
- Une Action peut appeler une autre Action, injectée par le constructeur.

## DTO

- Classes `final readonly` dans `app/Data/`, suffixées `Data` : `CreateOrderData`.
- Propriétés typées et promues dans le constructeur. Aucune logique hormis d'éventuelles méthodes de construction nommées (`fromArray()`).

## Services externes

- Toute intégration avec un système externe (AWS, API tierce) passe par une classe de `app/Services/` qui implémente une interface. L'interface est liée dans un service provider.
- Les Actions dépendent de l'interface, jamais du SDK ou du client HTTP directement. C'est ce qui rend les Actions testables sans réseau.

## Événements

- Les effets de bord secondaires (notification, synchronisation, audit) passent par un événement et un listener en queue, pas par un appel direct dans l'Action.
- Un événement émis dans une transaction implémente `ShouldDispatchAfterCommit`, pour ne jamais être traité si la transaction échoue.

## Convention de configuration

- On configure les composants Laravel avec les propriétés de classe (`$fillable`, `$tries`, `$connection`…), pas avec les attributs PHP proposés depuis Laravel 13. Ne mélange jamais les deux styles.
