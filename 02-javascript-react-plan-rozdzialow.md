# Ebook 2: JavaScript, TypeScript i React. Plan rozdziałów

Stan: plan, bez tekstu. Ten sam schemat rozdziału co w ebooku PHP: Pojęcie, Co ma umieć, Kod, Sprawdź czy działa, Zadania, Częste błędy, Git. Każdy rozdział to osobna gałąź i PR, review tak jak dotąd.

Droga: trzy mini projekty w czystym JS, potem te same projekty utwardzone TypeScriptem, potem własny mini-React, potem kino w React. Każdy etap pracuje na kodzie z poprzedniego.

Założenia:

- Ebook jest samodzielny. Backend to gotowe repo `kino-api` (patrz niżej), więc może go wziąć uczeń, który chce tylko React. Kto przeszedł ebook 1, rozpozna w nim swój framework.
- Uczeń zna jakiś język programowania (u Kuby PHP), więc nie uczymy programowania od zera, tylko JavaScriptu „przez różnice” i przeglądarki jako platformy.
- Część I to trzy mini projekty, każdy na innym API przeglądarki, każdy z zadaniem „zrób drugą instancję”. Trzeci projekt ma zaboleć (stan rozsypany po kodzie), bo to uzasadnia część III.
- Część II to TypeScript na własnym kodzie z części I. Zero nowej funkcjonalności, tylko typowanie. Zadania w stylu „znajdź błędy, które TS złapał w Twoim kodzie”, nie „napisz typ”.
- Obiektówka: klasy najpierw, potem ten sam licznik jako domknięcie. Stan plus funkcje, które go zmieniają, w dwóch zapisach. To most do hooków. Bez klasowych komponentów Reacta.
- Od części III wszystko w TypeScript. Mini-React typowany lekko, tam gdzie typy zaciemniają mechanizm, mówimy wprost, że luzujemy. Kino w TS od pierwszego pliku.
- Zadania: po trzy na rozdział w częściach I i II, zawsze jedno „wyjaśnij własnymi słowami” obok „dopisz”. Odpowiedzi na pytania są warunkiem merge.
- Część IV to jedno kino rozbite na moduły, nie kilka osobnych aplikacji. Filmy z TMDB, seanse, sale i rezerwacje z `kino-api`.
- Narzędzia od pierwszego rozdziału: Node, Vite, ESLint, Prettier, Vitest. Tailwind przez plugin Vite od startu kina.

## Gotowy backend: repo `kino-api`

Osobne repo, napisane na mini-frameworku z ebooka 1, dostarczone uczniowi jako gotowe. Dla ucznia od Reacta to czarna skrzynka z README, dla ucznia po ebooku 1 to jego framework w prawdziwym użyciu.

- SQLite zamiast MySQL, start jednym poleceniem, seed z filmami (identyfikatory TMDB), seansami, salami, zajętymi miejscami i kontem testowym.
- Endpointy: lista seansów, seans z salą i zajętymi miejscami, blokada miejsca z czasem wygaśnięcia, rezerwacja, logowanie tokenem, historia biletów użytkownika. Walidacja zwraca błędy jako JSON. CORS dla `localhost` i domeny z `.env`.
- Endpointy według REST z rozdziału 20 ebooka 1: zasoby w liczbie mnogiej, `PATCH` do częściowych zmian, 201 z `Location`, 204 przy usuwaniu, 422 z `{"errors": {...}}`, 409 przy konflikcie miejsca. Uczeń od Reacta dostaje tę samą „gramatykę” API, którą zobaczy w pracy.
- README z opisem endpointów i przykładami odpowiedzi, tak żeby rozdział 10 mógł zacząć się od „wejdź na `/api/screenings` i zobacz JSON”. Do tego plik `kino-api.http` z gotowymi żądaniami (jak `api.http` w rozdziale 20 ebooka 1) i opis OpenAPI, żeby uczeń poznał oba sposoby dokumentowania API.
- Lokalnie u każdego ucznia, nie wspólny serwer. Wspólna instancja oznacza, że uczniowie nawzajem zajmują sobie miejsca.
- Dla tych po ebooku 1: dodatek „Jak zbudowano kino-api”, z blokadą miejsc po stronie serwera i tokenami jako dwoma rzeczami, których nie było w książce. Opcjonalny.

