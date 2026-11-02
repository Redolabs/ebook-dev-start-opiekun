# Rozdział 5: rozwiązania

## Zadanie 1: `createStopwatch`

```js
function createStopwatch(root) {
  const timeEl = root.querySelector('.czas');
  const statusEl = root.querySelector('.status');
  const lapsEl = root.querySelector('.laps');

  let startedAt = null;
  let stoppedAt = null;
  let intervalId = null;
  const laps = [];

  function elapsed() {
    if (startedAt === null) {
      return 0;
    }
    const end = stoppedAt ?? Date.now();
    return Math.round((end - startedAt) / 1000);
  }

  function isRunning() {
    return intervalId !== null;
  }

  function render() {
    timeEl.textContent = formatTime(elapsed());
    statusEl.textContent = startedAt !== null && !isRunning() ? 'Zatrzymano' : '';
    lapsEl.textContent = '';
    for (const lap of laps) {
      const li = document.createElement('li');
      li.textContent = formatTime(lap);
      lapsEl.append(li);
    }
  }

  function stop() {
    if (intervalId !== null) {
      clearInterval(intervalId);
      intervalId = null;
      stoppedAt = Date.now();
    }
    render();
  }

  function start() {
    stop();
    startedAt = Date.now();
    stoppedAt = null;
    laps.length = 0;
    render();
    intervalId = setInterval(render, 250);
  }

  function lap() {
    if (isRunning()) {
      laps.push(elapsed());
      render();
    }
  }

  root.querySelector('[data-action="start"]').addEventListener('click', start);
  root.querySelector('[data-action="stop"]').addEventListener('click', stop);
  root.querySelector('[data-action="lap"]').addEventListener('click', lap);

  render();

  return { start, stop, lap, laps: () => [...laps] };
}
```

`document.createElement` i `append` nie były jeszcze w książce; uczeń może zrobić listę przez `lapsEl.innerHTML = laps.map(...).join('')`. To działa, ale `innerHTML` z danymi od użytkownika to XSS; tu dane są liczbami, więc akceptowalne, z komentarzem w review. Rozdział 6 wprowadzi `createElement`.

Żeby stoper przeszedł przez `makeCountdown`, musiałby przyjmować `options` i mieć `label`; `start(seconds)` nie ma dla niego sensu. Dobra odpowiedź: to różne rzeczy z częściowo wspólnym interfejsem; wspólne jest `start()`/`stop()`, a wciskanie stopera w pętlę liczników to zły pomysł.

**Na co patrzeć przy review.** `laps()` zwraca kopię (`[...laps]`), nie oryginał. `stoppedAt`, żeby czas po Stop nie płynął (w rozdziale 3 to była uwaga, tu jest w zadaniu wprost: „stan to startedAt, intervalId i laps”, więc jeśli uczeń doszedł do `stoppedAt` sam, to plus). Czy `lap()` nic nie robi, gdy stoper stoi.

## Zadanie 2: `no-shadow`

W `eslint.config.js`, w `rules`: `'no-shadow': 'error'`. W kodzie z rozdziału nie ma przykrycia (`countdown` w `onFinish` i `countdowns` to różne nazwy), więc lint zostaje czysty; może zgłosić `left` lub `s`, jeśli uczeń nazwał je inaczej. Celowe `start(seconds = seconds)`: ESLint zgłasza `'seconds' is already declared in the upper scope`, a przeglądarka rzuca `ReferenceError: Cannot access 'seconds' before initialization`, bo parametr `seconds` przykrywa opcję, a jego wartość domyślna odwołuje się do niego samego.

**Na co patrzeć przy review.** Czy uczeń opisał **oba** komunikaty (ESLint i przeglądarka). To ćwiczenie pokazuje, że linter łapie błąd przed uruchomieniem.

## Zadanie 3: wyjaśnienia

1. **Gdzie mieszka `endsAt`.** W klasie: jako pole obiektu, `this.endsAt`, dostępne dla każdego, kto ma obiekt. W fabryce: jako zmienna lokalna wywołania `createCountdown`, utrzymywana przy życiu przez funkcje, które ją widzą (domknięcie); z zewnątrz nie ma do niej drogi, bo zwrócony obiekt zawiera tylko funkcje, nie zmienną.
2. **`tick` a `this.tick`.** `tick` w fabryce nie używa `this`, tylko zmiennych z domknięcia, więc nie ma znaczenia, kto i jak go wywoła. `this.tick` w klasie potrzebuje obiektu przed kropką, a `setInterval` go nie daje. To nie „ta sama” funkcja: jedna ma stan w zasięgu, druga w `this`.
3. **Dwa razy na tym samym `root`.** Dwa wywołania to dwa zasięgi, czyli dwa niezależne stany, i dwa komplety reakcji na te same przyciski. Jedno kliknięcie Start uruchamia oba liczniki; oba piszą do tych samych `.czas` i `.status`, więc cyfry pokazują raz jeden, raz drugi (różnią się o ułamek sekundy, więc na oko może nie być widać, ale `countdowns.length` i dwa interwały są). Plus za zauważenie, że to ten sam problem co podwójny `setInterval` z rozdziału 2, tylko piętro wyżej.

**Na co patrzeć przy review.** W odpowiedzi 1 słowo „domknięcie” albo opis „funkcja trzyma zmienne z miejsca, gdzie powstała”. W odpowiedzi 3 liczby: dwa stany, dwie reakcje.

## Uwagi do całego rozdziału

Sprawdź przełącznik `makeCountdown`: oba warianty mają działać bez innych zmian. Sprawdź, że `let endsAt` jest **wewnątrz** fabryki, nie na poziomie modułu. `const countdown = {...}` przed `tick` albo po: oba działają, jeśli `start`/`stop` są deklaracjami `function`; jeśli uczeń użył strzałek i kolejność się posypała, komunikat `Cannot access ... before initialization` wskaże miejsce. Jeśli główny plik ma już 200 linii, to dobrze: rozdział 6 zaczyna się od tego bólu.
