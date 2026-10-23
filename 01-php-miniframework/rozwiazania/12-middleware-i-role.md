# Rozdział 12: middleware i role

## Zadanie 1: middleware guest

Rozwiązanie referencyjne jest w [rozdziale 13](../../../ebook-dev-start/01-php-miniframework/rozdzialy/13-czesc-dla-uzytkownika-rejestracja-profil-zmiana.md), krok 1 (klasa `GuestMiddleware`), a rejestracja nazwy `guest` i trasy z nim są w [rozdziale 13](../../../ebook-dev-start/01-php-miniframework/rozdzialy/13-czesc-dla-uzytkownika-rejestracja-profil-zmiana.md), krok 5.

**Na co patrzeć przy review:** `guest` ma stać na obu trasach `/login` (GET i POST), a nie tylko na GET. Nie może stać na trasie z `auth`, a już na pewno nie na `/` (pętla przekierowań). Nazwa musi być zarejestrowana przez `$router->middleware('guest', ...)` na górze `routes.php`, inaczej „Nieznane middleware: guest”. Middleware ma zwracać `Response::redirect('/')`, nie wywoływać `header()`.

## Zadanie 2: kolumna „Zrobionych”

`AdminController::index()`:

```php
    public function index(Request $request): Response
    {
        $users = User::all();
        $counts = [];
        $doneCounts = [];

        foreach ($users as $user) {
            $tasks = Task::where('user_id', $user->id);
            $counts[$user->id] = count($tasks);
            $doneCounts[$user->id] = count(array_filter($tasks, fn(Task $task) => (int) $task->done === 1));
        }

        return $this->view('admin/index', [
            'users' => $users,
            'counts' => $counts,
            'doneCounts' => $doneCounts,
        ]);
    }
```

W `views/admin/index.php` nagłówek `<th>Zrobionych</th>` i komórka `<td><?= e($doneCounts[$user->id]) ?></td>`.

Książka dopuszcza drugie zapytanie w pętli, ale `Model` nie umie warunku na dwóch kolumnach (`user_id` i `done`), więc „drugie zapytanie” wymagałoby nowej metody z SQL-em w modelu. Policzenie zrobionych z już pobranej tablicy jest prostsze i nie dokłada zapytań. Jeśli uczeń dopisał do `Task` metodę z `SELECT COUNT(*) ... WHERE user_id = ? AND done = 1`, to też dobre rozwiązanie, o ile SQL jest w modelu, a nie w kontrolerze.

W [rozdziale 14](../../../ebook-dev-start/01-php-miniframework/rozdzialy/14-panel-admina-na-powaznie.md) lista użytkowników przechodzi na `User::withTaskCounts()`, które liczy tylko wszystkie zadania; żeby kolumna „Zrobionych” przetrwała, do zapytania trzeba dopisać `COALESCE(SUM(tasks.done), 0) AS done_count` i w widoku użyć `$user->done_count`.

**Na co patrzeć przy review:** brak SQL-a w kontrolerze. `(int)` albo luźne porównanie przy `done`: z MySQL na PHP 8.1+ przychodzi `int`, ale `=== '1'` nigdy nie zadziała. Widok przepuszcza liczbę przez `e()` (od [rozdziału 5](../../../ebook-dev-start/01-php-miniframework/rozdzialy/05-widoki-layout-i-kontroler-bazowy.md) `e()` przyjmuje `mixed`).

## Zadanie 3: Router::group()

W `core/Router.php` nowe pole, nowa metoda i zmienione `add()`:

```php
    private array $groupMiddleware = [];

    public function group(array $middleware, callable $callback): void
    {
        $previous = $this->groupMiddleware;
        $this->groupMiddleware = array_merge($previous, $middleware);

        $callback($this);

        $this->groupMiddleware = $previous;
    }

    private function add(string $method, string $path, array $action, array $middleware): void
    {
        $pattern = preg_replace('#\{(\w+)\}#', '(?P<$1>[^/]+)', $path);

        $this->routes[] = [
            'method' => $method,
            'pattern' => '#^' . $pattern . '$#',
            'action' => $action,
            'middleware' => array_merge($this->groupMiddleware, $middleware),
        ];
    }
```

`routes.php`:

```php
$router->group(['auth'], function (Router $router) {
    $router->post('/logout', [AuthController::class, 'logout']);
    $router->get('/', [TaskController::class, 'index']);
    $router->get('/tasks/create', [TaskController::class, 'create']);
    $router->post('/tasks', [TaskController::class, 'store']);
    $router->get('/tasks/{id}', [TaskController::class, 'show']);
    $router->post('/tasks/{id}/done', [TaskController::class, 'done']);
    $router->post('/tasks/{id}/delete', [TaskController::class, 'destroy']);
    $router->get('/api/tasks', [TaskController::class, 'api']);

    $router->get('/admin', [AdminController::class, 'index'], ['role:admin']);
});
```

Na górze `routes.php` potrzebne jest `use Core\Router;` dla typu parametru (albo parametr bez typu).

**Na co patrzeć przy review:** po wyjściu z grupy middleware grupy musi zniknąć; najczęstszy błąd to brak przywrócenia `$previous` (albo `= []` zamiast przywrócenia, co psuje grupy zagnieżdżone), przez co wszystkie trasy pod grupą dostają `auth`, łącznie z `/login`. Middleware grupy ma stać przed middleware trasy, inaczej `['role:admin']` wykona się przed `auth` (eksperyment 4 z rozdziału). Drugi częsty błąd: domknięcie bez parametru, `function () { $router->get(...); }`, daje `Undefined variable $router`, bo funkcja anonimowa nie widzi zmiennych z zewnątrz.
