# Rozdział 4: Router

## Zadanie 1: trasa /hello z parametrem z adresu

Do `HomeController`:

```php
    public function hello(Request $request): Response
    {
        return new Response('<h1>Cześć, ' . $request->input('name', 'nieznajomy') . '!</h1>');
    }
```

Do `routes.php`:

```php
$router->get('/hello', [HomeController::class, 'hello']);
```

`/hello?name=Jakub` pokazuje „Cześć, Jakub!”, a `/hello?name=<b>Jakub</b>` pokazuje pogrubione imię, bo tekst od użytkownika wszedł do HTML-a bez obróbki. To celowy pokaz XSS, który [rozdział 5](../../../ebook-dev-start/01-php-miniframework/rozdzialy/05-widoki-layout-i-kontroler-bazowy.md) naprawia funkcją `e()`. Książka używa query stringa, a nie `/hello/{name}`, bo `</b>` ma ukośnik (`[^/]+` dałoby 404), a `Request::path()` nie dekoduje `%3C`, podczas gdy `$_GET` jest dekodowane przez PHP.

**Na co patrzeć przy review:** sygnatura metody ma drugi parametr `string $name`; brak go daje `ArgumentCountError`. Na tym etapie brak `e()` jest zamierzony, [rozdział 5](../../../ebook-dev-start/01-php-miniframework/rozdzialy/05-widoki-layout-i-kontroler-bazowy.md) to naprawia. Sprawdź, czy uczeń nie dopisał do routera specjalnej obsługi tej jednej trasy.

## Zadanie 2: dwa parametry

```php
    public function comment(Request $request, string $id, string $commentId): Response
    {
        return new Response('Zadanie ' . $id . ', komentarz ' . $commentId);
    }
```

```php
$router->get('/tasks/{id}/comments/{commentId}', [HomeController::class, 'comment']);
```

`/tasks/5/comments/12` daje „Zadanie 5, komentarz 12”. Argumenty przychodzą w kolejności, w jakiej parametry stoją w adresie, a nie według nazw: router robi `...array_values($params)`, więc nazwy w sygnaturze metody nie mają znaczenia. Zamiana parametrów w sygnaturze na `(Request $request, string $commentId, string $id)` zamienia wartości miejscami.

**Na co patrzeć przy review:** uczeń ma umieć powiedzieć, dlaczego kolejność jest „z adresu”: `preg_match` wypełnia grupy po kolei, `array_values` gubi nazwy. Jeśli twierdzi, że PHP dopasowuje po nazwie, dopytaj, bo w PHP 8 istnieją argumenty nazwane i łatwo to pomylić (`...` na tablicy z kluczami tekstowymi dałoby argumenty nazwane, ale `array_values` je usuwa).

## Zadanie 3: kolejność /tasks/create i /tasks/{id}

Bez kodu. Router sprawdza trasy w kolejności rejestracji i bierze pierwszą pasującą. Wzorzec `#^/tasks/(?P<id>[^/]+)$#` pasuje do `/tasks/create`, bo `create` to „jeden lub więcej znaków bez ukośnika”, więc wywołuje się `show()` z `$id = 'create'`. Trasa `/tasks/create` zarejestrowana później nigdy nie dostaje szansy.

**Na co patrzeć przy review:** dobra odpowiedź wymienia oba elementy: „pierwsza pasująca wygrywa” i „`{id}` łapie dowolny tekst, nie tylko liczby”. Jeśli uczeń proponuje naprawę przez zmianę wzorca na `[0-9]+`, to też poprawne i ciekawe (Laravel ma do tego `where('id', '[0-9]+')`), ale zasada z książki to kolejność rejestracji.
