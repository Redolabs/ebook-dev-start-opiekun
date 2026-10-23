# Rozdział 9: metody magiczne, czyli jak Model udaje, że ma pola

## Zadanie 1: __get z wyjątkiem

```php
    public function __get(string $name): mixed
    {
        if (!array_key_exists($name, $this->attributes)) {
            throw new \RuntimeException("Brak kolumny $name");
        }

        return $this->attributes[$name];
    }
```

`$task->nie_ma ?? 'brak'` dalej daje `brak`: operator `??` najpierw pyta `__isset`, ta zwraca `false`, więc PHP w ogóle nie woła `__get`. Odpowiedź na pytanie o lepszy wariant nie ma jednej poprawnej wersji. Za wyjątkiem: literówka `$task->titel` w widoku wybucha od razu z nazwą kolumny, zamiast cicho wypisać pusty tekst. Przeciw: nowy obiekt bez `id` (`new Task()` przed `save()`) rzuca wyjątek przy odczycie `$task->id` czy `$task->done`, choć te kolumny istnieją w tabeli, a każdy odczyt opcjonalnej kolumny musi iść przez `??`. Laravel domyślnie zwraca `null`, a wyjątek włącza `Model::preventAccessingMissingAttributes()`.

**Na co patrzeć przy review:** `array_key_exists`, nie `isset`; z `isset` kolumna z wartością `NULL` (np. pusta data) też rzuci wyjątek. Wyjaśnienie o `??` musi wskazywać `__isset`, a nie „bo `??` łapie wyjątki” (nie łapie). Książka nie mówi, czy uczeń ma zostawić tę wersję na dalsze rozdziały: obie działają z resztą kodu książki, więc zostaw decyzję uczniowi, ale niech umie ją uzasadnić.

## Zadanie 2: fresh() i refresh()

```php
    public function fresh(): ?static
    {
        if (!isset($this->attributes['id'])) {
            return null;
        }

        return static::find((int) $this->attributes['id']);
    }

    public function refresh(): void
    {
        $fresh = $this->fresh();

        if ($fresh === null) {
            throw new \RuntimeException('Tego wiersza nie ma w bazie');
        }

        $this->attributes = $fresh->attributes;
        $this->dirty = [];
    }
```

**Na co patrzeć przy review:** `fresh()` nie może zmieniać `$this`, a `refresh()` nie może tworzyć nowego obiektu na zewnątrz ani zwracać go zamiast zmieniać siebie. `refresh()` ma czyścić `$dirty`, inaczej następne `save()` nadpisze świeże dane starymi zmianami. Dostęp do `$fresh->attributes` jest legalny, bo `private` w PHP dotyczy klasy, nie obiektu (to samo co w `save()`); jeśli uczeń zrobił `attributes` `protected` „bo nie działało”, wróćcie do kroku 3. Brak obsługi usuniętego wiersza (`find()` zwraca `null`) to częsty błąd: `null->attributes` daje ostrzeżenie i pusty model.

## Zadanie 3: filtr ?filter=open i ?filter=done

```php
    public function index(Request $request): Response
    {
        $filter = $request->input('filter');

        $tasks = match ($filter) {
            'open' => Task::whereDone(0),
            'done' => Task::whereDone(1),
            default => Task::all(),
        };

        return $this->view('tasks/index', ['tasks' => $tasks]);
    }
```

Nad listą w `views/tasks/index.php`:

```php
<p>
    <a href="/">Wszystkie</a> |
    <a href="/?filter=open">Do zrobienia</a> |
    <a href="/?filter=done">Zrobione</a>
</p>
```

Odpowiedź na pytanie: `where()` wkleja nazwę kolumny prosto do SQL-a, bez `?`. `Task::where($request->input('filter'), 1)` oddaje użytkownikowi kawałek zapytania: `?filter=1=1 OR id` daje `WHERE 1=1 OR id = ?`, a nieistniejąca nazwa daje błąd bazy. `match` działa jak biała lista: do SQL-a trafia tylko nazwa kolumny wpisana w kodzie.

**Na co patrzeć przy review:** `match` z gałęzią `default`, bez niej `?filter=xyz` rzuca `UnhandledMatchError`. [Rozdział 11](../../../ebook-dev-start/01-php-miniframework/rozdzialy/11-walidacja-formularzy.md) (zadanie 3) robi prawie to samo z parametrem `status` i walidacją; jeśli uczeń zrobi oba, powinien mieć jeden filtr, nie dwa. Od [rozdziału 12](../../../ebook-dev-start/01-php-miniframework/rozdzialy/12-middleware-i-role.md) lista ma pokazywać tylko zadania zalogowanego, a `Task::whereDone()` pokazuje wszystkich, więc filtr trzeba przerobić (kod przy [rozdziale 11](../../../ebook-dev-start/01-php-miniframework/rozdzialy/11-walidacja-formularzy.md), zadanie 3).
