# Rozdział 7: Model

## Zadanie 1: Model::update()

Rozwiązanie referencyjne jest w [rozdziale 9](../../../ebook-dev-start/01-php-miniframework/rozdzialy/09-metody-magiczne-czyli-jak-model-udaje.md), krok 1.

**Na co patrzeć przy review:** kolejność wartości w `execute()` musi odpowiadać kolejności `?`: najpierw kolumny, `id` na końcu. Najczęstszy błąd to `[$this->id, ...array_values($data)]`, przez co `UPDATE` trafia w zły wiersz albo w żaden. Drugi: brak `$this->attributes = array_merge(...)`, więc po `update()` obiekt pokazuje starą wartość ([rozdział 9](../../../ebook-dev-start/01-php-miniframework/rozdzialy/09-metody-magiczne-czyli-jak-model-udaje.md) każe to dopisać, więc nie musisz jeszcze tego wymagać, ale warto zapytać „co pokaże `$task->done` po `markDone()`?”). Klucze `$data` idą do SQL jako nazwy kolumn: zapytaj ucznia, skąd będą pochodzić.

## Zadanie 2: Task::markDone()

```php
<?php

declare(strict_types=1);

namespace App\Models;

use Core\Model;

class Task extends Model
{
    protected static string $table = 'tasks';

    public function markDone(): void
    {
        $this->update(['done' => 1]);
    }
}
```

W [rozdziale 9](../../../ebook-dev-start/01-php-miniframework/rozdzialy/09-metody-magiczne-czyli-jak-model-udaje.md) (krok 3) to się zmieni w `$this->done = 1; $this->save();`.

**Na co patrzeć przy review:** metoda ma być w `Task`, nie w `Model`, i nie może mieć SQL-a; jedyna linia to wywołanie `update()`. Jeśli uczeń napisał tu `prepare('UPDATE tasks ...')`, nie zrozumiał, po co jest `update()` w klasie bazowej.

## Zadanie 3: tabela projects i model Project

```sql
CREATE TABLE projects (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL
);

INSERT INTO projects (name) VALUES ('Dom'), ('Szkoła');
```

`app/Models/Project.php`:

```php
<?php

declare(strict_types=1);

namespace App\Models;

use Core\Model;

class Project extends Model
{
    protected static string $table = 'projects';
}
```

`app/Controllers/ProjectController.php`:

```php
<?php

declare(strict_types=1);

namespace App\Controllers;

use App\Models\Project;
use Core\Controller;
use Core\Request;
use Core\Response;

class ProjectController extends Controller
{
    public function index(Request $request): Response
    {
        return $this->view('projects/index', ['projects' => Project::all()]);
    }
}
```

`views/projects/index.php`:

```php
<h1>Projekty</h1>

<ul>
    <?php foreach ($projects as $project): ?>
        <li><?= e($project->name) ?></li>
    <?php endforeach; ?>
</ul>
```

`routes.php`:

```php
$router->get('/projects', [ProjectController::class, 'index']);
```

Odpowiedź: w `core/` zero zmienionych linii.

**Na co patrzeć przy review:** w diffie nie może być nic w `core/`. Jeśli jest (np. warunek na nazwę tabeli w `Model`), to znaczy, że `static::$table` nie zostało zrozumiane i trzeba wrócić do kroku 7 [rozdziału 0](../../../ebook-dev-start/01-php-miniframework/rozdzialy/00-obiekty-w-php-zanim-zaczniemy.md). Zapomniany `use App\Controllers\ProjectController;` w `routes.php` daje „Class not found”. Model ma tylko nazwę tabeli, bez kopiowania `all()`.
