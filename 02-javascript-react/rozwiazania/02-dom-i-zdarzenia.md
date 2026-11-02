# Rozdział 2: rozwiązania

## Zadanie 1 i 2: tytuł karty

Oba zadania to zmiana wyłącznie w `render()`, bo tytuł karty jest częścią tego, co widać:

```js
function render(remainingSeconds) {
  const time = formatTime(remainingSeconds);
  czasEl.textContent = time;
  czasEl.classList.toggle('koniec', remainingSeconds === 0);
  statusEl.textContent = remainingSeconds === 0 ? 'Koniec' : '';
  document.title = remainingSeconds === 0 ? 'Koniec!' : `${time} · Licznik`;
}
```

Po Stop tytuł ma wrócić do „Licznik”. Najczyściej: reakcja na Stop woła `render`? Nie, bo `render` nie wie, że zatrzymano. Dwa akceptowalne rozwiązania:

- `stop()` ustawia `document.title = 'Licznik'` (prostsze, ale to drugie miejsce piszące do DOM; w rozdziale 3 zniknie, gdy `render` dostanie cały stan);
- `render` dostaje drugi argument `running` i ustawia tytuł zależnie od niego: `render(remaining, intervalId !== null)`.

**Na co patrzeć przy review.** Czy `document.title` jest ustawiane w jednym miejscu. Jeśli uczeń wpisał je w `tick()`, w reakcji na `submit` i w reakcji na Stop osobno, to działa, ale pokazuje, że nie zrozumiał roli `render()`; odeślij do pytania 3 z zadania. Sprawdź przypadek: Start, Stop, Start: tytuł ma znów odliczać. I przypadek Koniec: tytuł „Koniec!”, nie „00:00 · Licznik”.

## Zadanie 3: wyjaśnienia

1. **`tick` a `tick()`.** `setInterval` chce dostać funkcję, którą sam będzie wywoływać. `tick` to funkcja jako wartość. `tick()` to wywołanie: wykona się natychmiast, raz, a do `setInterval` trafi to, co `tick` zwróciło, czyli `undefined`. `setInterval(undefined, 250)` nic nie robi (w niektórych przeglądarkach rzuci błąd). Efekt: cyfry pojawiają się raz i stoją.
2. **Karta w tle.** Przeglądarka w tle wywołuje interwał rzadziej niż co sekundę. Wersja z `remaining - 1` zakłada, że każde wywołanie to sekunda, więc po 120 sekundach w tle mogła odjąć np. 60. Wersja z `endsAt` przy każdym wywołaniu pyta zegar i liczy różnicę, więc jedno wywołanie po powrocie od razu pokazuje prawdę. Stan to moment końca, nie licznik tyknięć.
3. **Jedno miejsce pisania do DOM.** Gdy wszystko, co widać, wynika z jednej funkcji i jednego argumentu, nie da się mieć ekranu w stanie niespójnym (cyfry mówią jedno, tytuł drugie). W zadaniu 2 ustawianie `document.title` w `tick()` i w Stop osobno oznaczałoby, że przy Koniec tytuł ustawia `tick`, przy Stop reakcja na Stop, a przy Start nikt, więc po Stop i Start tytuł pokazywałby „Licznik” do pierwszego tyknięcia. Plus za zauważenie, że to przygotowanie pod `render(state)` z rozdziału 8 i pod React.

**Na co patrzeć przy review.** Pytanie 1 najczęściej dostaje odpowiedź „bo z nawiasami się wywołuje”; dopytaj, co dokładnie dostanie `setInterval`. Pytanie 2 ma mieć słowo „zegar” albo „Date.now” i zdanie o tym, czym jest stan w obu wersjach.

## Uwagi do całego rozdziału

Uruchom projekt i zrób punkty 2, 3 i 5 z „Sprawdź, czy działa” (podwójny Start, Stop, przycisk Stop bez `type="button"`). Sprawdź w kodzie: `event.preventDefault()` jako pierwsza linia w `submit`, `Number(...)` przy odczycie pola, `stop()` zamiast powtórzonego `clearInterval`, brak zmiennej `remaining` w zasięgu globalnym po kroku 5. Jeśli uczeń zostawił wersję z `remaining - 1` z kroku 4 i nie przeszedł do `endsAt`, to Request changes: krok 5 jest sednem rozdziału.
