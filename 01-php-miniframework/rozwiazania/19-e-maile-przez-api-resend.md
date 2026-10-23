# Rozdział 19: e-maile przez API Resend

## Zadanie 1: powiadomienie dla admina

`.env` i `.env.example`:

```
ADMIN_EMAIL=twoj@adres.pl
```

`register()` i nowa metoda w `app/Controllers/AuthController.php` (importy jak w kroku 6):

```php
    public function register(Request $request): Response
    {
        $validator = Validator::make($request->all(), [
            'name' => 'required|min:2|max:100',
            'email' => 'required|email|max:255|unique:users,email',
            'password' => 'required|min:8|confirmed',
        ]);

        if ($validator->fails()) {
            return $this->view('auth/register', [
                'errors' => $validator->errors(),
                'old' => $request->all(),
            ]);
        }

        $data = $validator->validated();

        // Pola wypisane jawnie: rola zostaje domyślna ('user') z tabeli
        $user = User::create([
            'name' => $data['name'],
            'email' => $data['email'],
            'password' => password_hash($data['password'], PASSWORD_DEFAULT),
        ]);

        $this->sendMail($user->email, 'Witaj w liście zadań', 'mail/welcome', [
            'user' => $user,
            'url' => Env::get('APP_URL', 'http://localhost:8000'),
        ]);

        $adminEmail = Env::get('ADMIN_EMAIL', '');

        if ($adminEmail !== '') {
            $this->sendMail($adminEmail, 'Nowy użytkownik: ' . $user->name, 'mail/new-user', [
                'user' => $user,
                'url' => Env::get('APP_URL', 'http://localhost:8000') . '/admin/users',
            ]);
        }

        Auth::attempt($data['email'], $data['password']);
        Session::flash('success', 'Konto założone. Witaj, ' . $data['name'] . '!');

        return $this->redirect('/');
    }

    // Mail to dodatek: błąd trafia do logu, a rejestracja idzie dalej
    private function sendMail(string $to, string $subject, string $view, array $data): void
    {
        try {
            Mail::make()->send($to, $subject, View::renderPartial($view, $data));
        } catch (\Exception $e) {
            ErrorHandler::log($e);
        }
    }
```

Plik `views/mail/new-user.php`:

```php
<!DOCTYPE html>
<html lang="pl">
<body style="font-family: Arial, sans-serif; color: #111827; line-height: 1.5">
    <h1 style="font-size: 20px">Nowy użytkownik</h1>
    <p><strong><?= e($user->name) ?></strong> (<?= e($user->email) ?>) założył konto <?= e($user->created_at) ?>.</p>
    <p><a href="<?= e($url) ?>">Lista użytkowników w panelu admina</a></p>
</body>
</html>
```

**Na co patrzeć przy review:** każdy mail w osobnym `try`, żeby błąd powitania nie zablokował powiadomienia i odwrotnie (w wersji z `sendMail()` wychodzi to samo); `renderPartial` też jest w `try`, bo literówka w nazwie widoku maila to ostrzeżenie zamienione na `ErrorException`; imię i e-mail w widoku przez `e()`, bo admin otwiera mail z danymi wpisanymi przez obcą osobę; temat nie przechodzi przez `e()` i to jest dobrze, bo temat to zwykły tekst, nie HTML (z `e()` admin zobaczyłby `&lt;`); adres admina z `.env`, nie z bazy po roli, bo tak brzmi zadanie (wersja z `User::where('role', 'admin')` też jest w porządku, jeśli uczeń ją uzasadni); przy darmowym planie `ADMIN_EMAIL` musi być adresem właściciela konta Resend albo działać `MAIL_TO_OVERRIDE`.

## Zadanie 2: jedna ponowna próba

Plik `core/ResendMailer.php`:

```php
<?php

declare(strict_types=1);

namespace Core;

class ResendMailer implements Mailer
{
    private const URL = 'https://api.resend.com/emails';

    public function __construct(
        private Http $http,
        private string $apiKey,
        private string $from,
        private ?string $toOverride = null,
        private int $retryDelayMs = 500,
    ) {
    }

    public function send(string $to, string $subject, string $html): void
    {
        if ($this->toOverride !== null) {
            $subject = "[do: $to] $subject";
            $to = $this->toOverride;
        }

        $payload = [
            'from' => $this->from,
            'to' => [$to],
            'subject' => $subject,
            'html' => $html,
        ];

        $result = $this->post($payload);

        // 429 i 5xx to kłopot po stronie usługi, warto spróbować jeszcze raz
        if ($this->shouldRetry($result['status'])) {
            usleep($this->retryDelayMs * 1000);
            $result = $this->post($payload);
        }

        if ($result['status'] !== 200) {
            throw new \RuntimeException(sprintf(
                'Resend odrzucił e-mail (status %d): %s',
                $result['status'],
                $result['body']['message'] ?? 'brak opisu',
            ));
        }
    }

    private function post(array $payload): array
    {
        return $this->http->post(self::URL, $payload, [
            'Authorization' => 'Bearer ' . $this->apiKey,
        ]);
    }

    private function shouldRetry(int $status): bool
    {
        return $status === 429 || $status >= 500;
    }
}
```

Plik `tests/ResendMailerRetryTest.php`:

```php
<?php

declare(strict_types=1);

namespace Tests;

use Core\Http;
use Core\ResendMailer;
use PHPUnit\Framework\TestCase;

final class SequenceHttp extends Http
{
    public int $calls = 0;

    public function __construct(private array $results)
    {
    }

    public function post(string $url, array $json, array $headers = []): array
    {
        $this->calls++;

        return array_shift($this->results);
    }
}

final class ResendMailerRetryTest extends TestCase
{
    public function testRetriesOnceAfter429(): void
    {
        $http = new SequenceHttp([
            ['status' => 429, 'body' => ['message' => 'Too many requests.']],
            ['status' => 200, 'body' => ['id' => 'abc']],
        ]);

        (new ResendMailer($http, 're_x', 'onboarding@resend.dev', null, 0))->send('a@example.com', 'T', '<p>x</p>');

        $this->assertSame(2, $http->calls);
    }

    public function testGivesUpAfterSecondFailure(): void
    {
        $http = new SequenceHttp([
            ['status' => 503, 'body' => []],
            ['status' => 503, 'body' => ['message' => 'API is temporarily unavailable']],
            ['status' => 200, 'body' => []],
        ]);

        try {
            (new ResendMailer($http, 're_x', 'onboarding@resend.dev', null, 0))->send('a@example.com', 'T', '<p>x</p>');
            $this->fail('Powinien polecieć wyjątek');
        } catch (\RuntimeException $e) {
            $this->assertSame(2, $http->calls);
            $this->assertStringContainsString('temporarily unavailable', $e->getMessage());
        }
    }

    public function testDoesNotRetryOn422(): void
    {
        $http = new SequenceHttp([
            ['status' => 422, 'body' => ['message' => 'Invalid `to` field.']],
            ['status' => 200, 'body' => []],
        ]);

        $this->expectException(\RuntimeException::class);

        try {
            (new ResendMailer($http, 're_x', 'onboarding@resend.dev', null, 0))->send('a@example.com', 'T', '<p>x</p>');
        } finally {
            $this->assertSame(1, $http->calls);
        }
    }
}
```

**Na co patrzeć przy review:** dokładnie jedna ponowna próba, nie pętla `while` bez limitu (przy awarii Resend użytkownik czekałby w nieskończoność, a przy 429 dokładalibyśmy żądań do już przeciążonego limitu); brak ponowienia przy 4xx innych niż 429; ten sam `$payload` w obu próbach (uczniowie lubią budować tablicę drugi raz i rozjechać pola); opóźnienie jako parametr konstruktora z wartością domyślną, żeby test nie spał pół sekundy, a `Mail::make()` dalej działało bez zmian; trzeci wynik `200` w drugim teście jest pułapką: jeśli uczeń ponawia dwa razy, test to wyłapie; dobrze, jeśli uczeń zauważy, że `usleep` blokuje żądanie użytkownika i że w Laravelu robi się to w kolejce (można o tym porozmawiać, kodu nie trzeba).

## Zadanie 3: GET /api/health

Plik `app/Controllers/HealthController.php`:

```php
<?php

declare(strict_types=1);

namespace App\Controllers;

use Core\Controller;
use Core\Database;
use Core\Env;
use Core\ErrorHandler;
use Core\Request;
use Core\Response;

class HealthController extends Controller
{
    public function show(Request $request): Response
    {
        try {
            Database::connection()->query('SELECT 1');
            $database = 'ok';
        } catch (\PDOException $e) {
            ErrorHandler::log($e);
            $database = 'error';
        }

        $driver = Env::get('MAIL_DRIVER', 'log');

        $data = [
            'status' => $database === 'ok' ? 'ok' : 'error',
            'database' => $database,
            'mail' => [
                'driver' => $driver,
                // Tylko informacja, czy klucz jest, nigdy sam klucz
                'api_key_set' => Env::get('RESEND_API_KEY', '') !== '',
                'from' => Env::get('MAIL_FROM', ''),
            ],
        ];

        $status = $database === 'ok' ? 200 : 503;
        $body = json_encode($data, JSON_UNESCAPED_UNICODE | JSON_THROW_ON_ERROR);

        return (new Response($body, $status))->header('Content-Type', 'application/json');
    }
}
```

W `routes.php`:

```php
use App\Controllers\HealthController;

$router->get('/api/health', [HealthController::class, 'show']);
```

Przykładowa odpowiedź:

```
{"status":"ok","database":"ok","mail":{"driver":"resend","api_key_set":true,"from":"Lista zadań <onboarding@resend.dev>"}}
```

**Na co patrzeć przy review:** w odpowiedzi nie ma klucza ani jego fragmentu (`substr($key, 0, 6)` też odpada: zawęża zgadywanie i zdradza, który klucz jest w użyciu); `catch (\PDOException)` wokół zapytania, bo bez niego awaria bazy skończy się stroną HTML 500 z `ErrorHandler`, a monitoring chce JSON-a; status 503 przy awarii, nie 200 z `"error"` w treści, bo narzędzia monitorujące patrzą najpierw na status; trasa bez `auth`; endpoint nie wysyła maila ani nie pyta API Resend przy każdym wywołaniu (monitoring odpytuje co minutę, to 1440 żądań dziennie i zjedzony limit); `Response::json()` nie umie ustawić statusu, więc albo `new Response(...)` jak wyżej, albo uczeń dopisał do `json()` drugi parametr `int $status = 200`, co też jest dobrym rozwiązaniem.
