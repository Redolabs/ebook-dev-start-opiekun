# Rozdział 10: rozwiązania

## Zadanie 1: seanse pogrupowane po dniu

`src/screenings.js`:

```js
export function groupByDay(screenings) {
  return screenings.reduce((groups, screening) => {
    const day = screening.starts_at.slice(0, 10);
    groups[day] = [...(groups[day] ?? []), screening];
    return groups;
  }, {});
}
```

Wersja z `push` na `groups[day]` jest tu w porządku (akumulator jest lokalny, nikt go nie współdzieli), ale po rozdziale 8 uczeń może odruchowo unikać mutacji; obie wersje akceptuj. Rysowanie w `ui.js`:

```js
export function renderScreeningsByDay(rootEl, groups) {
  const sections = Object.entries(groups).map(([day, screenings]) => {
    const section = document.createElement('section');
    const heading = document.createElement('h2');
    heading.textContent = formatDay(`${day} 00:00:00`);
    const list = document.createElement('ul');
    list.replaceChildren(...screenings.map((screening) => createScreeningElement(screening)));
    section.append(heading, list);
    return section;
  });
  rootEl.replaceChildren(...sections);
}
```

Klucze obiektu w formacie `RRRR-MM-DD` sortują się leksykograficznie tak samo jak chronologicznie, a API zwraca seanse już posortowane, więc kolejność dni jest dobra bez dodatkowego `sort`. Test:

```js
import { describe, it, expect } from 'vitest';
import { groupByDay } from './screenings.js';

describe('groupByDay', () => {
  it('grupuje po dacie z starts_at', () => {
    const groups = groupByDay([
      { id: 1, starts_at: '2026-11-03 18:00:00' },
      { id: 2, starts_at: '2026-11-04 15:30:00' },
      { id: 3, starts_at: '2026-11-03 20:45:00' },
    ]);
    expect(Object.keys(groups)).toEqual(['2026-11-03', '2026-11-04']);
    expect(groups['2026-11-03'].map((s) => s.id)).toEqual([1, 3]);
  });

  it('dla pustej tablicy daje pusty obiekt', () => {
    expect(groupByDay([])).toEqual({});
  });
});
```

Sprawdzone na danych z `kino-api`: 6 dni, pierwszy dzień ma mniej seansów (te, które już minęły, API pomija).

**Na co patrzeć przy review.** `groupByDay` w osobnym module bez DOM-u i z testem; jeśli grupowanie jest wplecione w `renderScreenings`, poproś o wyciągnięcie. Klucz z `slice(0, 10)`, nie z `formatDay` (sformatowany tekst nie sortuje się i zależy od lokalizacji). Zmienna `#seanse` z `<ul>` musiała zmienić się na kontener (`<div>`), bo `<h2>` wewnątrz `<ul>` to niepoprawny HTML.

## Zadanie 2: filtr po filmie

`api.js`:

```js
export function getMovies() {
  return fetchJson('/movies');
}

export function getScreenings(movieId = null) {
  const query = movieId === null ? '' : `?movie_id=${movieId}`;
  return fetchJson(`/screenings${query}`);
}
```

`main.js`:

```js
async function init() {
  messageEl.textContent = 'Ładowanie…';
  try {
    const [movies, screenings] = await Promise.all([getMovies(), getScreenings()]);
    renderMovieOptions(selectEl, movies);
    show(screenings);
  } catch (error) {
    console.error(error);
    messageEl.textContent = 'Nie udało się pobrać repertuaru. Czy kino-api działa?';
  }
}

selectEl.addEventListener('change', async () => {
  const movieId = selectEl.value === '' ? null : Number(selectEl.value);
  show(await getScreenings(movieId));
});
```

Lepsze rozwiązanie używa `URLSearchParams` zamiast sklejania (`new URLSearchParams({ movie_id: movieId })`), ale książka tego jeszcze nie wprowadziła; pochwal, jeśli uczeń znalazł sam. Przy dwóch osobnych `await` czas ładowania to suma obu żądań; z `Promise.all` maksimum z nich. W Network widać to jako dwa paski zaczynające się w tym samym momencie kontra jeden za drugim.

**Na co patrzeć przy review.** `Promise.all` z destrukturyzacją tablicy. Obsługa błędu w handlerze `change`: w rozwiązaniu wyżej jej nie ma i to jest luka (`Uncaught (in promise)`), którą warto wskazać w komentarzu bez Request changes; rozdział 11 robi to porządnie. Czy opcja „wszystkie filmy” wraca do pełnej listy.

## Zadanie 3: wyjaśnienia

1. **Kolejność.** `console.log` przed `await` wykonuje się synchronicznie, przy wywołaniu funkcji. Przy `await` funkcja oddaje sterowanie i zwraca Promise, więc kod **za wywołaniem** funkcji `async` idzie dalej natychmiast; dopiero gdy Promise z `await` się rozstrzygnie i bieżący kod się skończy, funkcja dokończa resztę. Dobra odpowiedź używa słów „oddaje sterowanie” lub „kolejka” albo „pętla zdarzeń”.
2. **404 i `fetch`.** `fetch` zgłasza błąd tylko, gdy nie dostał odpowiedzi (sieć, CORS). Odpowiedź z dowolnym statusem to sukces Promise. Bez `if (!response.ok)` `body` byłby obiektem `{ error: ... }`, a `renderScreenings` dostałoby obiekt zamiast tablicy: `screenings.map is not a function` w konsoli i pusta strona bez komunikatu dla użytkownika.
3. **CORS.** Chroni użytkownika przeglądarki przed stronami, które chciałyby czytać odpowiedzi z innych serwisów w jego imieniu (z jego ciasteczkami). Żądanie dochodzi do serwera i jest wykonywane (log pokazuje `200`); blokowany jest tylko odczyt odpowiedzi w przeglądarce, a `curl` lub inny serwer nie mają tego ograniczenia. Dlatego autoryzacja musi być po stronie serwera, a CORS to tylko ustawienie, komu przeglądarka pokaże wynik.

**Na co patrzeć przy review.** W odpowiedzi 1 rozróżnienie „przed `await`” od „za wywołaniem”. W odpowiedzi 3 zdanie, że żądanie **dochodzi** do serwera.

## Uwagi do całego rozdziału

Nowy projekt `repertuar/`: sprawdź ESLint i Prettier (częsty brak `eslint.config.js` w nowym katalogu, lint „przechodzi”, bo nie ma czego lintować: `npm run lint` wypisze wtedy błąd o braku konfiguracji). `BASE_URL` w jednym miejscu. Każde `await` w `try`/`catch` albo w funkcji, której wywołanie ma `catch`. `console.error(error)` obok komunikatu dla użytkownika, nie zamiast. Komunikat dla użytkownika po polsku, nie `error.message`. Jeśli uczeń użył `.then` zamiast `async`/`await`: poprawne, ale poproś o jedną wersję w całym projekcie.
