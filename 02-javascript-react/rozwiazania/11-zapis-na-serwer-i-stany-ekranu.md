# Rozdział 11: rozwiązania

## Zadanie 1: przycisk zajęty w trakcie zapisu

`view` dostaje pole `pending` (klucz miejsca albo `null`). W `main.js`:

```js
let view = { status: 'idle', screening: null, error: null, pending: null };

async function toggleSeat(row, seat) {
  if (view.pending !== null) {
    return;
  }
  const { screening } = view;
  const status = seatStatus(screening, row, seat);
  setView({ ...view, pending: `${row}-${seat}`, error: null });
  try {
    if (status === 'wolne') {
      await createHold(screening.id, row, seat);
    } else if (status === 'moje') {
      const hold = screening.seats.mine.find((s) => s.row === row && s.seat === seat);
      await deleteHold(hold.hold_id);
    }
    setView({ status: 'ready', screening: await getScreening(screening.id), error: null, pending: null });
  } catch (error) {
    if (error instanceof ApiError) {
      setView({
        status: 'ready',
        screening: await getScreening(screening.id),
        error: error.message,
        pending: null,
      });
    } else {
      console.error(error);
      setView({ ...view, status: 'error', error: 'Brak połączenia z kino-api.', pending: null });
    }
  }
}
```

W `ui.js` `renderHall(gridEl, screening, pending)` przekazuje `pending` do `createSeatElement`, które robi:

```js
const isPending = seatKey(row, seat) === pending;
button.classList.toggle('zapis', isPending);
button.disabled = isPending || status === 'sprzedane' || status === 'zajete';
```

Plus `.miejsce.zapis { opacity: 0.5; }`. Z throttlingiem w Network drugie kliknięcie nie wysyła nic.

**Na co patrzeć przy review.** Blokada przez stan i `render`, nie przez `button.disabled = true` z `main.js` (to byłoby pisanie do DOM-u poza `render`, czyli dokładnie to, od czego rozdział 8 odchodzi). `pending` czyszczone we **wszystkich** trzech wyjściach (sukces, `ApiError`, awaria); jeśli zostanie ustawione po błędzie, sala jest na stałe zablokowana. Wyjście na początku `toggleSeat`, gdy `pending !== null`, albo równoważnie `disabled` na wszystkich przyciskach podczas zapisu (akceptowalne, ale pierwsze jest prostsze).

## Zadanie 2: komunikat, który znika, i walidacja

Znikanie:

```js
function showNotice(message) {
  setView({ ...view, error: message });
  setTimeout(() => {
    if (view.error === message) {
      setView({ ...view, error: null });
    }
  }, 5000);
}
```

Warunek `view.error === message` pilnuje, żeby stary `setTimeout` nie skasował nowszego komunikatu. Alternatywa: pamiętać `timeoutId` i robić `clearTimeout` przed ustawieniem nowego; też dobra, a nawet lepsza, gdy dwa razy przyjdzie ten sam tekst.

Walidacja (`422`): w `catch`, zanim pokażesz komunikat:

```js
const message =
  error.status === 422 && error.body?.errors
    ? Object.values(error.body.errors).join(', ')
    : error.message;
```

Wywołanie testowe z konsoli po dopisaniu `window.createHold = createHold` na chwilę: `await createHold(11, 'a', 1)` daje `422` z `{ errors: { row: 'Pole row musi być liczbą całkowitą.' } }` (sprawdzone na `kino-api`). Można też to umieścić w `ApiError`: konstruktor składa komunikat z `errors`, gdy nie ma `error`; wtedy `error.message` jest zawsze gotowe dla użytkownika i to najlepsze miejsce.

**Na co patrzeć przy review.** Warunek przed skasowaniem komunikatu (albo `clearTimeout`); bez niego uczeń ma błąd, który pokazuje się tylko przy dwóch szybkich `409`. `Object.values`, nie `Object.keys`. Czy `422` nie jest traktowane jak awaria sieci.

## Zadanie 3: wyjaśnienia

1. **Dwa kliknięcia.** Wolne miejsce dwa razy: dwa `POST`, pierwszy `201` tworzy blokadę, drugi `200`, bo `kino-api` traktuje ponowną blokadę tego samego miejsca przez tego samego klienta jako odnowienie czasu; to nie błąd, serwer jest tu idempotentny. Potem dwa `GET`, oba rysują to samo. Swoje miejsce dwa razy: dwa `DELETE` z tym samym `hold_id`; pierwszy `204`, drugi `404` „Nie ma takiej blokady.”, bo blokada już nie istnieje. Użytkownik widzi przez chwilę komunikat o błędzie, choć zrobił to, co chciał. Zadanie 1 usuwa oba przypadki u źródła.
2. **Pobranie po `201`.** Odpowiedź z `POST` mówi tylko o Twojej blokadzie; nie mówi, co w międzyczasie zrobili inni (nowe `held`, nowe `taken`). Serwer jest jedynym źródłem prawdy o sali, lokalne dopisanie do `mine` to drugie źródło, które zacznie się rozjeżdżać. Kosztem jest jedno dodatkowe żądanie; rozdział 12 to zmierzy i zapyta, czy warto.
3. **`error` a `ready` z `error`.** `error` to „nie mam danych, nie mogę nic pokazać” (sala nie przyszła); `ready` z `error` to „mam dane, pokazuję je, a obok komunikat, że ostatnia operacja się nie udała”. W pierwszym użytkownik nie ma co klikać, w drugim może od razu wybrać inne miejsce. Jedno pole `error` nie wystarczy, bo nie mówi, czy sala jest aktualna; stąd `status`.

**Na co patrzeć przy review.** W odpowiedzi 1 oba statusy drugich żądań (`200` i `404`). W odpowiedzi 3 rozróżnienie „mam dane / nie mam danych”.

## Uwagi do całego rozdziału

Przed review uruchom `kino-api` i przeklikaj z dwoma oknami (zwykłe i prywatne). Sprawdź: `Content-Type` i `X-Client-Id` w `fetchJson`, a nie w każdej funkcji osobno; `getClientId` zapisuje do `localStorage` (odświeżenie nie zmienia koloru miejsc); `response.status === 204` przed `json()`; `instanceof ApiError` rozdziela odmowę od awarii; po każdej operacji ponowne `getScreening`. `view` z jawnym `status`, nie wnioskowanie ze `screening === null`. Pętle `for` w `renderHall` są celowe; nie wymagaj `map`.

Rozdział ma celowo zdublowany wzorzec: `view` z `setView` dla sali, a lista seansów dalej „ręcznie” w `loadScreenings`. Rozdział 12 to ujednolica i rozlicza koszty pełnego rerenderu. Nie prostuj tego w review.
