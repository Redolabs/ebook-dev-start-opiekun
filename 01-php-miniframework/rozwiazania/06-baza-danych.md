# Rozdział 6: baza danych

## Zadanie 1: show() z zapytaniem z parametrem

```php
<?php

declare(strict_types=1);

namespace App\Controllers;

use Core\Controller;
use Core\Database;
use Core\Request;
use Core\Response;

class TaskController extends Controller
{
    public function index(Request $request): Response
    {
        $tasks = Database::connection()->query('SELECT * FROM tasks')->fetchAll();

        return $this->view('tasks/index', ['tasks' => $tasks]);
    }

    public function show(Request $request, string $id): Response
    {
        $stmt = Database::connection()->prepare('SELECT * FROM tasks WHERE id = ?');
        $stmt->execute([(int) $id]);
        $task = $stmt->fetch();

        if ($task === false) {
            return new Response('Nie ma takiego zadania', 404);
        }

        return $this->view('tasks/show', ['task' => $task]);
    }
}
```

`views/tasks/show.php`:

```php
<h1><?= e($task['title']) ?></h1>
<p>Status: <?= $task['done'] ? 'zrobione' : 'do zrobienia' ?></p>
<p><a href="/">Wróć</a></p>
```

**Na co patrzeć przy review:** zapytanie ma mieć `?` i `execute([...])`, nigdy sklejanie. Porównanie z `false` (a nie z `null`), bo tyle zwraca `fetch()` przy braku wiersza. Widok czyta tablicę (`$task['title']`); w [rozdziale 7](../../../ebook-dev-start/01-php-miniframework/rozdzialy/07-model.md) zmieni się na `$task->title` i `Task::find()`. Częsty błąd: brak `use Core\Response;` po dopisaniu `new Response(..., 404)`.

## Zadanie 2: /tasks/0 OR 1=1

Bez kodu poza tymczasowym `$id = urldecode($id);` na początku `show()` (przeglądarka wysyła `/tasks/0%20OR%201=1`, a `Request::path()` nie dekoduje adresu). Z `prepare` i `?` MySQL porównuje `id` z napisem `'0 OR 1=1'`, zamienia go na liczbę `0` i nie znajduje nic: 404. Ze sklejaniem zapytanie brzmi `WHERE id = 0 OR 1=1`, warunek jest zawsze prawdziwy i `fetch()` zwraca pierwszy wiersz z tabeli, choć nikt o niego nie prosił (`fetchAll()` zwróciłoby wszystkie).

Pułapka: jeśli uczeń w `show()` ma już `(int) $id` z zadania 1, to rzutowanie zamieni `'0 OR 1=1'` na `0` jeszcze przed SQL i obie wersje dadzą 404. Na czas eksperymentu rzutowanie trzeba zdjąć. Książka każe użyć id `0`, a nie `1`, bo napis `'1 OR 1=1'` MySQL zamienia na `1` i wersja bezpieczna pokazałaby zadanie 1, co zaciemnia pokaz.

**Na co patrzeć przy review:** w commicie nie może zostać ani wersja ze sklejaniem, ani `urldecode`. Uczeń ma umieć powiedzieć, że przy `?` dane i kod SQL idą do bazy osobno, więc dane nigdy nie stają się częścią zapytania.