Ebook 1 nie wymaga zmian. Zależność między książkami znika, ebook 2 można zacząć po rozdziale 20 PHP albo w ogóle bez PHP.

## Część I. JavaScript przez trzy mini projekty (rozdziały 0–12)

### 0. JavaScript dla kogoś, kto zna inny język
Zmienne (`const`/`let`), typy i `===`, `undefined` vs `null`, funkcje i funkcje strzałkowe, tablice i obiekty jako literały, szablony `${}`, brak ścisłych typów i co z tego wynika (zapowiedź części II). Konsola przeglądarki jako REPL. Krótki rozdział, tabela „w PHP / w JS”.

### 1. Przygotowanie projektu
Node i npm (odpowiednik Composera), Vite, ESLint, Prettier, `.gitignore` z `node_modules`. Jedno repo `frontend`, każdy mini projekt w osobnym katalogu z własnym Vite. Uruchomienie `kino-api` lokalnie. Rytuał Git i szablon PR jak w PHP. Zadanie: Prettier ma formatować przy zapisie, ESLint ma krzyczeć o nieużytej zmiennej.

### Mini projekt 1: licznik odliczający (rozdziały 2–5)

### 2. DOM i zdarzenia
`querySelector`, `textContent`, `classList`, `addEventListener`, obiekt zdarzenia. Statyczny licznik z HTML i przyciskiem start. `Date` i różnica czasu w milisekundach. Zadanie: przycisk „reset”; wyjaśnij, czym różni się `textContent` od `innerHTML` i kiedy to drugie jest groźne.

### 3. Klasa `Countdown`
`class`, konstruktor, pola, metody, `this`. `setInterval`/`clearInterval`, metoda `render()`, która jako jedyna dotyka DOM. Zadanie: licznik w górę (stoper); drugi licznik na stronie przez `new Countdown(...)` bez zmiany klasy; wyjaśnij, dlaczego `render()` ma być jedynym miejscem zapisu do DOM.

### 4. Konfiguracja i wiele instancji
Opcje w konstruktorze (format, cel, element), callback `onFinish`, `data-*` w HTML, pętla tworząca licznik z każdego elementu. Pułapka: `this` w callbacku `setInterval`, funkcje strzałkowe jako rozwiązanie. Zadanie: stoper z międzyczasami; licznik, który po zakończeniu uruchamia następny.

### 5. Domknięcia i funkcje jako wartości
Ten sam licznik jako `createCountdown()`: funkcja zwraca obiekt z metodami, stan żyje w domknięciu, bez `this`. Porównanie obu wersji obok siebie. Funkcja przyjmująca funkcję. Zadanie: `createStopwatch()`; wyjaśnij, gdzie „mieszka” stan w wersji klasowej, a gdzie w wersji z domknięciem. Zapowiedź: `useState` to to drugie.

### Mini projekt 2: fiszki do nauki (rozdziały 6–9)

### 6. Moduły i praca na tablicach
`import`/`export`, podział na pliki (dane, logika, DOM). `map`, `filter`, `find`, `reduce`, `sort` na tablicy fiszek. Destrukturyzacja, spread, skrócony zapis obiektów. Zadanie: losowa kolejność bez mutowania oryginału; policz fiszki według kategorii przez `reduce`.

### 7. Formularze, klawiatura i localStorage
Formularz dodawania fiszki, `submit` i `preventDefault`, `FormData`. Zdarzenia klawiatury: spacja odwraca, strzałki przechodzą dalej. `localStorage` z `JSON.stringify`/`parse`. Zadanie: usuwanie fiszki; eksport i import do pliku JSON; wyjaśnij, czym localStorage różni się od sesji na serwerze i co w nim nie może leżeć.

