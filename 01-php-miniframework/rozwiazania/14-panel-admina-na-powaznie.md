# Rozdział 14: panel admina na poważnie

## Zadanie 1: wyszukiwarka użytkowników

Do `app/Models/User.php`:

```php
    public static function search(string $q): array
    {
        return static::query(
            'SELECT users.*, COUNT(tasks.id) AS tasks_count
             FROM users
             LEFT JOIN tasks ON tasks.user_id = users.id
             WHERE users.name LIKE ? OR users.email LIKE ?
             GROUP BY users.id
             ORDER BY users.id',
            ["%$q%", "%$q%"]
        );
    }
```

`AdminController::users()`:

```php
    public function users(Request $request): Response
    {
        $q = trim((string) $request->input('q', ''));
        $users = $q === '' ? User::withTaskCounts() : User::search($q);

        return $this->view('admin/users', ['users' => $users, 'q' => $q]);
    }
```

Nad tabelą w `views/admin/users.php`:

```php
<form method="get" action="/admin/users">
    <input type="search" name="q" value="<?= e($q) ?>" placeholder="Imię lub e-mail">
    <button type="submit">Szukaj</button>
    <?php if ($q !== ''): ?>
        <a href="/admin/users">Wyczyść</a>
    <?php endif; ?>
</form>

<?php if ($users === []): ?>
    <p>Nikogo nie znaleziono.</p>
<?php endif; ?>
```

**Na co patrzeć przy review:** `%` w wartości parametru, nie w SQL-u. Formularz `method="get"` bez `csrf_field()` (GET niczego nie zmienia, a token w adresie trafiłby do historii przeglądarki). `value` przez `e()`. Mało kto zauważy, że `%` i `_` wpisane w pole też działają jak symbole wieloznaczne; to nie dziura, ale warto zapytać.

## Zadanie 2: paginacja

Do `app/Models/User.php`:

```php
    public static function paginateWithTaskCounts(int $page, int $perPage): array
    {
        return static::query(
            'SELECT users.*, COUNT(tasks.id) AS tasks_count
             FROM users
             LEFT JOIN tasks ON tasks.user_id = users.id
             GROUP BY users.id
             ORDER BY users.id
             LIMIT ? OFFSET ?',
            [$perPage, ($page - 1) * $perPage]
        );
    }
```

`AdminController::users()`:

```php
    public function users(Request $request): Response
    {
        $perPage = 10;
        $pages = max(1, (int) ceil(User::count() / $perPage));
        $page = min(max(1, (int) $request->input('page', 1)), $pages);

        return $this->view('admin/users', [
            'users' => User::paginateWithTaskCounts($page, $perPage),
            'page' => $page,
            'pages' => $pages,
        ]);
    }
```

Pod tabelą:

```php
<p>
    <?php if ($page > 1): ?>
        <a href="/admin/users?page=<?= $page - 1 ?>">Poprzednia</a>
    <?php endif; ?>

    Strona <?= e($page) ?> z <?= e($pages) ?>

    <?php if ($page < $pages): ?>
        <a href="/admin/users?page=<?= $page + 1 ?>">Następna</a>
    <?php endif; ?>
</p>
```

**Na co patrzeć przy review:** `max(1, ...)` przed liczeniem `OFFSET`, inaczej `?page=-5` daje ujemny `OFFSET` i błąd bazy. `min(..., $pages)` nie jest wymagane przez książkę, ale bez niego `?page=999` pokazuje pustą tabelę; to dobre pytanie na review. Parametry `LIMIT` i `OFFSET` idą przez `query()` (czyli `bindValue` z `PDO::PARAM_INT`), nie są wklejane do SQL-a. Książka nie mówi, jak paginacja ma współpracować z wyszukiwarką z zadania 1; jeśli uczeń zrobi oba, linki stron muszą nieść `q`, a liczba stron musi liczyć wyniki wyszukiwania, nie `User::count()`. Prostsze wyjście: przy niepustym `q` wyłączyć paginację.

## Zadanie 3: blokowanie konta

`routes.php`:

```php
$router->post('/admin/users/{id}/toggle', [AdminController::class, 'toggleActive'], $admin);
```

Do `AdminController`:

```php
    public function toggleActive(Request $request, string $id): Response
    {
        if ($this->isMe($id)) {
            return $this->backWithError('/admin/users', 'Nie możesz zablokować własnego konta');
        }

        $user = User::find((int) $id);
        if ($user === null) {
            return $this->backWithError('/admin/users', 'Nie ma takiego użytkownika');
        }

        $user->is_active = $user->is_active ? 0 : 1;
        $user->save();

        $action = $user->is_active ? 'Odblokowano' : 'Zablokowano';
        Session::flash('success', "$action użytkownika {$user->name}");

        return $this->redirect('/admin/users');
    }
```

