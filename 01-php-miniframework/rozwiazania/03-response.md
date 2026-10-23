# Rozdział 3: Response

## Zadanie 1: Response::json() i /api

Do `core/Response.php`:

```php
    public static function json(array $data): self
    {
        $body = json_encode($data, JSON_UNESCAPED_UNICODE | JSON_THROW_ON_ERROR);

        return (new self($body))->header('Content-Type', 'application/json');
    }
```

W `public/index.php` dodatkowa gałąź przed `else`:

```php
} elseif ($request->path() === '/api') {
    $response = Response::json(['ok' => true]);
```

Wersja referencyjna metody jest też w [rozdziale 9](../../../ebook-dev-start/01-php-miniframework/rozdzialy/09-metody-magiczne-czyli-jak-model-udaje.md), krok 7 (identyczna) i w [rozdziale 17](../../../ebook-dev-start/01-php-miniframework/rozdzialy/17-testy-automatyczne-z-phpunit.md), krok 3 (bez flag).

**Na co patrzeć przy review:** metoda ma być statyczna i zwracać `self`, zbudowana tak jak `redirect()`, czyli przez `header()`, a nie przez wywołanie `header()` z PHP. Najczęstszy błąd to `echo json_encode(...)` w `index.php` albo bezpośrednie `header('Content-Type: ...')`, czyli dokładnie to, czego rozdział zakazuje. Drugi: pominięty nagłówek, przez co przeglądarka pokazuje JSON jako tekst; w DevTools w Network powinno być `Content-Type: application/json`. Flagi `json_encode` nie są wymagane, ale bez `JSON_THROW_ON_ERROR` błąd kodowania daje `false` i `TypeError` w konstruktorze.
