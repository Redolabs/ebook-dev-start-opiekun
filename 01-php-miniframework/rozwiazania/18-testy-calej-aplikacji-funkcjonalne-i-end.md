# Rozdział 18: testy całej aplikacji: funkcjonalne i end-to-end

## Zadanie 1: panel admina w testach funkcjonalnych

```php
<?php

declare(strict_types=1);

namespace Tests\Feature;

use App\Models\User;
use Tests\FeatureTestCase;

final class AdminActionsTest extends FeatureTestCase
{
    public function testAdminCannotDeleteHimself(): void
    {
        $admin = $this->createUser(['role' => 'admin']);

        $response = $this->actingAs($admin)->post('/admin/users/' . $admin->id . '/delete');

        $this->assertRedirectTo($response, '/admin/users');
        $this->assertSame('Nie możesz usunąć własnego konta z panelu', $_SESSION['_flash']['error'] ?? null);
        $this->assertNotNull(User::find($admin->id));
    }

    public function testAdminCannotChangeOwnRole(): void
    {
        $admin = $this->createUser(['role' => 'admin']);

        $response = $this->actingAs($admin)->post('/admin/users/' . $admin->id . '/role', ['role' => 'user']);

        $this->assertRedirectTo($response, '/admin/users');
        $this->assertSame('admin', User::find($admin->id)->role);
    }

    public function testAdminCanMakeUserAnAdmin(): void
    {
        $admin = $this->createUser(['role' => 'admin']);
        $ola = $this->createUser(['name' => 'Ola']);

        $response = $this->actingAs($admin)->post('/admin/users/' . $ola->id . '/role', ['role' => 'admin']);

        $this->assertRedirectTo($response, '/admin/users');
        $this->assertSame('admin', User::find($ola->id)->role);
    }

    public function testRoleMustBeUserOrAdmin(): void
    {
        $admin = $this->createUser(['role' => 'admin']);
        $ola = $this->createUser(['name' => 'Ola']);

        $response = $this->actingAs($admin)->post('/admin/users/' . $ola->id . '/role', ['role' => 'boss']);

        $this->assertRedirectTo($response, '/admin/users');
        $this->assertSame('Nieprawidłowa rola', $_SESSION['_flash']['error'] ?? null);
        $this->assertSame('user', User::find($ola->id)->role);
    }

    public function testUserCannotChangeRoles(): void
    {
        $kuba = $this->createUser();

        $response = $this->actingAs($kuba)->post('/admin/users/' . $kuba->id . '/role', ['role' => 'admin']);

        $this->assertStatus($response, 403);
        $this->assertSame('user', User::find($kuba->id)->role);
    }
}
```

Po usunięciu `isMe()` z `destroyUser()` pada `testAdminCannotDeleteHimself`: zamiast komunikatu jest przekierowanie z flashem `success`, a `User::find($admin->id)` zwraca `null`.

**Na co patrzeć przy review:**

- Każdy test sprawdza skutek w bazie (`User::find(...)->role`, `assertNotNull`), a nie tylko status. Samo 302 przyjdzie także wtedy, gdy kontroler nic nie zrobił.
- Status 302 i komunikat flash są sprawdzane oba: `backWithError()` i udana akcja przekierowują na ten sam adres.
- Test 403 dla zwykłego użytkownika sprawdza middleware `role:admin` na trasie POST, nie tylko na `GET /admin`.
- `User::find()` po akcji, a nie stary obiekt `$ola`: obiekt w teście nie wie, że wiersz w bazie się zmienił.
- Nazwy testów czytają się jak zdania w `--testdox`.

## Zadanie 2: dwie przeglądarki w e2e

Zmiany w `tests/E2E/ServerTestCase.php` (pole, `setUp()`, `tearDown()` i nowa metoda):

```php
    // Nazwa sesji => plik z ciasteczkami; każda nazwa to osobna „przeglądarka”
    private array $cookieFiles = [];
    private string $cookieFile;

    protected function setUp(): void
    {
        parent::setUp();

        $this->cookieFiles = [];
        $this->useSession('pierwsza');
    }

    protected function tearDown(): void
    {
        foreach ($this->cookieFiles as $file) {
            unlink($file);
        }

        parent::tearDown();
    }

    // Przełącza na inną „przeglądarkę”; przy pierwszym użyciu nazwy tworzy pusty plik
    protected function useSession(string $name): void
    {
        $this->cookieFiles[$name] ??= tempnam(sys_get_temp_dir(), 'cookies');
        $this->cookieFile = $this->cookieFiles[$name];
    }
```

Plik `tests/E2E/TaskPrivacyTest.php`:

```php
<?php

declare(strict_types=1);

namespace Tests\E2E;

final class TaskPrivacyTest extends ServerTestCase
{
    public function testUserCannotSeeSomeoneElsesTask(): void
    {
        // Ola zakłada konto i dodaje zadanie
        $this->useSession('ola');
        $this->register('Ola', 'ola@example.com');

        $form = $this->get('/tasks/create');
        $this->post('/tasks', ['_token' => $this->csrfToken($form['body']), 'title' => 'Tajny plan Oli']);

        $home = $this->get('/');
        $this->assertSame(1, preg_match('#href="/tasks/(\d+)"#', $home['body'], $matches));
        $taskId = $matches[1];

        // Kuba w drugiej „przeglądarce” próbuje dostać się do zadania Oli
        $this->useSession('kuba');
        $this->register('Kuba', 'kuba@example.com');

        $home = $this->get('/');
        $this->assertStringNotContainsString('Tajny plan Oli', $home['body']);

        $show = $this->get('/tasks/' . $taskId);
        $this->assertSame(404, $show['status']);
        $this->assertStringNotContainsString('Tajny plan Oli', $show['body']);

        $delete = $this->post('/tasks/' . $taskId . '/delete', ['_token' => $this->csrfToken($home['body'])]);
        $this->assertSame(404, $delete['status']);

        // Ola wraca do swojej przeglądarki: zadanie dalej jest
        $this->useSession('ola');
        $this->assertStringContainsString('Tajny plan Oli', $this->get('/')['body']);
    }

    private function register(string $name, string $email): void
    {
        $page = $this->get('/register');

        $response = $this->post('/register', [
            '_token' => $this->csrfToken($page['body']),
            'name' => $name,
            'email' => $email,
            'password' => 'tajne123',
            'password_confirmation' => 'tajne123',
        ]);

        $this->assertSame('/', $response['location']);
    }
}
```

Z `findOwn()` zepsutym na `===` ten test pada na linii z `assertSame(404, $show['status'])`: `Failed asserting that 200 is identical to 404.`

**Na co patrzeć przy review:**

- `TaskFlowTest` dalej przechodzi bez zmian: domyślna sesja tworzy się w `setUp()`.
- Wszystkie pliki z ciasteczkami są usuwane w `tearDown()`, nie tylko ostatni.
- Token do `delete` pochodzi ze strony Kuby, nie Oli. Token Oli dałby 419 i test przeszedłby z niewłaściwego powodu (gdyby ktoś sprawdzał tylko „nie 200”).
- Sprawdzenie na końcu, że zadanie Oli istnieje, odróżnia „Kuba dostał 404” od „zadanie zniknęło”.
- `register()` jako prywatny pomocnik w klasie testu, nie skopiowane dwa razy.
- Wyrażenie `#href="/tasks/(\d+)"#` nie złapie `/tasks/create`, bo `\d+` to same cyfry.

## Zadanie 3: złe hasło w przeglądarce

```js
import { test, expect } from '@playwright/test';

test('logowanie złym hasłem pokazuje komunikat', async ({ page }) => {
    const email = `ola${Date.now()}@example.com`;

    // Konto zakładamy przez formularz, bo z JavaScriptu nie dotykamy bazy
    await page.goto('/register');
    await page.fill('input[name=name]', 'Ola');
    await page.fill('input[name=email]', email);
    await page.fill('input[name=password]', 'tajne123');
    await page.fill('input[name=password_confirmation]', 'tajne123');
    await page.click('button[type=submit]');
    await expect(page).toHaveURL('/');

    await page.getByRole('button', { name: 'Wyloguj' }).click();
    await expect(page).toHaveURL('/login');

    // Złe hasło: zostajemy na /login z czerwonym komunikatem
    await page.fill('input[name=email]', email);
    await page.fill('input[name=password]', 'zle-haslo');
    await page.getByRole('button', { name: 'Zaloguj' }).click();

    await expect(page).toHaveURL('/login');
    await expect(page.locator('main')).toContainText('Nieprawidłowy e-mail lub hasło');

    // Dobre hasło: wchodzimy
    await page.fill('input[name=email]', email);
    await page.fill('input[name=password]', 'tajne123');
    await page.getByRole('button', { name: 'Zaloguj' }).click();

    await expect(page).toHaveURL('/');
    await expect(page.locator('nav')).toContainText('Ola');
});
```

Wynik `npx playwright test`: oba testy przechodzą, Playwright domyślnie uruchamia je równolegle w dwóch procesach („Running 2 tests using 2 workers”). Dzięki unikalnym e-mailom nie przeszkadzają sobie.

**Na co patrzeć przy review:**

- Każde kliknięcie i `expect` ma `await`.
- Konto zakładane przez formularz, a nie wpisane na sztywno: test nie zależy od tego, co jest w bazie.
- Przycisk „Zaloguj” szukany przez `getByRole`. `button[type=submit]` też by zadziałał, bo gość ma na stronie jeden przycisk, ale `getByRole` czyta się jak instrukcja dla człowieka i nie pęknie, gdy w menu dojdzie drugi formularz.
- Po złym haśle sprawdzone i adres, i komunikat. Sam adres `/login` byłby też wtedy, gdyby formularz w ogóle się nie wysłał.
- Druga część z dobrym hasłem pokazuje, że komunikat nie „zablokował” logowania i że flash zniknął po jednym wyświetleniu.
