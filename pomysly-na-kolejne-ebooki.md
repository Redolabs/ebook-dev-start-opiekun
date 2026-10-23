# Pomysły na kolejne ebooki i dodatki

Notatki na później. Ten sam schemat co w „Mini-framework PHP krok po kroku”: uczeń buduje sam mini-wersję tego, czego używa framework, a na końcu dostaje mapę na prawdziwy framework. Każdy rozdział: Pojęcie, Co ma umieć, Kod, Sprawdź czy działa, Zadanie, Częste błędy, Git.

## Proponowana kolejność

1. PHP (gotowe, 23 rozdziały: 0-22, w tym wdrożenie na VPS).
2. React, bo rozdział 20 ebooka PHP już go zaczyna, a API w PHP jest gotowe jako backend.
3. Python, jako „ten sam pomysł w drugim języku”, co najlepiej utrwala, że framework to nie magia.

## Dodatek C do ebooka PHP: Git, gdy pracujesz w zespole

Czego celowo nie ma w rozdziale 1 i co warto dopisać, gdy Kuba zacznie pracować z kimś na jednym repo:

- konflikt wywołany celowo (dwie gałęzie zmieniają tę samą linię) i rozwiązanie go krok po kroku,
- `git stash`, gdy trzeba przerwać pracę i przeskoczyć na inną gałąź,
- `git rebase` vs `merge` (kiedy co, i zasada: nie przepisuj historii, którą ktoś już pobrał),
- `cherry-pick`,
- tagi i wydania (`v1.0.0`, GitHub Releases),
- code review z drugiej strony: jak komentować cudzy PR,
- `git bisect` do szukania commita, który coś zepsuł.

## Ebook 2: JavaScript, własny React i aplikacja w React

Cel: ta sama droga co w PHP. Najpierw ból (lista zadań w czystym JS, stan rozsypany po kodzie, DOM aktualizowany ręcznie), potem narzędzie, które go leczy (własny mini-React), potem prawdziwy React. Backend to API z ebooka PHP (`/api/tasks`), więc Kuba od pierwszego dnia pracuje na prawdziwych danych.

Zasady:

- JavaScript uczony przez projekt, nie przez listę składni. Każdy rozdział dokłada kawałek działającej aplikacji.
- Obiektowość z dystansem: klasy, `this`, prototypy i moduły trzeba znać, bo są w cudzym kodzie, ale React jest funkcyjny. Więcej czasu na domknięcia, funkcje jako wartości, `map`/`filter`/`reduce`, destrukturyzację, spread, niemutowalność. Bez klasowych komponentów Reacta.
- Dobre praktyki jako narzędzia: ESLint i Prettier od pierwszego rozdziału, małe i czyste funkcje, nazwy, brak globalnych zmiennych, rytuał Git z ebooka PHP.
- Mini-React ograniczony do sedna (5 rozdziałów), cel to zrozumienie, nie odtworzenie Reacta.

Szkic:

| Część | Rozdziały | Efekt |
| --- | --- | --- |
| I. JavaScript przez projekt | 0–5 | Projekt z Vite, ESLint, Prettier, Git; lista zadań w czystym JS na API z PHP (DOM, zdarzenia, `fetch`, formularz); moduły `import`/`export`; domknięcia i funkcje jako wartości; tablice i obiekty; `async`/`await`; obiekty, klasy i `this` z dystansem |
| II. Dobre praktyki | 6–7 | Refaktor własnego kodu: małe funkcje, czyste funkcje, niemutowalność, podział na moduły; pierwsze testy w Vitest |
| III. Własny React | 8–12 | `createElement` i drzewo elementów; render do DOM; `useState` i rerender (dlaczego `setX` zamiast przypisania); `useEffect`; prosty diff i po co `key` |
| IV. Prawdziwy React | 13–22 | Ten sam komponent w prawdziwym React (zero zmian w myśleniu); komponenty, props, podnoszenie stanu; formularze; router (najpierw własny na `history.pushState`, potem React Router); dane z API, stany ładowania i błędu, własny `useFetch`; logowanie tokenem (Sanctum-like z ebooka PHP, rozdział 21); kontekst i globalny stan; Tailwind z Vite (prawdziwy build zamiast CDN); React Testing Library i Playwright; build i wdrożenie (Vercel/Netlify/GitHub Pages) razem z backendem PHP |
| V. Co dalej | 23–24 | TypeScript (po `strict_types` w PHP Kuba od razu rozumie, po co); Next.js; mapa „Twój mini-React a React” |

## Ebook 3: Mini-framework webowy w Pythonie

Cel: ta sama aplikacja „lista zadań” napisana w Pythonie na własnym mini-frameworku, a na końcu mapa na Flaska, FastAPI i Django.

Szkic rozdziałów:

0. Python, którego będziesz potrzebować: typy, funkcje, klasy, `self`, dziedziczenie, wyjątki, listy i słowniki, comprehensions, moduły, środowisko wirtualne (`venv`), `pip`.
1. Projekt: `venv`, `pip`, `pyproject.toml`, Git.
2. WSGI: Request i Response (odpowiednik rozdziałów 2 i 3 z PHP).
3. Router z dekoratorami: `@app.get('/tasks/{id}')`, czyli dekorator jako odpowiednik metod magicznych PHP (funkcja, która dostaje funkcję).
4. Szablony: własny mini-silnik na `str.format`, potem Jinja2.
5. Baza: `sqlite3` z `?`, potem mini-ORM z `__getattr__`, `__setattr__`, `classmethod`, `dataclass` (odpowiednik rozdziału 9 o metodach magicznych).
6. Pełna aplikacja CRUD.
7. Sesje, hasła (`bcrypt` lub `hashlib.scrypt`), CSRF.
8. Walidacja: własny walidator, potem Pydantic.
9. Middleware jako funkcje opakowujące (closures i dekoratory jeszcze raz).
10. Rejestracja, profil, role, panel admina.
11. Obsługa błędów, logowanie (`logging`), tryb debug.
12. Testy: `pytest`, fixtures, klient testowy, e2e, Playwright dla Pythona.
13. API JSON i React z ebooka 2 jako frontend.
14. Zewnętrzne API: `httpx`/`requests`, e-mail przez Resend, asynchroniczność (`async`/`await`) jako zapowiedź FastAPI.
15. Co dalej: Flask (najbliższy), FastAPI (typy, async, dokumentacja z kodu), Django (wszystko w pudełku, ORM, admin); mapa „Twój kod a Flask/FastAPI/Django”.

Pythonowe „magiczne” odpowiedniki do wytłumaczenia na prawdziwym kodzie: dunder methods (`__init__`, `__getattr__`, `__repr__`, `__iter__`, `__enter__`/`__exit__`), dekoratory, context managery, generatory, `@property`, `@classmethod`/`@staticmethod`.

## Inne drobne pomysły

- Wdrożenie: zrobione, to rozdział 21 ebooka PHP (Mikrus jako główna ścieżka, klasyczny VPS w ramce z różnicami). Do rozważenia później: Playwright w CI, monitoring (UptimeRobot) jako osobny mini-rozdział.
- Docker dla ebooka PHP: `docker compose` z PHP i MySQL, żeby projekt dało się uruchomić na każdym komputerze jednym poleceniem.
- Krótki ebook „SQL od zera dla webdevelopera”: JOIN, GROUP BY, indeksy, EXPLAIN, transakcje, na bazie z listy zadań.
