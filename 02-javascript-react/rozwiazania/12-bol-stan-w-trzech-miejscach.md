# Rozdział 12: rozwiązania

## Zadanie 1: pomiar z podziałem

W `ui.js`:

```js
export function bigHall(screening) {
  return { ...screening, hall: { ...screening.hall, rows: 25, seats_per_row: 40 } };
}

export function render(state, elements) {
  const t0 = performance.now();
  renderList(elements.list, state.list, state.hall.id);
  const t1 = performance.now();
  renderHallView(elements.hall, state.hall);
  const t2 = performance.now();
  console.log(`lista: ${(t1 - t0).toFixed(1)} ms, sala: ${(t2 - t1).toFixed(1)} ms`);
}
```

i na czas pomiaru w `renderHallView`: `renderHall(elements.grid, bigHall(hall.screening))`. Wyniki z mojego komputera (Chrome, headless, mediana z 20 powtórzeń; `performance.now()` mierzy tylko JavaScript, układ i malowanie są osobno):

| Wariant | Lista (57 seansów) | Sala |
| --- | --- | --- |
| 60 miejsc, bez spowolnienia | 4,8 ms | 0,1 ms |
| 1000 miejsc, bez spowolnienia | 4,8 ms | 1,6 ms (+3 ms układ) |
| 60 miejsc, CPU 4× | 19,8 ms | 0,4 ms |
| 1000 miejsc, CPU 4× | 19,8 ms | 7,4 ms (+14 ms układ) |

Liczby u ucznia będą inne (sprzęt, przeglądarka, liczba seansów w bazie zależna od godziny), ale proporcja powinna się powtórzyć: lista droższa od sali, nawet tysiącmiejscowej. Powód: `toLocaleDateString` i `toLocaleTimeString` dla każdego seansu. Jeśli uczeń doszedł do tego sam, to bardzo dobry znak; jeśli nie, zapytaj w review, co w `createScreeningElement` może być drogie, i podpowiedz, żeby zmierzył `formatDay` w pętli 1000 razy w konsoli.

**Na co patrzeć przy review.** `bigHall` kopiuje przez spread, nie modyfikuje seansu ze stanu. Tabela w PR ma cztery wiersze i obie kolumny. Na końcu `bigHall` i logi mają zniknąć z kodu (sprawdź diff gałęzi z `main`: w finalnym `ui.js` nie ma `performance`). Jeśli uczeń zmierzył tylko `setState` łącznie, poproś o podział; to jest sedno zadania.

## Zadanie 2: `patchHall`

```js
export function patchHall(gridEl, previous, screening) {
  const sameHall =
    previous !== null &&
    previous.hall.rows === screening.hall.rows &&
    previous.hall.seats_per_row === screening.hall.seats_per_row &&
    gridEl.children.length > 0;
  if (!sameHall) {
    renderHall(gridEl, screening);
    return 0;
  }
  let changed = 0;
  for (const button of gridEl.querySelectorAll('.miejsce')) {
    const row = Number(button.dataset.row);
    const seat = Number(button.dataset.seat);
    const status = seatStatus(screening, row, seat);
    if (seatStatus(previous, row, seat) !== status) {
      button.className = `miejsce ${status}`;
      button.disabled = status === 'sprzedane' || status === 'zajete';
      changed++;
    }
  }
  return changed;
}
```

`renderHallView(elements, hall, previous = null)` wywołuje `patchHall(elements.grid, previous, hall.screening)` zamiast `renderHall`, `render(state, elements, previousState = null)` przekazuje `previousState?.hall.screening ?? null`, a `setState` zapamiętuje `const previousState = state` przed podmianą. Sprawdzone w jsdom na żywym `kino-api`: po kliknięciu `document.activeElement` to nadal ten sam przycisk (ten sam obiekt, nie nowy o tym samym `data-key`), po zwykłym kliknięciu `zmienione miejsca: 1`, po `409` z drugiego klienta też `1` (miejsce `wolne` → `zajete`; więcej tylko, gdy w międzyczasie ktoś inny zmienił coś jeszcze).

Uwaga na przypadek, który łatwo przeoczyć: `previous` ma być **poprzednim seansem w stanie**, nie poprzednią odpowiedzią serwera. Gdy użytkownik przechodzi z Sali A do Sali B o tych samych wymiarach, `patchHall` zaktualizuje przyciski, a nie przerysuje: to poprawne, bo przyciski są te same, a `seatStatus` dla nowego seansu daje nowe statusy. Przy zmianie seansu `openScreening` ustawia najpierw `screening: null` (stan `loading`), więc `previous` jest `null` i i tak idzie pełne rysowanie.

