# Rozdział 1: przygotowanie projektu

## Zadanie 1: klasa App\Models\Task

Plik `app/Models/Task.php`:

```php
<?php

declare(strict_types=1);

namespace App\Models;

class Task
{
    public function title(): string
    {
        return 'Pierwsze zadanie z autoloadu';
    }
}
```

`public/index.php`:

```php
<?php

declare(strict_types=1);

require __DIR__ . '/../vendor/autoload.php';

use App\Models\Task;
use Core\Hello;

$hello = new Hello();
echo $hello->say(), '<br>';

$task = new Task();
echo $task->title();
```

**Na co patrzeć przy review:** w commicie nie może być zmiany w `composer.json`, a plik musi leżeć dokładnie w `app/Models/Task.php` (wielkie `M`, wielkie `T`). Nie trzeba nawet `composer dump-autoload`, bo reguła PSR-4 rozwiązuje nazwy w locie; jeśli uczeń twierdzi, że musiał, zapytaj, czy przypadkiem nie poprawiał wcześniej literówki w ścieżce. Zapytaj ustnie, jak PHP znalazł plik: oczekiwana odpowiedź to „`App\` mapuje się na `app/`, reszta nazwy to ścieżka”. W [rozdziale 7](../../../ebook-dev-start/01-php-miniframework/rozdzialy/07-model.md) ta klasa zmieni się w model dziedziczący po `Core\Model`.