### 8. Stan jako jedna wartość i czyste funkcje
Cały stan aplikacji w jednym obiekcie, funkcje `addCard(state, card)` zwracają nowy stan zamiast zmieniać stary, `render(state)` rysuje wszystko od zera. Niemutowalność, dlaczego kopiujemy tablice. To jest „React bez Reacta”. Zadanie: tryb nauki; statystyki; wyjaśnij, co się zepsuje, jeśli `addCard` zrobi `push` na starej tablicy.

### 9. Pierwsze testy: Vitest
Testy czystych funkcji z rozdziału 8, bez przeglądarki. `describe`, `it`, `expect`, tryb watch. Zadanie: test, który łapie mutację; test losowania z podstawionym generatorem.

### Mini projekt 3: repertuar na `kino-api` (rozdziały 10–12)

### 10. `fetch`, Promise i `async`/`await`
Pobranie `/api/screenings` z `kino-api`, JSON, kolejność wykonania (event loop na jednym przykładzie), `try`/`catch` przy `await`. CORS: dlaczego przeglądarka blokuje i jak backend na to odpowiada. Zadanie: wyświetl seanse pogrupowane po dniu; wyjaśnij, dlaczego `console.log` przed `await` wypisuje się wcześniej niż po.

### 11. Zapis na serwer i stany ekranu
Prosty wybór miejsca i `POST` blokady, nagłówki, `JSON.stringify`. Stany: ładowanie, błąd, pusto, dane. Odpowiedź z błędem walidacji z API. Zadanie: wyłącz przycisk podczas zapisu; pokaż komunikat błędu z API; wyjaśnij, co się dzieje, gdy użytkownik kliknie dwa razy.

### 12. Ból: stan w trzech miejscach
Miejsce jest w DOM, w tablicy w pamięci i w bazie. Refaktor do `render(state)` z rozdziału 8. Potem granica: pełny rerender gubi fokus, miga, przy dużej sali jest wolny. Rozdział kończy się pytaniem: jak rysować tylko to, co się zmieniło? Zadanie: zmierz czas renderu dla sali 1000 miejsc; opisz, które elementy naprawdę się zmieniły po kliknięciu jednego miejsca.

## Część II. TypeScript na własnym kodzie (rozdziały 13–15)

Trzy projekty z części I przechodzą na TS. Żadnej nowej funkcjonalności.

### 13. Typy na liczniku i fiszkach
Vite z TS, `tsconfig` ze `strict`, typy prostych wartości, `interface` dla fiszki i opcji licznika, typy parametrów i zwrotów, `null` i `undefined` pod `strict`. Edytor podkreśla błąd, zanim odświeżysz stronę. Odniesienie do `strict_types` z PHP. Zadanie: przenieś licznik i fiszki na TS; wypisz trzy błędy, które TS znalazł w Twoim kodzie, i wyjaśnij każdy.

### 14. Dane z zewnątrz
`fetch` zwraca `unknown`, nie to, co chcesz. `interface` odpowiedzi `kino-api`, rzutowanie a sprawdzanie, walidacja w czasie wykonania na granicy sieci. Najważniejsza lekcja modułu: typ to obietnica w kodzie, nie kontrola danych z sieci. Zadanie: zepsuj odpowiedź z API i pokaż, że typ tego nie łapie, a walidator tak; wyjaśnij różnicę.

### 15. Unie i generyki na stanie
Stan ekranu jako unia dyskryminowana: ładowanie, błąd, dane, i `switch`, który nie pozwala zapomnieć o przypadku. Generyk na przykładzie funkcji pobierającej dowolny typ z API. `type` vs `interface`, kiedy co. Zadanie: stan repertuaru jako unia; generyczna funkcja `loadJson<T>`; wyjaśnij, dlaczego `if (state.data)` jest gorsze niż `switch (state.status)`.

## Część III. Własny mini-React w TypeScript (rozdziały 16–20)

### 16. Drzewo elementów
Interfejs jako obiekt: `{ type, props, children }`, typ `VNode`. Funkcja `h()` zamiast JSX, komponent jako funkcja zwracająca drzewo. Licznik z rozdziału 3 opisany jako drzewo. Zadanie: drzewo dla listy fiszek; wyjaśnij, czym drzewo elementów różni się od DOM.

