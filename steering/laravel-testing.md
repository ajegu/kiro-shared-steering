---
inclusion: fileMatch
fileMatchPattern: "tests/**/*.php"
---

# Tests avec PHPUnit

## Syntaxe

- Utilise exclusivement les attributs PHPUnit : `#[Test]`, `#[DataProvider('...')]`, `#[CoversClass(...)]`. N'utilise jamais les annotations en docblock (`@test`, `@dataProvider`, `@covers`) : elles ne sont plus supportées.
- Classes de test `final`, avec `declare(strict_types=1);`.
- Noms de méthodes en snake_case qui décrivent le comportement : `#[Test] public function it_rejects_a_negative_amount(): void`.
- Les data providers sont des méthodes `public static` qui renvoient un `iterable` avec des clés descriptives.
- Structure Arrange / Act / Assert, séparée par une ligne vide.

## Quoi tester

Pour chaque endpoint, un test Feature dans `tests/Feature/Http/` qui couvre au minimum :

1. Le cas nominal : statut, structure et valeurs de la réponse.
2. La validation : un data provider avec les entrées invalides et le `code` d'erreur attendu.
3. L'authentification (401) et l'autorisation (403).
4. La ressource introuvable (404), quand l'endpoint en a une.
5. Le format des erreurs, vérifié au moins une fois par endpoint.

Pour chaque Action, un test Unit ou Feature qui couvre les règles métier et chaque exception qu'elle peut lever.

## Assertions

- Vérifie les valeurs, pas seulement la structure : `assertJsonPath('data.status', 'paid')`.
- Vérifie l'état en base après une écriture : `assertDatabaseHas()`, `assertDatabaseMissing()`.
- Pour une erreur, vérifie au minimum `type`, `code` et le statut HTTP.

## Isolation

- Base de données : trait `RefreshDatabase` (ou `LazilyRefreshDatabase`). Données créées par les factories, jamais d'identifiants en dur.
- Aucun appel réseau : `Http::fake()` avec `Http::preventStrayRequests()`, `Queue::fake()`, `Event::fake()`, `Storage::fake('s3')`.
- SDK AWS : remplace l'implémentation du service dans le container par un fake, ou utilise `Aws\MockHandler`. Un test n'appelle jamais AWS.
- Temps : fige la date avec `$this->travelTo(...)`. Jamais de `sleep()` dans un test.
- Les tests sont indépendants : aucun ordre d'exécution, aucun état partagé entre méthodes.

## Exécution

- `php artisan test` doit passer, y compris en mode `--parallel`.
- Un test qui échoue ne se corrige pas en le supprimant, en le marquant `markTestSkipped()` ou en affaiblissant ses assertions. Si le comportement attendu a changé, dis-le explicitement.
