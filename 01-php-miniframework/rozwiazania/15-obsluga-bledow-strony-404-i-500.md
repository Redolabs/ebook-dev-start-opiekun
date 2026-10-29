# Rozdział 15: obsługa błędów, strony 404 i 500, tryb debug

## Zadanie 1: NotFoundException

Plik `core/NotFoundException.php`:

```php
<?php

declare(strict_types=1);

namespace Core;

class NotFoundException extends HttpException
{
    public function __construct(string $message = '')
    {
        parent::__construct(404, $message);
    }
}
```

Ostatnia linijka `Router::dispatch()`:

```php
        throw new NotFoundException();
```

`handle()` działa bez zmian, bo `instanceof` sprawdza nie tylko dokładną klasę, ale też wszystkich przodków: `NotFoundException` dziedziczy po `HttpException`, więc jest `HttpException` (tak jak `Admin` z [rozdziału 0](../../../ebook-dev-start/01-php-miniframework/rozdzialy/00-obiekty-w-php-zanim-zaczniemy.md) jest `Account`). Status 404 bierze się z `status()`, odziedziczonej razem z polem.

**Na co patrzeć przy review:** konstruktor ma wołać `parent::__construct(404, $message)`; bez tego `status()` czyta niezainicjalizowane pole i wybucha. W `handle()` nie może pojawić się osobny `if` dla `NotFoundException`, to cały sens zadania. Test z [rozdziału 17](../../../ebook-dev-start/01-php-miniframework/rozdzialy/17-testy-automatyczne-z-phpunit.md) `testUnknownPathThrowsHttpException` dalej przechodzi z tego samego powodu; dobre pytanie kontrolne.

## Zadanie 2: log na każdy dzień

W `ErrorHandler::log()` jedna linia:

```php
        $file = dirname(__DIR__) . '/storage/logs/app-' . date('Y-m-d') . '.log';
```

Wzorzec `storage/logs/*.log` w `.gitignore` pasuje do każdej nazwy kończącej się na `.log`, więc obejmuje też `app-2026-10-22.log`. Zalety po pół roku: zamiast jednego pliku na setki megabajtów masz małe pliki, błąd z konkretnego dnia znajdujesz po nazwie, a stare logi usuwasz albo archiwizujesz jednym poleceniem (`find storage/logs -name '*.log' -mtime +30 -delete`), nie ruszając bieżącego. Laravel ma to samo jako kanał `daily`.

**Na co patrzeć przy review:** data w formacie `Y-m-d`, żeby pliki sortowały się alfabetycznie w kolejności dni (`d-m-Y` tego nie da). Zostaje `FILE_APPEND`. Uczeń powinien zauważyć, że `tail -f storage/logs/app.log` z rozdziału trzeba teraz uruchamiać na pliku z dzisiejszą datą.

## Zadanie 3: 404 z adresem

W `Router::dispatch()`:

```php
        throw new HttpException(404, 'Nie znaleziono: ' . $request->path());
```

(Jeśli uczeń zrobił zadanie 1, to `throw new NotFoundException('Nie znaleziono: ' . $request->path());`.)

`views/errors/404.php`:

```php
<h1>404: nie ma takiej strony</h1>
<p><?= e($message ?: 'Strona, której szukasz, nie istnieje albo została usunięta.') ?></p>
<p><a href="/">Wróć na stronę główną</a></p>
```

Bez `e()` odpowiedź na `curl "http://localhost:8000/<b>test</b>"` zawiera dosłownie `<p>Nie znaleziono: /<b>test</b></p>`, czyli HTML z adresu trafia do strony bez zmian: to odbity XSS. Przeglądarka zwykle koduje `<` w ścieżce, ale to jej uprzejmość, nie gwarancja. Atakujący nie musi używać przeglądarki, inne klienty i stare przeglądarki kodują inaczej, a ten sam komunikat mógłby kiedyś dostać wartość, którą PHP już dekoduje (np. z `$_GET`, gdzie `?q=<script>` przychodzi jako `<script>`). Zasada się nie zmienia: wszystko, co trafia do HTML-a, idzie przez `e()`.

**Na co patrzeć przy review:** `404.php` musi obsłużyć pusty komunikat, bo `abort(404)` z `findOwn()` nie podaje treści; bez `?:` albo `if` strona pokaże pusty akapit. `e()` przywrócone w commicie. Uczeń, który odpowie „przeglądarka i tak zakoduje, więc `e()` niepotrzebne”, wyciągnął odwrotny wniosek niż trzeba.

## Zadanie 4 (dla chętnych): composer lint

W `composer.json`:

```json
    "scripts": {
        "lint": "find app core public views routes.php -name '*.php' -print0 | xargs -0 -n1 php -l"
    }
```

`-print0` i `-0` przekazują nazwy rozdzielone znakiem zerowym, więc spacje w nazwach plików niczego nie psują. `-n1` daje `php -l` jeden plik naraz, bo tak działa `-l`. Przy błędzie `php -l` kończy się kodem 255, `xargs` przerywa na pierwszym złym pliku, a Composer zgłasza „returned with error code 1”. To wystarczy, żeby zablokować commit albo krok w CI.

`.git/hooks/pre-commit` (potem `chmod +x .git/hooks/pre-commit`):

```sh
#!/bin/sh
composer lint || exit 1
```

Sprawdzone: usunięty średnik daje `Parse error ... on line N`, `composer lint` kończy się kodem 1, a `git commit` się nie wykonuje.

**Na co patrzeć przy review:** `vendor/` poza listą (inaczej lint trwa długo i sprawdza cudzy kod). `routes.php` na liście, bo leży poza katalogami. Hak nie jest częścią repozytorium (`.git/hooks` nie idzie do Gita), więc w PR go nie zobaczysz; poproś o opis w „Jak sprawdzić” albo o skopiowanie go do repo (np. `bin/pre-commit`) z instrukcją instalacji. Cały wynik `composer lint` to lista „No syntax errors detected”; jeśli uczeń przekierował wyjście do `/dev/null`, upewnij się, że błędy (stderr) dalej widać. W [rozdziale 17](../../../ebook-dev-start/01-php-miniframework/rozdzialy/17-testy-automatyczne-z-phpunit.md) `test` trafia do tej samej sekcji `scripts`; druga sekcja `scripts` w JSON-ie nadpisze pierwszą. W [rozdziale 18](../../../ebook-dev-start/01-php-miniframework/rozdzialy/18-testy-calej-aplikacji-funkcjonalne-i-end.md) można dołożyć `- run: composer lint` przed `composer test` w GitHub Actions.