### 17. Render do DOM
Rekurencyjne tworzenie elementów, atrybuty, `on*` jako zdarzenia, tekst. Pierwszy działający `render(tree, container)`. Zadanie: obsługa `className` i `style`; komponent zagnieżdżony w komponencie.

### 18. `useState<T>` i rerender
Tablica stanów i indeks hooka, dlaczego `setX` zamiast przypisania, dlaczego hooki muszą być wywoływane w tej samej kolejności. Generyk z rozdziału 15 w praktyce. Licznik kliknięć, potem licznik odliczający jako komponent. Zadanie: dwa niezależne liczniki; wyjaśnij, co się stanie, gdy `useState` trafi do `if`.

### 19. `useEffect`
Efekt po renderze, tablica zależności, funkcja czyszcząca. `setInterval` w komponencie z poprawnym `clearInterval`. Pobranie seansów z API w efekcie, stan jako unia z rozdziału 15. Zadanie: licznik, który zatrzymuje się po odmontowaniu; wyjaśnij, dlaczego bez czyszczenia powstają dwa interwały.

### 20. Diff i `key`
Porównanie starego i nowego drzewa, aktualizacja tylko zmienionych węzłów, dlaczego lista bez `key` skacze i gubi stan. Sala z rozdziału 12 działa na mini-React bez mignięć. Mapa „Twój mini-React a React”: co masz, czego nie (Fiber, batching, Suspense). Zadanie: usuń `key` i pokaż błąd; dodaj `key` i wyjaśnij.

## Część IV. Kino w React (rozdziały 21–31)

### 21. Projekt kina
Vite z React i TS, JSX (to samo `h()`, tylko składnia), komponenty i typowane props, Tailwind przez plugin Vite. Ten sam `Countdown` w prawdziwym React, zero zmian w myśleniu. Struktura katalogów, ESLint z regułami hooków. Katalog `ui/` z `Button` i `Card` jako zalążek własnej biblioteki komponentów (warianty przez props, jedno miejsce na wygląd); dla porównania jeden komponent w CSS Modules i krótko, kiedy zespół wybiera Tailwind, a kiedy CSS Modules, oraz czym są gotowe biblioteki (MUI, shadcn/ui). Zadanie: komponent `MovieCard` ze statycznymi danymi, zbudowany z `Card` i `Button`; wyjaśnij, na co kompiluje się JSX.

### 22. Repertuar z TMDB i `kino-api`
Klucz TMDB w `.env` (`VITE_`), seanse z `kino-api`, plakaty i opisy z TMDB po identyfikatorze. Własny hook `useFetch<T>` ze stanem jako unią, lista filmów, wyszukiwanie z opóźnieniem (debounce). Zadanie: filtrowanie po dniu; wyjaśnij, dlaczego klucz w `.env` frontu i tak jest widoczny w przeglądarce i co z tym zrobić.

### 23. Routing
Krótko własny router na `history.pushState` i `popstate`, potem React Router: `/`, `/film/:id`, `/seans/:id/sala`. Parametry, linki, 404. Zadanie: strona „nie znaleziono”; przewijanie do góry przy zmianie strony.

### 24. Sala i wybór miejsc
Siatka miejsc z `kino-api` (rzędy, numery, zajęte), zaznaczanie, limit biletów, podnoszenie stanu do komponentu sali, komponenty kontrolowane. Najtrudniejszy stan w książce. Dostępność na prawdziwym przykładzie: miejsce to `<button>`, a nie `<div>`, opis dla czytnika ekranu („rząd 5, miejsce 12, wolne”), poruszanie się po sali strzałkami, fokus widoczny. Zadanie: różne ceny stref; miejsca dla niepełnosprawnych; wybierz dwa miejsca samą klawiaturą; wyjaśnij, dlaczego stan zaznaczonych nie może leżeć w pojedynczym `Seat`.

