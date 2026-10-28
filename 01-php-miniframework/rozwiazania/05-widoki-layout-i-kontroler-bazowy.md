# Rozdział 5: widoki, layout i kontroler bazowy

## Zadanie 1: widok show

Do `TaskController`:

```php
    public function show(Request $request, string $id): Response
    {
        return $this->view('tasks/show', ['id' => $id]);
    }
```

`views/tasks/show.php`:

```php
<h1>Zadanie numer <?= e($id) ?></h1>
<p><a href="/">Wróć</a></p>
```

`routes.php`:

```php
$router->get('/tasks/{id}', [TaskController::class, 'show']);
```

**Na co patrzeć przy review:** `$id` pochodzi z adresu, więc w widoku musi iść przez `e()`. Kontroler nie może zawierać HTML-a (`new Response('<h1>...')`), to cały sens rozdziału. Sprawdź, że `HomeController.php` jest usunięty, a trasa `/tasks/{id}` wskazuje na `TaskController`. W [rozdziale 6](../../../ebook-dev-start/01-php-miniframework/rozdzialy/06-baza-danych.md) `show()` zacznie pobierać zadanie z bazy, a w [rozdziale 7](../../../ebook-dev-start/01-php-miniframework/rozdzialy/07-model.md) dostanie obiekt `Task`.

## Zadanie 2: tytuł strony w layoucie

W `TaskController::index()`:

```php
        return $this->view('tasks/index', ['tasks' => $tasks, 'title' => 'Zadania']);
```

W `views/layout.php`:

```php
    <title><?= e($title ?? 'Lista zadań') ?></title>
```

Odpowiedź na pytanie: `View::render()` robi `extract($data)`, co tworzy zmienne lokalne w tej funkcji, a oba `require` (widok i layout) wykonują się w tej samej funkcji, więc widzą te same zmienne lokalne. Tak samo layout widzi `$content`.

**Na co patrzeć przy review:** `??` i `e()` muszą być oba. Bez `??` każda strona bez tytułu daje ostrzeżenie (od [rozdziału 15](../../../ebook-dev-start/01-php-miniframework/rozdzialy/15-obsluga-bledow-strony-404-i-500.md) nawet 500). Dobrze, jeśli uczeń sam zauważy skutek uboczny: widok może nadpisać zmienną layoutu, np. ustawić u siebie `$title`. Pełne listingi layoutu w [rozdziałach 13](../../../ebook-dev-start/01-php-miniframework/rozdzialy/13-czesc-dla-uzytkownika-rejestracja-profil-zmiana.md) i 16 mają już `e($title ?? 'Lista zadań')`, więc layout jest bezpieczny. Gubi się za to `'title' => 'Zadania'` w kontrolerze: listingi `index()` w rozdziałach 7, 8 i 12 go nie mają, a uczeń, który wkleja je na ślepo, traci rozwiązanie (tak było u Supereeeq w rozdziałach 7 i 8). Książka ma od rozdziału 5 akapit „Twoje zadania a pełne listingi z książki” i przypomnienia przy listingach.
