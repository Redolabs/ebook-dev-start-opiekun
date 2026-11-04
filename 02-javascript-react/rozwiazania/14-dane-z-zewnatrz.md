# Rozdział 14: rozwiązania

## Zadanie 1: zepsuta odpowiedź przez „Override content”

Chrome (od wersji 117) pozwala nadpisać treść odpowiedzi: Network, prawy przycisk na żądaniu `screenings`, „Override content”, wybór katalogu na nadpisania (dowolny pusty katalog poza repozytorium), potem edycja w panelu Sources i Ctrl+S. Firefox nie ma odpowiednika; uczeń na Firefoksie może zrobić to samo, zmieniając na chwilę `kino-api`: w `app/Controllers/Api/ScreeningController.php` w `index()` dopisać przed `return` linię rzutującą `price_cents` pierwszego seansu na tekst, a potem `git checkout -- app/`.

Oczekiwane obserwacje:

- **Z dekoderem**: konsola `DecodeError: seans[0]: pole price_cents ma być liczbą, jest string`, ekran „Nie udało się pobrać repertuaru”. Jedna linia mówi wszystko.
- **Z `as Screening[]`**: strona **działa**. Lista wygląda normalnie, cena pierwszego seansu to „32,00 zł”, bo `'3200' / 100` daje `32` (dzielenie konwertuje tekst na liczbę). Konsola pusta. Dopiero `'3200' + 100` w konsoli daje `'3200100'`: konkatenacja, bo plus z tekstem skleja. Kod, który by się wywalił: każde sumowanie cen (`total + screening.price_cents` w koszyku z rozdziału 26 dałoby tekst `'03200...'` zamiast liczby), każde porównanie `price_cents > 2000` (porównanie tekstu z liczbą działa, ale `'3200' > '900'` dwóch tekstów już nie, bo porównuje leksykograficznie), `toFixed` na tekście (`TypeError`).

To jest najważniejsza obserwacja rozdziału: rzutowanie nie tylko przesuwa błąd daleko od przyczyny, ale bywa, że **nie daje żadnego błędu**, tylko złe wyniki. Jeśli uczeń napisał „z `as` też się wywaliło”, to prawdopodobnie zepsuł inne pole (np. `starts_at`); poproś o powtórzenie z `price_cents`.

**Na co patrzeć przy review.** Rozróżnienie „wygląda na działającą” od „działa”. Przykład kodu, który by się wywalił, z uzasadnieniem (konkatenacja). Potwierdzenie, że nadpisanie zostało wyłączone (fioletowa ikona w Network); zdarza się, że uczeń zostawia je włączone i w następnym rozdziale widzi dziwne dane.

## Zadanie 2: dekoder dla `localStorage`

```ts
function isRecord(value: unknown): value is Record<string, unknown> {
  return typeof value === 'object' && value !== null;
}

function decodeCard(value: unknown, index: number): Card {
  if (!isRecord(value)) {
    throw new Error(`fiszka[${index}]: oczekiwano obiektu`);
  }
  const { id, front, back, category } = value;
  if (
    typeof id !== 'string' ||
    typeof front !== 'string' ||
    typeof back !== 'string' ||
    typeof category !== 'string'
  ) {
    throw new Error(`fiszka[${index}]: id, front, back i category mają być tekstami`);
  }
  return { id, front, back, category };
}

export function decodeCards(value: unknown): Card[] {
  if (!Array.isArray(value)) {
    throw new Error('fiszki: oczekiwano tablicy');
  }
  return value.map(decodeCard);
}

export function loadCards(fallback: Card[]): Card[] {
  try {
    const raw = localStorage.getItem(KEY);
    return raw === null ? fallback : decodeCards(JSON.parse(raw));
  } catch (error) {
    console.error('Nie udało się odczytać fiszek, używam domyślnych', error);
    return fallback;
  }
}
```

Sprawdzone: `tsc`, ESLint, trzy testy w `storage.test.ts` i jsdom z zepsutym `localStorage` (`[{ id: 'x', front: 'tylko front' }]`): osiem domyślnych fiszek, `console.error` z `fiszka[0]: id, front, back i category mają być tekstami`. Po destrukturyzacji `const { id, front, back, category } = value` każde pole ma typ `unknown`; po czterech `typeof` TypeScript zawęża je do `string` i literał `{ id, front, back, category }` pasuje do `Card` bez rzutowania. Uczeń, który dopisał `known?: boolean` w rozdziale 8, musi je obsłużyć: `known: typeof value.known === 'boolean' ? value.known : undefined` (albo pominąć pole, bo jest opcjonalne).

Wariant, który uczeń może wybrać: skopiować `decode.ts` z repertuaru razem z `number`/`string`/`record`. Poprawny, ale w fiszkach to przerost; zaakceptuj, zapytaj, czy czterdzieści linii narzędzi na jedno pole to dobry stosunek.

**Na co patrzeć przy review.** `value !== null` w `isRecord`. Dekodowanie **w `try`**, żeby `DecodeError`/`Error` z dekodera trafiał do istniejącego `catch` i `fallback`; częsty błąd to `decodeCards` wywołane poza `try`, przez co zepsute dane blokują całą aplikację. Testy czystej funkcji, nie `loadCards` (które potrzebuje `localStorage`; jeśli uczeń przetestował `loadCards` przez podstawienie `globalThis.localStorage`, też dobrze, ale zapytaj, czemu nie prościej).

