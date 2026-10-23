# Rozdział 13: część dla użytkownika: rejestracja, profil, zmiana hasła

## Zadanie 1: usunięcie własnego konta

`routes.php`:

```php
$router->post('/profile/delete', [ProfileController::class, 'destroy'], ['auth']);
```

Do `ProfileController`:

```php
    public function destroy(Request $request): Response
    {
        $user = Auth::user();

        if (!password_verify((string) $request->input('delete_password'), $user->password)) {
            return $this->view('profile/edit', [
                'user' => $user,
                'errors' => ['delete_password' => 'Hasło jest nieprawidłowe'],
            ]);
        }

        $user->delete();
        Auth::logout();
        Session::flash('success', 'Konto zostało usunięte');

        return $this->redirect('/login');
    }
```

Na dole `views/profile/edit.php`:

```php
<h2>Usuń konto</h2>

<form method="post" action="/profile/delete" onsubmit="return confirm('Usunąć konto i wszystkie zadania?')">
    <?= csrf_field() ?>

    <p>
        <label>Hasło <input type="password" name="delete_password"></label>
        <?php if ($errors['delete_password'] ?? null): ?>
            <br><small style="color: red"><?= e($errors['delete_password']) ?></small>
        <?php endif; ?>
    </p>

    <button type="submit">Usuń konto</button>
</form>
```

Flash się nie pokaże. `Auth::logout()` woła `Session::destroy()`, które czyści `$_SESSION` i usuwa sesję po stronie serwera (`session_destroy()`). `Session::flash()` wpisuje komunikat do tablicy `$_SESSION`, ale ta tablica nie należy już do żadnej aktywnej sesji, więc na koniec żądania nic jej nie zapisuje. Po przekierowaniu przeglądarka zaczyna z nową, pustą sesją. Laravel ma na to osobne metody: `invalidate()` czyści dane i od razu zakłada nową sesję.

**Na co patrzeć przy review:** pole hasła ma inną nazwę niż `password`, bo w tym samym widoku formularz zmiany hasła wyświetla `$errors['password']`; przy tej samej nazwie błąd pokazałby się pod oboma formularzami (książka nie podaje nazwy pola). Kolejność ma znaczenie: najpierw `delete()`, potem `logout()`; odwrotnie `Auth::user()` byłby już `null`. Trasa musi być POST z CSRF, nigdy link. Każdy `return $this->view('profile/edit', ...)` musi mieć klucz `'user'`.

## Zadanie 2: publiczny profil

`app/Controllers/UserController.php`:

```php
<?php

declare(strict_types=1);

namespace App\Controllers;

use App\Models\Task;
use App\Models\User;
use Core\Controller;
use Core\Request;
use Core\Response;

class UserController extends Controller
{
    public function show(Request $request, string $id): Response
    {
        $user = User::find((int) $id);

        if ($user === null) {
            return new Response('Nie ma takiego użytkownika', 404);
        }

        return $this->view('users/show', [
            'user' => $user,
            'tasksCount' => count(Task::where('user_id', $user->id)),
        ]);
    }
}
```

`views/users/show.php`:

```php
<h1><?= e($user->name) ?></h1>
<p>Konto od: <?= e($user->created_at) ?></p>
<p>Zadań: <?= e($tasksCount) ?></p>
```

`routes.php`:

```php
$router->get('/users/{id}', [UserController::class, 'show'], ['auth']);
```

W `views/profile/edit.php`, pod nagłówkiem:

```php
<p><a href="/users/<?= e(\Core\Auth::id()) ?>">Zobacz, jak widzą mnie inni</a></p>
```

Imię w menu zostaje linkiem do `/profile`. W [rozdziale 15](../../../ebook-dev-start/01-php-miniframework/rozdzialy/15-obsluga-bledow-strony-404-i-500.md) `new Response(..., 404)` zmieni się w `abort(404)`.

**Na co patrzeć przy review:** widok nie może wyświetlać e-maila, roli ani tym bardziej `$user->password`; najprościej sprawdzić, czy uczeń nie przekazuje do widoku całego `toArray()`. Liczenie zadań w kontrolerze albo w modelu, nie w widoku. `(int) $id` przy `find()`. Trasa za `auth`, nie `guest`.

## Zadanie 3: zmiana e-maila wymaga hasła

`ProfileController::update()`:

```php
    public function update(Request $request): Response
    {
        $user = Auth::user();
        $newEmail = trim((string) $request->input('email'));
        $emailChanged = strtolower($newEmail) !== strtolower($user->email);

        $rules = [
            'name' => 'required|min:2|max:100',
            'email' => 'required|email|max:255',
        ];

        if ($emailChanged) {
            $rules['email'] .= '|unique:users,email';
            $rules['email_password'] = 'required';
        }

        $validator = Validator::make($request->all(), $rules);

        if ($validator->fails()) {
            return $this->view('profile/edit', [
                'user' => $user,
                'errors' => $validator->errors(),
                'old' => $request->all(),
            ]);
        }

        if ($emailChanged && !password_verify((string) $request->input('email_password'), $user->password)) {
            return $this->view('profile/edit', [
                'user' => $user,
                'errors' => ['email_password' => 'Obecne hasło jest nieprawidłowe'],
                'old' => $request->all(),
            ]);
        }

        $data = $validator->validated();

        $user->name = $data['name'];
        $user->email = $data['email'];
        $user->save();

        Session::flash('success', 'Profil zapisany');

        return $this->redirect('/profile');
    }
```

W pierwszym formularzu `views/profile/edit.php`, przed przyciskiem:

```php
    <p>
        <label>Obecne hasło (tylko przy zmianie e-maila) <input type="password" name="email_password"></label>
        <?php if ($errors['email_password'] ?? null): ?>
            <br><small style="color: red"><?= e($errors['email_password']) ?></small>
        <?php endif; ?>
    </p>
```

**Na co patrzeć przy review:** hasło sprawdzane tylko przy zmianie e-maila, a samo imię zapisuje się bez hasła. Porównanie adresów po `trim` i `strtolower` (patrz „Częste błędy” rozdziału), inaczej zmiana wielkości liter albo spacja na końcu dają „zajęty adres”. Hasło nie wraca do `value`. Pole nazywa się `email_password`, nie `current_password`: ta druga nazwa jest zajęta przez formularz zmiany hasła i błąd pokazałby się pod oboma formularzami.
