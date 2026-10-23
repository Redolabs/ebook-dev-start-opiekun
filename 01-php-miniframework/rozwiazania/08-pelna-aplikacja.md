# Rozdział 8: pełna aplikacja

## Zadanie 1: edycja tytułu

`routes.php` (dopisane do listy z rozdziału):

```php
$router->get('/tasks/{id}/edit', [TaskController::class, 'edit']);
$router->post('/tasks/{id}', [TaskController::class, 'update']);
```

Do `TaskController`:

```php
    public function edit(Request $request, string $id): Response
    {
        $task = Task::find((int) $id);

        if ($task === null) {
            return new Response('Nie ma takiego zadania', 404);
        }

        return $this->view('tasks/edit', ['task' => $task]);
    }

    public function update(Request $request, string $id): Response
    {
        $task = Task::find((int) $id);

        if ($task === null) {
            return new Response('Nie ma takiego zadania', 404);
        }

        $title = trim((string) $request->input('title', ''));

        if ($title === '') {
            return $this->view('tasks/edit', ['task' => $task, 'error' => 'Tytuł nie może być pusty']);
        }

        $task->update(['title' => $title]);

        return $this->redirect('/tasks/' . $task->id);
    }
```

`views/tasks/edit.php`:

```php
<h1>Edycja zadania</h1>

<?php if (!empty($error)): ?>
    <p style="color: red"><?= e($error) ?></p>
<?php endif; ?>

<form method="post" action="/tasks/<?= e($task->id) ?>">
    <label>Tytuł <input type="text" name="title" value="<?= e($task->title) ?>" autofocus></label>
    <button type="submit">Zapisz</button>
</form>

<p><a href="/tasks/<?= e($task->id) ?>">Anuluj</a></p>
```

W `views/tasks/show.php` link: `<a href="/tasks/<?= e($task->id) ?>/edit">Edytuj</a>`.

To rozwiązanie zmieni się kilka razy: w [rozdziale 10](../../../ebook-dev-start/01-php-miniframework/rozdzialy/10-sesje-logowanie-i-csrf.md) formularz dostaje `<?= csrf_field() ?>`, w [rozdziale 11](../../../ebook-dev-start/01-php-miniframework/rozdzialy/11-walidacja-formularzy.md) (krok 5) `update()` dostaje Validator i styl `$task->title = ...; $task->save();`, w [rozdziale 12](../../../ebook-dev-start/01-php-miniframework/rozdzialy/12-middleware-i-role.md) obie metody powinny używać `findOwn()` i obie trasy dostać `['auth']` (książka o tym nie wspomina, a pełne `routes.php` z rozdziałów 12 i 13 tych tras nie mają), a w [rozdziale 15](../../../ebook-dev-start/01-php-miniframework/rozdzialy/15-obsluga-bledow-strony-404-i-500.md) `new Response(..., 404)` zamienia się w `abort(404)`.

**Na co patrzeć przy review:** do `update()` modelu ma iść tablica z jednym kluczem wpisanym w kodzie, nigdy `$request->all()` ani `$_POST`. `value` w formularzu musi przejść przez `e()`, inaczej cudzysłów w tytule psuje stronę. Przekierowanie po zapisie (POST, redirect, GET), a nie renderowanie widoku. Częsty błąd: trasa `get` zamiast `post` dla zapisu albo `action` formularza bez id.

## Zadanie 2: tytuł `<script>alert('xss')</script>`

Bez kodu. Za brak alertu odpowiada funkcja `e()` z `core/helpers.php` (czyli `htmlspecialchars` z `ENT_QUOTES`), wywołana wszędzie, gdzie tytuł trafia do HTML-a: `views/tasks/index.php` (link z tytułem), `views/tasks/show.php` (nagłówek) i `views/tasks/edit.php` (atrybut `value`). Tytuł jest zapisany w bazie dosłownie, z `<script>`; kodowanie dzieje się dopiero przy wyświetlaniu.

**Na co patrzeć przy review:** uczeń zwykle wymienia jedno miejsce. Poproś, żeby przeszukał projekt (`grep -rn 'title' views/`) i znalazł każde. Jeśli twierdzi, że „baza wycięła skrypt”, sprawdźcie razem w MySQL: tytuł jest tam w całości. To dobry moment, żeby powiedzieć: dane zapisujemy takie, jakie przyszły, a kodujemy przy wypisywaniu, w kontekście HTML.

## Zadanie 3: tytuł `Robert'); DROP TABLE tasks;--`

Bez kodu. `Task::create()` buduje `INSERT INTO tasks (title) VALUES (?)` i przekazuje tytuł osobno w `execute()`. Baza dostaje treść zapytania i wartość oddzielnie, więc apostrof i średnik są zwykłymi znakami w tekście, a nie częścią SQL-a. Nazwa kolumny `title` jest wpisana w kodzie kontrolera, nie przychodzi z formularza.

**Na co patrzeć przy review:** odpowiedź ma zawierać „prepared statement” albo „znak zapytania i osobne dane”, a nie „PDO filtruje znaki” czy „escapuje apostrofy”. Ta druga wersja jest myląca i warto ją sprostować.