### 25. Rezerwacja z limitem czasu
`Countdown` z rozdziału 5 w React. Po zaznaczeniu miejsce blokowane w `kino-api` na kilka minut, wygaśnięcie po stronie serwera, bo timer w przeglądarce można oszukać. Konflikt: ktoś zajął miejsce w międzyczasie, 409 z API i odświeżenie sali. Pojęcie: skąd przeglądarka ma wiedzieć, że ktoś inny zajął miejsce. Odpytywanie co kilka sekund (polling) w kodzie, a obok, bez kodu, Server-Sent Events i WebSocket: kiedy który i dlaczego czat czy giełda nie odpytują. Zadanie: odliczanie widoczne na wszystkich stronach kroku zakupu; otwórz salę w dwóch oknach i pokaż, po ilu sekundach drugie okno widzi zajęte miejsce; wyjaśnij, dlaczego blokada musi żyć na serwerze.

### 26. Formularz zakupu i walidacja
Dane kupującego, podsumowanie, walidacja po stronie klienta i błędy z API jako JSON. Najpierw własny formularz ze stanem na każde pole, potem ten sam w React Hook Form ze schematem w Zod (walidacja w czasie wykonania z rozdziału 14, typ formularza wyprowadzony ze schematu). Komponent pola wielokrotnego użytku z typowanymi props. Zadanie: potwierdzenie z numerem rezerwacji; błąd 422 z API pokazany pod właściwym polem w React Hook Form; wyjaśnij, po co walidować dwa razy i co React Hook Form robi inaczej niż stan na każde pole (ile razy rysuje się formularz).

### 27. Logowanie tokenem
Logowanie do `kino-api`, przechowywanie tokenu, kontekst użytkownika, trasy chronione, historia biletów. Wylogowanie i wygaśnięcie tokenu. Zadanie: przekierowanie po zalogowaniu na stronę, z której przyszedł użytkownik; wyjaśnij różnicę między sesją z ciasteczkiem a tokenem.

### 28. Stan z serwera: TanStack Query
Ból z rozdziałów 22–27: własny `useFetch` pobiera repertuar od nowa przy każdym wejściu na stronę, po rezerwacji historia biletów pokazuje stare dane, a polling z rozdziału 25 jest napisany ręcznie. Pojęcie: stan z serwera (kopia danych, które należą do API) a stan aplikacji (co użytkownik właśnie zaznacza). `useQuery` zamiast `useFetch` (klucz zapytania, cache, `staleTime`), `useMutation` dla rezerwacji, unieważnienie (`invalidateQueries`) historii biletów po zakupie, `refetchInterval` zamiast ręcznego pollingu, optymistyczne zaznaczenie miejsca z cofnięciem przy 409. Devtools TanStack Query. Zadanie: przejście film → repertuar → film nie wysyła drugiego żądania; optymistyczne usunięcie biletu z historii z przywróceniem przy błędzie; wyjaśnij, które dane z kina są stanem z serwera, a które nie.

### 29. Stan globalny: Context i Redux Toolkit
Rezerwacja przechodząca między stronami: `useReducer` i Context, akcje jako unia dyskryminowana (rozdział 15 raz jeszcze). Granica: każdy komponent pod Contextem rysuje się przy każdej zmianie. Ten sam reducer przeniesiony do Redux Toolkit (`createSlice`, `configureStore`, `useSelector`), żeby było widać, że to ta sama idea z narzędziami: selektory, Redux DevTools z historią akcji. Krótko Zustand jako lżejsza alternatywa. Po rozdziale 28 w globalnym stanie zostaje mało, i to jest wniosek rozdziału. Zadanie: przeładowanie strony nie gubi rezerwacji (localStorage z rozdziału 7); w Redux DevTools cofnij się o dwie akcje; wyjaśnij, czym reducer różni się od `setState` i dlaczego listy filmów nie trzymasz w Reduksie.

### 30. Testy komponentów i ścieżki zakupu
Vitest z React Testing Library dla wyboru miejsc (także klawiaturą z rozdziału 24), Playwright dla pełnej ścieżki: film, sala, rezerwacja, zakup. Mockowanie TMDB i `kino-api` (MSW). Zadanie: test konfliktu miejsca; wyjaśnij, co testuje RTL, a czego Playwright.

