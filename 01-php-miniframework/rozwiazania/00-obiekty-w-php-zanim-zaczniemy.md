# Rozdział 0: obiekty w PHP, zanim zaczniemy

## Zadanie 1: portfel

```php
<?php

declare(strict_types=1);

class NotEnoughMoneyException extends \Exception
{
}

class Wallet
{
    private function __construct(private int $balance)
    {
    }

    public static function empty(): self
    {
        return new self(0);
    }

    public function balance(): int
    {
        return $this->balance;
    }

    public function deposit(int $amount): void
    {
        $this->assertPositive($amount);
        $this->balance += $amount;
    }

    public function withdraw(int $amount): void
    {
        $this->assertPositive($amount);

        if ($amount > $this->balance) {
            throw new NotEnoughMoneyException("Masz {$this->balance} gr, a chcesz wypłacić $amount gr");
        }

        $this->balance -= $amount;
    }

    private function assertPositive(int $amount): void
    {
        if ($amount <= 0) {
            throw new \InvalidArgumentException('Kwota musi być większa od zera');
        }
    }
}

$wallet = Wallet::empty();
$wallet->deposit(5000);
$wallet->withdraw(1999);
echo 'Saldo: ', $wallet->balance(), ' gr', PHP_EOL;

try {
    $wallet->deposit(-100);
} catch (\InvalidArgumentException $e) {
    echo 'Błąd: ', $e->getMessage(), PHP_EOL;
}

try {
    $wallet->withdraw(100000);
} catch (NotEnoughMoneyException $e) {
    echo 'Błąd: ', $e->getMessage(), PHP_EOL;
}

try {
    $wallet->balance = 1000000;
} catch (\Error $e) {
    echo 'Błąd: ', $e->getMessage(), PHP_EOL;
}
```

Wynik: `Saldo: 3001 gr`, potem trzy komunikaty błędów. Ostatni to `Cannot access private property Wallet::$balance`.

**Na co patrzeć przy review:** sprawdzanie kwoty powinno być w jednym miejscu (prywatna metoda), a nie skopiowane do obu metod. Częsty błąd: `withdraw()` sprawdza tylko `> 0`, a nie saldo, albo sprawdza saldo przed sprawdzeniem znaku kwoty, więc `withdraw(-5)` zwiększa saldo. Uczeń często łapie wszystko jednym `catch (\Exception $e)`: zapytaj, czym to się różni od osobnych `catch`, i zauważ, że błąd zapisu do pola `private` to `\Error`, a nie `\Exception`, więc taki `catch` go nie złapie. Konstruktor `private` nie jest wymagany, ale jeśli uczeń go zrobi, to dobry znak, że rozumie sens fabryki.

## Zadanie 2: figury

```php
<?php

declare(strict_types=1);

abstract class Shape
{
    abstract public function area(): float;

    public function describe(): string
    {
        return static::class . ': ' . round($this->area(), 2);
    }
}

class Circle extends Shape
{
    public function __construct(private float $radius)
    {
    }

    public function area(): float
    {
        return M_PI * $this->radius ** 2;
    }
}

class Rectangle extends Shape
{
    public function __construct(private float $width, private float $height)
    {
    }

    public function area(): float
    {
        return $this->width * $this->height;
    }
}

$shapes = [new Circle(5), new Rectangle(3, 4), new Circle(1)];

echo implode(PHP_EOL, array_map(fn(Shape $shape) => $shape->describe(), $shapes)), PHP_EOL;

$total = array_sum(array_map(fn(Shape $shape) => $shape->area(), $shapes));
echo 'Suma pól: ', round($total, 2), PHP_EOL;
```

Wynik: `Circle: 78.54`, `Rectangle: 12`, `Circle: 3.14`, `Suma pól: 93.68`. Po zamianie na `self::class` każda linia zaczyna się od `Shape`, bo `self` to klasa, w której napisano kod (`Shape`), a `static` to klasa, na której wywołano metodę (to samo, co krok 7 z `new self()` i `new static()`).

**Na co patrzeć przy review:** pola w `Circle` i `Rectangle` mają być przez promocję w konstruktorze i `private` (albo `protected`), nie `public`. `describe()` ma być tylko w `Shape`; jeśli uczeń napisał je osobno w każdej figurze, nie zrozumiał, po co jest klasa bazowa. Wyjaśnienie różnicy `self`/`static` musi padać słowami „gdzie napisano” i „na czym wywołano”, to fundament [rozdziału 7](../../../ebook-dev-start/01-php-miniframework/rozdzialy/07-model.md). `new Circle(5)` z liczbą całkowitą działa mimo `strict_types`, bo PHP pozwala przekazać `int` tam, gdzie jest `float`; uczeń może o to zapytać.

## Zadanie 3: Twoje stare logowanie

```php
<?php

declare(strict_types=1);

class InvalidCredentialsException extends \Exception
{
}

class User
{
    public function __construct(
        private int $id,
        private string $name,
        private string $email,
        private string $passwordHash,
    ) {
    }

    public function id(): int
    {
        return $this->id;
    }

    public function name(): string
    {
        return $this->name;
    }

    public function email(): string
    {
        return $this->email;
    }

    public function checkPassword(string $password): bool
    {
        return password_verify($password, $this->passwordHash);
    }
}

class LoginService
{
    /** @param User[] $users */
    public function __construct(private array $users)
    {
    }

    public function attempt(string $email, string $password): User
    {
        foreach ($this->users as $user) {
            if ($user->email() === $email && $user->checkPassword($password)) {
                return $user;
            }
        }

        throw new InvalidCredentialsException('Nieprawidłowy e-mail lub hasło');
    }
}

$service = new LoginService([
    new User(1, 'Jakub', 'jakub@example.com', password_hash('tajne123', PASSWORD_DEFAULT)),
    new User(2, 'Ola', 'ola@example.com', password_hash('haslo456', PASSWORD_DEFAULT)),
]);

foreach ([['jakub@example.com', 'tajne123'], ['jakub@example.com', 'zle'], ['nikt@example.com', 'x']] as [$email, $password]) {
    try {
        $user = $service->attempt($email, $password);
        echo 'Zalogowano: ', $user->name(), PHP_EOL;
    } catch (InvalidCredentialsException $e) {
        echo 'Błąd: ', $e->getMessage(), PHP_EOL;
    }
}
```

Odpowiedź na pytanie z zadania: sprawdzanie hasła jest w modelu (`User::checkPassword`) i w serwisie, który nic nie wypisuje, a wypisywanie komunikatu jest w kodzie wywołującym, w `catch`. W starym skrypcie z `mysqli` zwykle jedno i drugie siedzi w jednym `if` razem z `echo`.

**Na co patrzeć przy review:** hasło w tablicy użytkowników ma być hashem z `password_hash`, a porównanie przez `password_verify`, nie `===`. Hash nie powinien wychodzić z obiektu przez getter, jeśli nie musi; metoda `checkPassword()` jest lepsza niż `passwordHash()`. Częsty błąd: dwa różne komunikaty („nie ma takiego e-maila” i „złe hasło”), co zdradza, które konta istnieją; w [rozdziale 10](../../../ebook-dev-start/01-php-miniframework/rozdzialy/10-sesje-logowanie-i-csrf.md) ten sam temat wraca. Jeśli `attempt()` zwraca `null` albo `false` zamiast rzucać wyjątek, uczeń nie wykonał polecenia.
