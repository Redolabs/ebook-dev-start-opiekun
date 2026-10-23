# Rozdział 17: testy automatyczne z PHPUnit

## Zadanie 1: CsrfTest

Plik `tests/CsrfTest.php`:

```php
<?php

declare(strict_types=1);

namespace Tests;

use Core\Csrf;
use PHPUnit\Framework\TestCase;

final class CsrfTest extends TestCase
{
    protected function setUp(): void
    {
        parent::setUp();

        $_SESSION = [];
    }

    public function testCheckAcceptsTokenFromSession(): void
    {
        $_SESSION = ['_csrf' => 'abc'];

        $this->assertTrue(Csrf::check('abc'));
    }

    public function testCheckRejectsWrongOrMissingToken(): void
    {
        $_SESSION = ['_csrf' => 'abc'];

        $this->assertFalse(Csrf::check('abd'));
        $this->assertFalse(Csrf::check(null));
    }

    public function testTokenHas64CharsAndDoesNotChange(): void
    {
        $first = Csrf::token();

        $this->assertSame(64, strlen($first));
        $this->assertSame($first, Csrf::token());
    }
}
```

**Na co patrzeć przy review:** `setUp()` czyści `$_SESSION`; bez tego trzeci test zależy od tego, czy wcześniej wykonał się test z `'abc'` (w sesji byłby token `abc` o długości 3). Brak `Session::start()` w testach. Asercje `assertTrue`/`assertFalse`, nie `assertEquals(true, ...)`. Dobra nazwa metod: czyta się jak zdanie w `--testdox`.

## Zadanie 2: test Model na SQLite w pamięci

W `core/Database.php` zmienione `connection()` i nowa metoda `disconnect()`:

```php
    public static function connection(): \PDO
    {
        if (self::$pdo === null) {
            $dsn = Env::get('DB_DSN') ?? sprintf(
                'mysql:host=%s;dbname=%s;charset=utf8mb4',
                Env::get('DB_HOST'),
                Env::get('DB_NAME'),
            );

            self::$pdo = new \PDO($dsn, Env::get('DB_USER'), Env::get('DB_PASS'), [
                \PDO::ATTR_ERRMODE => \PDO::ERRMODE_EXCEPTION,
                \PDO::ATTR_DEFAULT_FETCH_MODE => \PDO::FETCH_ASSOC,
            ]);
        }

        return self::$pdo;
    }

    // Dla testów: następne connection() otworzy nowe połączenie
    public static function disconnect(): void
    {
        self::$pdo = null;
    }
```

Plik `tests/ModelTest.php`:

```php
<?php

declare(strict_types=1);

namespace Tests;

use App\Models\Task;
use Core\Database;
use PHPUnit\Framework\TestCase;

final class ModelTest extends TestCase
{
    protected function setUp(): void
    {
        parent::setUp();

        $_ENV['DB_DSN'] = 'sqlite::memory:';
        Database::disconnect();

        Database::connection()->exec(
            'CREATE TABLE tasks (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                user_id INTEGER NOT NULL DEFAULT 1,
                title VARCHAR(255) NOT NULL,
                done TINYINT(1) NOT NULL DEFAULT 0,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            )'
        );
    }

    protected function tearDown(): void
    {
        Database::disconnect();
        unset($_ENV['DB_DSN']);

        parent::tearDown();
    }

    public function testCreateReturnsSavedTask(): void
    {
        $task = Task::create(['title' => 'Test']);

        $this->assertSame('Test', $task->title);
        $this->assertSame(1, $task->id);
        $this->assertSame(0, $task->done);
    }

    public function testFindReturnsNullForMissingRow(): void
    {
        $this->assertNull(Task::find(999));
    }

    public function testSaveUpdatesOnlyChangedRow(): void
    {
        $task = Task::create(['title' => 'Stary']);
        $task->title = 'Nowy';
        $task->save();

        $this->assertSame('Nowy', Task::find($task->id)->title);
        $this->assertFalse($task->isDirty());
    }
}
```

**Na co patrzeć przy review:** bez `disconnect()` (albo innego sposobu na wyzerowanie singletonu) połączenie z pierwszego testu przechodzi do następnych, więc drugi `CREATE TABLE` rzuca „table tasks already exists”; uczeń często obchodzi to przez `CREATE TABLE IF NOT EXISTS`, co ukrywa problem, bo dane z poprzednich testów zostają w tabeli. `tearDown()` sprząta `$_ENV`, żeby inne testy nie dostały SQLite. Warto zapytać, dlaczego `$task->id` jest `int` (PDO od PHP 8.1 oddaje liczby z SQLite i MySQL jako `int`). Kolumna `user_id` z wartością domyślną jest potrzebna tylko po to, żeby tabela wyglądała jak ta z [rozdziału 12](../../../ebook-dev-start/01-php-miniframework/rozdzialy/12-middleware-i-role.md); test działa i bez niej.

## Zadanie 3: HelpersTest

Plik `tests/HelpersTest.php`:

```php
<?php

declare(strict_types=1);

namespace Tests;

use PHPUnit\Framework\TestCase;

final class HelpersTest extends TestCase
{
    public function testEscapesHtml(): void
    {
        $this->assertSame('&lt;b&gt;', e('<b>'));
    }

    public function testEscapesBothQuotes(): void
    {
        $this->assertSame('&quot;&#039;', e('"\''));
    }

    public function testNullGivesEmptyString(): void
    {
        $this->assertSame('', e(null));
    }
}
```

**Na co patrzeć przy review:** `assertSame`, nie `assertEquals` (dla `e(null)` `assertEquals(null, ...)` by przeszło). Test na cudzysłowy nie jest wymagany, ale sprawdza `ENT_QUOTES`, czyli to, co chroni atrybuty `value="..."`; jeśli uczeń go dopisał, pochwal. Brak `use function` jest poprawny: `e()` jest w globalnej przestrzeni nazw, a PHP przy nieznanej funkcji w `Tests` sam sięga do globalnej.