### 31. Build, CI i wdrożenie
`npm run build`, co powstaje i dlaczego jest małe, zmienne środowiskowe produkcyjne, hosting frontu (Vercel, Netlify lub GitHub Pages), `kino-api` na Mikrusie (dla tych po ebooku 1) albo na dowolnym hostingu PHP, CORS na produkcji, HTTPS po obu stronach. GitHub Actions dla każdego PR: ESLint, `tsc --noEmit`, Vitest, Playwright; merge tylko na zielono. Lighthouse i Core Web Vitals (LCP, INP, CLS) na stronie repertuaru: plakaty z TMDB w odpowiednim rozmiarze, `loading="lazy"`, podział kodu na trasy (`React.lazy`). Zadanie: automatyczny deploy z `main`; popraw jeden wynik Lighthouse i pokaż przed/po w PR; wyjaśnij, dlaczego front i backend mogą stać na różnych serwerach.

## Część V. Praca w zespole i co dalej (rozdziały 32–33)

### 32. Kod od asystenta AI
Firmy w ofertach oczekują pracy z asystentem AI, a cały kurs uczy review. Uczeń zleca asystentowi funkcję w kinie (np. filtr seansów po gatunku z zapisem w adresie), dostaje kod i robi mu review tak, jak opiekun robił jemu: czy działa, czy przeklikane ścieżki, typy, stany ładowania i błędu, dostępność, testy. Typowe wpadki kodu z AI: nieistniejące API biblioteki, `any`, brak obsługi błędu, kod, który przechodzi testy napisane przez ten sam asystent. Zasada: odpowiadasz za każdą linijkę, którą wrzucasz, niezależnie od tego, kto ją napisał. Zadanie: PR z kodem od asystenta i z własnym review w opisie (co poprawiłeś i dlaczego); jeden test, który łapie błąd przeoczony przez asystenta; wyjaśnij, czego nie oddałbyś asystentowi bez sprawdzenia.

### 33. Mapa dalszej drogi
„Twój mini-React a React” raz jeszcze. Next.js: renderowanie na serwerze (SSR), statyczne strony (SSG) i renderowanie w przeglądarce (CSR) na przykładzie kina, App Router i Server Components, dlaczego repertuar jest dobrym kandydatem na SSR (SEO, rozdział 20 ebooka 1 o tym, kto buduje HTML). GraphQL jako inna odpowiedź na pytanie „jak pobierać dane”. React Native jako ten sam React na telefonie, TypeScript głębiej (typy warunkowe, `satisfies`). Co czytać, jakie projekty robić samemu.

## Do rozstrzygnięcia przed pisaniem

- `kino-api` trzeba napisać przed rozdziałem 10. Kto i kiedy: przed startem ebooka 2 w całości, czy endpointy dokładane przed rozdziałami, które ich potrzebują (10, 11, 25, 27).
- Klucz TMDB: własny Kuby czy wspólny; w rozdziale 22 i tak omawiamy, że jest widoczny.
- Czas: 34 rozdziały to więcej niż 23 w PHP. Przy tempie z PHP to około trzech miesięcy. Część II jest krótka, ale React zaczyna się dopiero w rozdziale 16, warto to powiedzieć uczniowi na starcie.
- WebSocket na prawdziwo: `php -S` obsługuje jedno żądanie naraz, a mini-framework nie trzyma połączeń, więc w rozdziale 25 jest polling i samo pojęcie. Jeśli chcemy kod, potrzebny jest osobny mały serwer (np. Node z biblioteką `ws`) obok `kino-api`, który rozgłasza „miejsce zajęte”. Do decyzji: dodatek po rozdziale 25 czy zostaje na pojęciu.
- Źródło zmian z 2026-10-29: przegląd 10 ofert z No Fluff Jobs (React, junior do senior). Najczęściej: React i TypeScript (9), REST API (8), testy (6), stan globalny/Redux (5), Next.js/SSR, wydajność, CI/CD, Vite, AI, biblioteki komponentów (po 4). Stąd rozdziały 28, 29, 32 i rozszerzenia 21, 24, 25, 26, 31. Next.js świadomie tylko w „Co dalej”: to osobny framework na osobną książkę.