**Na co patrzeć przy review.** Porównanie statusów przez `seatStatus` dla obu seansów, nie przez czytanie klasy z DOM-u (DOM nie jest źródłem, to lekcja rozdziału). Warunek na wymiary i na pustą siatkę. `return 0` w gałęzi pełnego rysowania (albo liczba wszystkich miejsc; obie wersje akceptuj, ale ma być spójne z logiem). Test fokusu zrobiony klawiaturą; przy myszy fokus i tak wraca na `body`, więc „działa” niczego nie dowodzi (tabela błędów). Jeśli uczeń zrobił to samo dla listy, to ponad zadanie; zobacz, czy obsłużył zmianę liczby seansów (zadanie 3 pyta właśnie o to).

## Zadanie 3: wyjaśnienia

1. **Trzy kopie.** Baza `kino-api` (wiersz blokady), `state.hall.screening.seats.mine` w pamięci, przycisk z klasą `moje` w DOM. Prawdą jest baza, bo tam wygasa blokada i tam trafiają zapisy innych klientów; pozostałe dwie powstały w chwili ostatniego `GET` i od tej chwili się starzeją. Przeglądarka nie dostaje powiadomień o zmianach na serwerze, dopóki sama nie zapyta (rozdział 25: odpytywanie, SSE, WebSocket). Dobra odpowiedź rozróżnia dwa problemy: stan ↔ serwer (nie do rozwiązania bez sieci) i stan ↔ DOM (rozwiązany przez `render(state)`).
2. **Poprawny, ale rozrzutny.** Poprawny, bo ekran jest zawsze funkcją stanu i nie ma ścieżki, na której DOM pokazuje coś, czego nie ma w `state`. Ginie wszystko, co przeglądarka trzyma w elemencie: fokus, zaznaczenie tekstu, pozycja przewinięcia wewnątrz elementu, stan `:hover` i podpowiedź `title`, trwająca animacja lub przejście CSS, tekst wpisany w `<input>`, który nie jest kontrolowany ze stanu. Trzy dowolne z tej listy wystarczą; plus za `<input>`, bo to przypadek, który w prawdziwej aplikacji boli najbardziej (i który React rozwiązuje komponentami kontrolowanymi, rozdział 24).
3. **Dlaczego lista jest trudniejsza.** `patchHall` działa, bo siatka ma stałe wymiary i każdy przycisk ma stałą tożsamość `rząd-miejsce`: stary przycisk `3-7` i nowy `3-7` to „to samo”. Lista nie ma stałej długości ani pozycji. Gdy seans zniknie ze środka, porównanie „po indeksie” zestawi stary czwarty `li` z nowym czwartym, który jest innym seansem, i zaktualizuje wszystkie `li` od tego miejsca w dół, choć zmienił się jeden. Gdy nowy seans pojawi się na początku, przesuną się wszystkie. Brakuje **klucza tożsamości** (tu `screening.id`, w sali `data-key`) i algorytmu, który dopasowuje stare do nowych po kluczu, a nie po pozycji, oraz dodaje i usuwa brakujące. To jest `key` z rozdziału 20. Jeśli uczeń napisze „trzeba porównać po `id`”, to wystarczy.

**Na co patrzeć przy review.** W odpowiedzi 1 słowo „baza” albo „serwer” jako prawda i rozróżnienie dwóch problemów synchronizacji. W odpowiedzi 3 przykład z przesunięciem elementów i wniosek o kluczu.

## Uwagi do całego rozdziału

Rozdział nie dodaje funkcji, więc review jest głównie o strukturze. Sprawdź `grep -n "textContent\|hidden\|className\|replaceChildren" src/main.js`: ma nic nie zwrócić. `state.js` bez importów z DOM-u. `withList`/`withHall` z podwójnym spreadem (częsty błąd: `{ ...state, hall: changes }`, który kasuje pola niepodane w `changes`; objawem jest `undefined` w `status` po `409`). Stan początkowy `list.status: 'loading'` i `render` przed `loadScreenings`.

Pomiary: nie oceniaj liczb, oceniaj metodę (podział, powtórzenie, spowolnienie CPU, tabela). Jeśli uczeń ma inną przeglądarkę niż Chrome, panel Performance wygląda inaczej, ale `performance.now()` działa wszędzie.

`console.log` z pomiarem jest w kodzie rozdziału celowo i ma zniknąć w zadaniu 1; jeśli został w finalnym PR, zwróć uwagę, ale nie blokuj.

Gałąź kończy część I. Następne trzy rozdziały to TypeScript na tym samym kodzie, bez nowych funkcji; przy review rozdziału 13 wrócisz do `state.js` z tego rozdziału, bo to on dostanie typy jako pierwszy w rozdziale 14.
