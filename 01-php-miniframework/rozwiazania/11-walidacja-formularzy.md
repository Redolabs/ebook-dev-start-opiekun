# Rozdział 11: walidacja formularzy

## Zadanie 1: reguła between

W tablicy `$messages`:

```php
        'between' => 'Pole :field musi być liczbą od :min do :max.',
```

Nowa metoda:

```php
    private function validateBetween(string $field, string $value, string $param): bool
    {
        [$min, $max] = explode(',', $param, 2) + [1 => ''];

        return is_numeric($value) && (float) $value >= (float) $min && (float) $value <= (float) $max;
    }
```

W `validate()` zamiast dotychczasowego przypisania błędu:

```php
                if (!$this->$method($field, $value, $param)) {
                    [$min, $max] = explode(',', $param, 2) + [1 => ''];

                    $this->errors[$field] = str_replace(
                        [':field', ':param', ':min', ':max'],
                        [$field, $param, $min, $max],
                        $this->messages[$name],
                    );
                    break;
                }
```

`Validator::make(['n' => '150'], ['n' => 'required|numeric|between:1,100'])` daje `fails() === true` i komunikat „Pole n musi być liczbą od 1 do 100.”, `'50'` przechodzi, a `'abc'` zatrzymuje się na `numeric`.

**Na co patrzeć przy review:** porównanie ma być liczbowe, przez `(float)`. Częsty błąd to `(int)`, które zamienia `'100.4'` na `100` i przepuszcza wartość spoza przedziału. `is_numeric` w samej metodzie chroni przed `between` użytym bez `numeric`. Granice włącznie: `1` i `100` mają przechodzić; poproś o sprawdzenie obu. Rozbicie parametru dla każdej reguły (także `in`) jest nieszkodliwe, bo inne komunikaty nie mają `:min` ani `:max`.

## Zadanie 2: własne komunikaty

Konstruktor i fabryka:

```php
    private function __construct(
        private array $data,
        private array $rules,
        private array $customMessages = [],
    ) {
    }

    public static function make(array $data, array $rules, array $messages = []): self
    {
        $validator = new self($data, $rules, $messages);
        $validator->validate();

        return $validator;
    }
```

W `validate()`, przy błędzie (razem ze zmianą z zadania 1):

```php
                if (!$this->$method($field, $value, $param)) {
                    [$min, $max] = explode(',', $param, 2) + [1 => ''];
                    $template = $this->customMessages["$field.$name"] ?? $this->messages[$name];

                    $this->errors[$field] = str_replace(
                        [':field', ':param', ':min', ':max'],
                        [$field, $param, $min, $max],
                        $template,
                    );
                    break;
                }
```

Użycie w kontrolerze:

```php
        $validator = Validator::make($request->all(), ['title' => 'required|max:255'], [
            'title.required' => 'Wpisz tytuł zadania.',
            'title.max' => 'Tytuł może mieć najwyżej :param znaków.',
        ]);
```

**Na co patrzeć przy review:** nowy parametr musi mieć wartość domyślną `[]`, inaczej wszystkie istniejące wywołania `make()` (i testy z [rozdziału 17](../../../ebook-dev-start/01-php-miniframework/rozdzialy/17-testy-automatyczne-z-phpunit.md)) przestaną działać. Pole na własne komunikaty nie może nazywać się `$messages`, bo tak nazywa się tablica domyślnych komunikatów; uczeń, który nadpisze ją w konstruktorze, zgubi resztę komunikatów. Własne komunikaty też powinny przechodzić przez `str_replace`, żeby `:param` w nich działało.

## Zadanie 3: filtr ?filter= z walidacją (przeróbka zadania 3 z [rozdziału 9](../../../ebook-dev-start/01-php-miniframework/rozdzialy/09-metody-magiczne-czyli-jak-model-udaje.md))

```php
    public function index(Request $request): Response
    {
        $validator = Validator::make($request->all(), ['filter' => 'in:open,done']);
        $filter = $validator->fails() ? '' : $validator->validated()['filter'];

        $tasks = match ($filter) {
            'open' => Task::whereDone(0),
            'done' => Task::whereDone(1),
            default => Task::all(),
        };

        return $this->view('tasks/index', ['tasks' => $tasks, 'filter' => $filter]);
    }
```

Linki nad listą zostają z [rozdziału 9](../../../ebook-dev-start/01-php-miniframework/rozdzialy/09-metody-magiczne-czyli-jak-model-udaje.md), zadanie 3. Brak parametru daje `''`, a puste pole bez `required` przechodzi walidację, więc `default` pokazuje wszystkie.

W [rozdziale 12](../../../ebook-dev-start/01-php-miniframework/rozdzialy/12-middleware-i-role.md) to się zmieni: lista ma pokazywać tylko zadania zalogowanego, a `Task::whereDone()` pokazuje zadania wszystkich. `Model` umie filtrować tylko po jednej kolumnie, więc najprościej przefiltrować w PHP:

```php
        $tasks = Task::where('user_id', Auth::id());

        if ($status !== '') {
            $wanted = $status === 'done' ? 1 : 0;
            $tasks = array_values(array_filter($tasks, fn(Task $task) => (int) $task->done === $wanted));
        }
```

**Na co patrzeć przy review:** zła wartość (`?status=xyz`) ma dać pełną listę, a nie stronę z błędem. Do SQL-a nie może trafić nic z `$status` poza wyborem w `match`. Po [rozdziale 12](../../../ebook-dev-start/01-php-miniframework/rozdzialy/12-middleware-i-role.md) sprawdź koniecznie, czy filtr nie pokazuje cudzych zadań: to najpoważniejszy błąd, jaki może tu zostać, i książka sama o nim nie przypomina.