W `views/admin/users.php`, w gałęzi `else` obok pozostałych przycisków:

```php
                    <form method="post" action="/admin/users/<?= e($user->id) ?>/toggle" style="display: inline">
                        <?= csrf_field() ?>
                        <button type="submit"><?= $user->is_active ? 'Zablokuj' : 'Odblokuj' ?></button>
                    </form>
```

W `core/Auth.php`:

```php
    public static function attempt(string $email, string $password): bool
    {
        $user = (self::$userModel)::first('email', $email);

        if ($user === null || !password_verify($password, $user->password)) {
            return false;
        }

        if (!$user->is_active) {
            return false;
        }

        Session::regenerate();
        Session::set('user_id', $user->id);
        self::$user = $user;

        return true;
    }

    public static function user(): ?Model
    {
        if (self::$user === null && Session::get('user_id') !== null) {
            $user = (self::$userModel)::find((int) Session::get('user_id'));

            if ($user !== null && $user->is_active) {
                self::$user = $user;
            } else {
                Session::remove('user_id');
            }
        }

        return self::$user;
    }
```

**Na co patrzeć przy review:** `is_active` sprawdzane po `password_verify`, jak każe książka; odwrotnie ktoś bez hasła mógłby sprawdzić, które konto jest zablokowane. Zablokowany zalogowany użytkownik przy następnym żądaniu ma być traktowany jak gość (przekierowanie na `/login` z `AuthMiddleware`); usunięcie `user_id` z sesji to porządek, nie wymóg. Reguła „nie siebie” przez `isMe()` jak w pozostałych akcjach. Warto zapytać ucznia o koszt: `core/Auth.php` zna teraz kolumnę `is_active` z aplikacji, czyli trochę łamie zasadę „core nie wie nic o app”; czystsza wersja to metoda modelu, np. `canLogIn()`, sprawdzana przez `method_exists`.

## Zadanie 4: edycja użytkownika przez admina

`routes.php`, pod trasami dodawania z kroku 8:

```php
$router->get('/admin/users/{id}/edit', [AdminController::class, 'editUser'], $admin);
$router->post('/admin/users/{id}', [AdminController::class, 'updateUser'], $admin);
```

Do `AdminController`:

```php
    public function editUser(Request $request, string $id): Response
    {
        $user = User::find((int) $id);
        if ($user === null) {
            return $this->backWithError('/admin/users', 'Nie ma takiego użytkownika');
        }

        return $this->view('admin/edit-user', ['user' => $user]);
    }
    public function updateUser(Request $request, string $id): Response
    {
        $user = User::find((int) $id);
        if ($user === null) {
            return $this->backWithError('/admin/users', 'Nie ma takiego użytkownika');
        }

        $rules = [
            'name' => 'required|min:2|max:100',
            'email' => 'required|email|max:255',
        ];

        // Ten sam wyjątek co w profilu: niezmieniony adres już jest w bazie
        if (strtolower(trim((string) $request->input('email'))) !== strtolower($user->email)) {
            $rules['email'] .= '|unique:users,email';
        }

        $validator = Validator::make($request->all(), $rules);

        if ($validator->fails()) {
            return $this->view('admin/edit-user', [
                'user' => $user,
                'errors' => $validator->errors(),
                'old' => $request->all(),
            ]);
        }

        $data = $validator->validated();

        $user->name = $data['name'];
        $user->email = $data['email'];
        $user->save();

        Session::flash('success', "Zapisano dane użytkownika {$user->name}");
        return $this->redirect('/admin/users');
    }
```

`views/admin/edit-user.php`:

```php
<?php require __DIR__ . '/_nav.php'; ?>

<h1>Edycja: <?= e($user->name) ?></h1>

<form method="post" action="/admin/users/<?= e($user->id) ?>">
    <?= csrf_field() ?>

    <p>
        <label>Imię <input type="text" name="name" value="<?= e($old['name'] ?? $user->name) ?>"></label>
        <?php if ($errors['name'] ?? null): ?>
            <br><small style="color: red"><?= e($errors['name']) ?></small>
        <?php endif; ?>
    </p>

    <p>
        <label>E-mail <input type="text" name="email" value="<?= e($old['email'] ?? $user->email) ?>"></label>
        <?php if ($errors['email'] ?? null): ?>
            <br><small style="color: red"><?= e($errors['email']) ?></small>
        <?php endif; ?>
    </p>

    <button type="submit">Zapisz</button>
    <a href="/admin/users">Anuluj</a>
</form>
```

W `views/admin/users.php`, w komórce akcji, przed `<?php if ($user->id === \Core\Auth::id()): ?>`:

```php
                <a href="/admin/users/<?= e($user->id) ?>/edit">Edytuj</a>
```

