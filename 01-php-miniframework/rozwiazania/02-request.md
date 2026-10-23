# Rozdział 2: Request

## Zadanie 1: isPost() i formularz

Do `core/Request.php`:

```php
    public function isPost(): bool
    {
        return $this->method === 'POST';
    }
```

`public/index.php`:

```php
<?php

declare(strict_types=1);

require __DIR__ . '/../vendor/autoload.php';

use Core\Request;

$request = Request::fromGlobals();

echo '<form method="post" action="/">';
echo '<input type="text" name="title"> <button type="submit">Wyślij</button>';
echo '</form>';

if ($request->isPost()) {
    echo '<p>Wysłano: ', htmlspecialchars((string) $request->input('title'), ENT_QUOTES, 'UTF-8'), '</p>';
} else {
    var_dump($request->input('title'));
}
```

**Na co patrzeć przy review:** `isPost()` ma korzystać z `$this->method` (już zamienionego na wielkie litery), a nie znowu z `$_SERVER`. Uczeń zwykle wypisuje tytuł bez `htmlspecialchars`; na tym etapie to jeszcze nie błąd w ocenie, ale warto rzucić „a co, jeśli wpiszę `<b>`?” jako zapowiedź [rozdziału 5](../../../ebook-dev-start/01-php-miniframework/rozdzialy/05-widoki-layout-i-kontroler-bazowy.md). Sprawdzenie „przy GET daje `null`” działa tylko pod gołym `/`: pod `/?title=x` metoda `input()` zwróci `x`, bo czyta też query string. Jeśli uczeń to zauważy, to dobry znak.
