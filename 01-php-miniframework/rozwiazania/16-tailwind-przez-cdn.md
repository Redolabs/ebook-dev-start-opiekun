# Rozdział 16: Tailwind przez CDN

## Zadanie 1: ostylowanie pozostałych widoków

Przykład dla `views/tasks/show.php`:

```php
<div class="bg-white rounded-lg shadow p-6 space-y-2">
    <h1 class="text-2xl font-bold"><?= e($task->title) ?></h1>
    <p class="text-sm text-gray-600">Status: <?= $task->done ? 'zrobione' : 'do zrobienia' ?></p>
    <p class="text-sm text-gray-600">Dodano: <?= e($task->created_at) ?></p>
    <p><a href="/" class="text-blue-600 hover:underline">Wróć</a></p>
</div>
```

Przykład tabeli dla `views/admin/tasks.php`:

```php
<table class="w-full text-left bg-white rounded-lg shadow">
    <tr class="border-b">
        <th class="px-3 py-2">Id</th>
        <th class="px-3 py-2">Tytuł</th>
        <th class="px-3 py-2">Właściciel</th>
        <th class="px-3 py-2">Status</th>
        <th class="px-3 py-2"></th>
    </tr>
    <?php foreach ($tasks as $task): ?>
        <tr class="border-b">
            <td class="px-3 py-2"><?= e($task->id) ?></td>
            <td class="px-3 py-2"><?= e($task->title) ?></td>
            <td class="px-3 py-2"><?= e($task->owner_name) ?></td>
            <td class="px-3 py-2"><?= $task->done ? 'zrobione' : 'do zrobienia' ?></td>
            <td class="px-3 py-2">
                <form method="post" action="/admin/tasks/<?= e($task->id) ?>/delete">
                    <?= csrf_field() ?>
                    <button type="submit" class="text-sm text-red-600 hover:underline">Usuń</button>
                </form>
            </td>
        </tr>
    <?php endforeach; ?>
</table>
```

Pozostałe widoki (rejestracja, profil, strony błędów) to ten sam zestaw klas co formularz z kroku 3 i karta `bg-white rounded-lg shadow p-6`.

**Na co patrzeć przy review:** diff ma zmieniać tylko atrybuty `class` i układ znaczników. Każde `e()`, `csrf_field()` i warunek `isset($errors[...])` muszą zostać; przy przepisywaniu widoków najłatwiej je zgubić. Stara linijka `td, th { ... }` w `<style>` z [rozdziału 14](../../../ebook-dev-start/01-php-miniframework/rozdzialy/14-panel-admina-na-powaznie.md) powinna zniknąć razem z `<style>`. Pole z błędem dostaje `border-red-500`.

## Zadanie 2: odznaka „Zrobione”

W `views/tasks/index.php`, zamiast warunkowego `line-through` w linku:

```php
                <a href="/tasks/<?= e($task->id) ?>" class="flex-1"><?= e($task->title) ?></a>

                <?php if ($task->done): ?>
                    <span class="inline-block bg-green-100 text-green-800 text-xs px-2 py-1 rounded-full">Zrobione</span>
                <?php endif; ?>
```

**Na co patrzeć przy review:** klasa `line-through text-gray-400` ma zniknąć z linku, a odznaka ma być w tym samym `if`, który wcześniej ukrywał przycisk „Zrobione”, albo w osobnym, ale nie oba naraz dla tego samego zadania. To zadanie na 5 minut; jeśli zajęło dłużej, sprawdź, czy uczeń nie walczył z CDN-em, który nie załadował się bez internetu.

## Zadanie 3: tryb ciemny

Uwaga: opis „jednym krokiem” w książce nie działa. Tailwind 3 z `cdn.tailwindcss.com` domyślnie włącza warianty `dark:` według ustawień systemu (`prefers-color-scheme`), a nie według klasy `dark` na `<html>`. Sama klasa `dark` nic nie zmienia, a u ucznia z ciemnym motywem systemu strona będzie ciemna także bez niej. Trzeba przełączyć Tailwinda na tryb klasowy jedną linijką konfiguracji zaraz po skrypcie CDN:

```php
<html lang="pl" class="dark">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Lista zadań</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script>tailwind.config = { darkMode: 'class' }</script>
</head>
<body class="bg-gray-100 text-gray-900 dark:bg-gray-900 dark:text-gray-100">
    <nav class="bg-white shadow dark:bg-gray-800">
```

Dalej w widokach: karty `bg-white dark:bg-gray-800`, pola formularzy `dark:bg-gray-700 dark:border-gray-600`, szary tekst `text-gray-600 dark:text-gray-300`. W Tailwindzie 4 (`@tailwindcss/browser@4`) odpowiednikiem jest `<style type="text/tailwindcss">@custom-variant dark (&:where(.dark, .dark *));</style>`.

**Na co patrzeć przy review:** każde `bg-white` w widokach ma swój `dark:` odpowiednik; najłatwiej sprawdzić przez `grep -rn 'bg-white' views/ | grep -v 'dark:'`. Kontrast w trybie ciemnym (szary tekst na szarym tle). Jeśli uczeń mówi, że „u mnie działa bez konfiguracji”, ma najpewniej ciemny motyw w systemie: niech usunie klasę `dark` i zobaczy, że nic się nie zmieni.