**Na co patrzeć przy review:** warunek dla `unique` porównuje z `$user->email` edytowanego użytkownika. Częsty błąd to porównanie z `Auth::user()->email`, czyli z adresem admina: wtedy każdy zapis cudzych danych bez zmiany e-maila kończy się „zajętym adresem”. Porównanie ma być po `trim()` i `strtolower()`, jak w [rozdziale 13](../../../ebook-dev-start/01-php-miniframework/rozdzialy/13-czesc-dla-uzytkownika-rejestracja-profil-zmiana.md). Do modelu trafiają tylko `name` i `email`; sprawdź, dopisując w DevTools pola `role=admin` i `password=x`: rola i hasło w bazie mają się nie zmienić. Każde `return $this->view('admin/edit-user', ...)` musi mieć klucz `'user'`, inaczej po błędzie walidacji jest `Undefined variable $user`. Trasy `POST /admin/users/{id}` i `POST /admin/users/{id}/role` się nie gryzą, bo `{id}` nie łapie ukośnika (`[^/]+`), a wzorzec jest zakotwiczony `^...$`; dobre pytanie kontrolne. Edycja własnego konta jest dozwolona (to tylko imię i e-mail), ale jeśli uczeń zrobił zadanie 3 z [rozdziału 13](../../../ebook-dev-start/01-php-miniframework/rozdzialy/13-czesc-dla-uzytkownika-rejestracja-profil-zmiana.md) (zmiana e-maila wymaga hasła), warto zapytać, czy panel nie omija tej ochrony dla konta admina.

## Zadanie 5: wymuszenie zmiany hasła przy pierwszym logowaniu

Migracja:

```sql
ALTER TABLE users ADD COLUMN must_change_password TINYINT(1) NOT NULL DEFAULT 0;
```

W `AdminController::storeUser()` jedna linia więcej w tablicy:

```php
        User::create([
            'name' => $data['name'],
            'email' => $data['email'],
            'role' => $data['role'],
            'password' => password_hash($data['password'], PASSWORD_DEFAULT),
            'must_change_password' => 1,
        ]);
```

Cały `app/Middleware/AuthMiddleware.php`:

```php
<?php

declare(strict_types=1);

namespace App\Middleware;

use Core\Auth;
use Core\Middleware;
use Core\Request;
use Core\Response;
use Core\Session;

class AuthMiddleware extends Middleware
{
    public function handle(Request $request, callable $next): Response
    {
        if (!Auth::check()) {
            Session::flash('error', 'Zaloguj się, żeby zobaczyć tę stronę');
            return Response::redirect('/login');
        }

        // Konto z hasłem startowym: najpierw zmiana hasła, potem reszta aplikacji
        $allowed = ['/profile', '/profile/password', '/logout'];

        if (Auth::user()->must_change_password && !in_array($request->path(), $allowed, true)) {
            Session::flash('error', 'Ustaw własne hasło, zanim zaczniesz korzystać z aplikacji');
            return Response::redirect('/profile');
        }

        return $next($request);
    }
}
```

W `ProfileController::updatePassword()` jedna linia przed `save()`:

```php
        $user->password = password_hash((string) $request->input('password'), PASSWORD_DEFAULT);
        $user->must_change_password = 0;
        $user->save();
```

**Na co patrzeć przy review:** lista wyjątków porównuje całe ścieżki (`in_array(..., true)`), a nie `str_starts_with`, bo `/profile-cokolwiek` nie powinno przechodzić. Bez `/profile` na liście jest pętla przekierowań („za dużo przekierowań”), bez `/profile/password` nie da się zapisać nowego hasła, bez `/logout` użytkownik nie może się wylogować. Odpowiedź na pytanie z treści: w `Auth::attempt()` można by tylko odmówić logowania, a żeby zmienić hasło, trzeba być zalogowanym; middleware wpuszcza, ale tylko na stronę profilu. Flaga zerowana w tym samym `save()` co hasło, czyli jedno `UPDATE`. `POST /profile` (zmiana imienia) też przechodzi, bo ma ścieżkę `/profile`; to akceptowalne, ale dobre pytanie. Admin z flagą też trafia na `/profile`, bo `auth` stoi przed `role:admin`. Uwaga na przyszłość: w [rozdziale 20](../../../ebook-dev-start/01-php-miniframework/rozdzialy/20-react-na-froncie-twoje-api-na.md) (krok 2) książka podaje całe nowe `handle()` z `wantsJson()`; kto zrobił to zadanie, musi przenieść blok z flagą, a dla żądań `/api/...` zamiast przekierowania lepiej `abort(403, ...)`. Z MySQL `must_change_password` przychodzi jako tekst `"0"` albo `"1"`, a `"0"` w PHP jest fałszem, więc warunek działa bez `(int)`.