## Zadanie 3: wyjaśnienia

1. **`as` a dekoder.** W czasie kompilacji oba dają `Screening[]`; różnica w tym, że `as` to deklaracja bez pokrycia, a dekoder ma typ zwracany wynikający z kodu, który coś sprawdził. W czasie wykonania `as` znika całkowicie (po wycięciu typów nie ma po nim śladu), dekoder wykonuje się i albo zwraca nowy obiekt, albo rzuca. Dobra odpowiedź używa słowa „wycięte” albo „nie istnieje w czasie wykonania”.
2. **Zatruty stan.** `setState` przypisuje `state = nextState`, a dopiero potem woła `render`. Gdy `render` rzuca, stan już jest podmieniony i zawiera dane, których nie da się narysować. `catch` w `loadScreenings` robi `setState(withList(state, { status: 'error', ... }))`, co kopiuje zepsute `screenings` do nowego stanu i `render` rzuca drugi raz, ale teraz **wewnątrz `catch`**, a `catch` nie jest w żadnym `try`, więc błąd leci do konsoli jako `Uncaught`. Uczeń może zaproponować poprawkę: `render` przed przypisaniem (wtedy stan zostaje stary) albo czyszczenie `screenings: []` w `catch`. Obie są słuszne i obie leczą objaw; przyczyną jest brak dekodera, i to ma być wniosek.
3. **Granice.** Sieć (`fetchJson`), `localStorage` (`loadCards`, `getClientId`), formularz (`FormData` w fiszkach), `dataset` (`Number(li.dataset.id)`, `Number(root.dataset.seconds)`), `catch (error)`. Niesprawdzona po tym rozdziale: `dataset` (`Number('abc')` to `NaN` i `openScreening(NaN)` wysłałoby `GET /screenings/NaN`, na co serwer odpowie `404`, więc objaw jest łagodny) oraz `getClientId` (tekst z `localStorage` przyjmowany bez sprawdzenia; ktoś może tam wpisać cokolwiek, ale to tylko jego problem). Każda sensowna lista z trzema pozycjami jest dobra.
4. **Dlaczego TS nie sprawdza w czasie wykonania.** Typy są wycinane przed uruchomieniem; w przeglądarce działa czysty JavaScript, który typów nie zna. PHP ma typy w samym silniku i sprawdza je przy każdym wywołaniu, co kosztuje czas wykonania i jest możliwe tylko dla typów, które silnik rozumie (prymitywy, klasy); PHP też nie sprawdzi kształtu tablicy asocjacyjnej z `json_decode`. TypeScript wybrał zero kosztu w czasie wykonania i pełną ekspresję typów (unie, literały, `Partial`), za cenę braku sprawdzeń. Plus za zauważenie, że `json_decode` w PHP ma dokładnie ten sam problem i `Validator` z ebooka 1 był jego dekoderem.

**Na co patrzeć przy review.** W odpowiedzi 2 kolejność `state = nextState` przed `render`. W odpowiedzi 4 słowo „wycięte” i analogia do `json_decode`.

## Uwagi do całego rozdziału

Przed review: `npm run check`, `npm run lint`, `npm test` w repertuarze (6 testów plus ewentualne własne) i w fiszkach (12 po zadaniu 2). Przeklikaj z dwoma oknami jak w rozdziale 11; dekoder dla `ScreeningWithSeats` łatwo zepsuć literówką w `seats_per_row` albo `hold_id`, a objaw to `DecodeError` przy otwieraniu sali.

Sprawdź `grep -rn " as " src/` w repertuarze: ma być pusto. Jedyne akceptowalne rzutowanie w całej części II to `as State` w teście `deepFreeze` z rozdziału 13.

`body: unknown` w `fetchJson` to jedna adnotacja, od której zależy cały rozdział; bez niej wszystko kompiluje się i działa, a dekodery są ozdobą. Sprawdź, że jest. Sprawdź też `interface RequestOptions` zamiast `RequestInit`: uczeń, który użył `RequestInit`, dostał błąd przy spreadzie `options.headers` i mógł go obejść przez `as Record<string, string>`; to właśnie ten typ obejścia, które rozdział odradza.

Czego nie wymagać: dekodowania wszystkich pól, które serwer zwraca; `where` w każdym komunikacie (wystarczy nazwa pola); osobnego `DecodeError` w fiszkach (zwykły `Error` jest w porządku). Czego wymagać: dekoder buduje nowy obiekt, nie zwraca wejścia po sprawdzeniu dwóch pól; `isRecord` jako predykat (`value is ...`), nie `: boolean`; testy dekoderów na danych zepsutych, nie tylko poprawnych.

Jeśli uczeń pyta, czemu nie Zod od razu: bo w rozdziale 26 ma rozumieć, co Zod robi za niego, i bo w kodzie, który dostanie w pracy, ręczne dekodery są częstsze niż się wydaje. Jeśli użył Zod sam, nie blokuj; poproś o jeden dekoder ręczny obok, żeby zadanie było zrobione.
