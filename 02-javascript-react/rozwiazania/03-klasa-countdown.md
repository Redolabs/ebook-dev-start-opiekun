# Rozdział 3: rozwiązania

## Zadanie 1: `Stopwatch`

```js
class Stopwatch {
  constructor(root) {
    this.timeEl = root.querySelector('.czas');
    this.statusEl = root.querySelector('.status');
    this.startedAt = null;
    this.intervalId = null;
    this.render();
  }

  start() {
    this.stop();
    this.startedAt = Date.now();
    this.intervalId = setInterval(() => this.render(), 250);
    this.render();
  }

  stop() {
    if (this.intervalId !== null) {
      clearInterval(this.intervalId);
      this.intervalId = null;
    }
    this.render();
  }

  isRunning() {
    return this.intervalId !== null;
  }

  elapsed() {
    if (this.startedAt === null) {
      return 0;
    }
    return Math.round((Date.now() - this.startedAt) / 1000);
  }

  render() {
    this.timeEl.textContent = formatTime(this.elapsed());
    this.statusEl.textContent = this.startedAt !== null && !this.isRunning() ? 'Zatrzymano' : '';
  }
}
```

Uwaga: po `stop()` czas dalej płynie w `elapsed()`, bo liczy z zegara. Poprawny stoper po Stop pokazuje czas zatrzymania, a nie czas od startu. Rozwiązanie: w `stop()` zapamiętać `this.stoppedAt = Date.now()` i w `elapsed()` liczyć do `stoppedAt`, gdy nie działa. To dobry temat na komentarz w review, jeśli uczeń tego nie zauważył; nie wymagaj, jeśli zadanie brzmi „liczy w górę”.

W `main.js` nie zmienia się nic poza linią `new Stopwatch(...)` zamiast `new Countdown(...)`, bo oba mają `start()` i `stop()`. Jedyna różnica: `start()` stopera nie przyjmuje sekund, więc `second.start(10)` zadziała (argument zignorowany), ale jest mylące.

**Na co patrzeć przy review.** Czy `elapsed()` liczy z zegara, czy zlicza tyknięcia (to drugie to regres do rozdziału 2). Czy `setInterval` ma strzałkę. Czy uczeń zauważył, że `main.js` nie wymaga zmian, i umiał to nazwać (interfejs).

## Zadanie 2: `reset()`

```js
  reset() {
    this.stop();
    this.endsAt = null;
    this.render();
  }
```

`stop()` zostawia `endsAt`, więc licznik pamięta, że coś odliczał, i pokazuje „Zatrzymano” z pozostałym czasem. `reset()` zeruje `endsAt`, więc `render()` wraca do `00:00` bez komunikatu i bez klasy `koniec`, bo oba warunki sprawdzają `endsAt !== null`.

**Na co patrzeć przy review.** `reset()` nie ma dotykać `textContent` ani `classList`; jeśli uczeń wpisał tam `this.timeEl.textContent = '00:00'`, to ominął `render()` i nie zrozumiał kroku 2. Przycisk Reset ma `type="button"`.

## Zadanie 3: wyjaśnienia

1. **`render()` bez argumentu.** Stan jest w obiekcie, więc `render` sam go czyta przez `this.remaining()`. Nie da się wywołać `render` ze złą liczbą, bo nie ma skąd jej podać; każde wywołanie pokazuje prawdę o obiekcie. Plus za zdanie „ekran jest funkcją stanu”.
2. **`null`, nie `0`.** `0` oznaczałoby „odliczano i skończyło się”, więc `remaining()` dałoby 0 i `render()` pokazałoby czerwone „Koniec” na starcie. `null` to osobny stan „nic nie odliczano” i oba warunki w `render()` (`endsAt !== null`) go rozróżniają.
3. **Eksperyment w konsoli.** Pola klasy są publiczne, więc każdy skrypt na stronie (i użytkownik w konsoli) może je zmienić i oszukać licznik. Wniosek: niczego, co ma znaczenie (czas na zakup, limit), nie wolno pilnować wyłącznie w przeglądarce; pilnuje serwer, a przeglądarka tylko pokazuje. Plus za odniesienie do blokady miejsc w kino-api.

**Na co patrzeć przy review.** Odpowiedź 3 często kończy się na „bo można zmienić w konsoli”; dopytaj, co z tego wynika dla kodu, który będzie pisał w części IV.

## Uwagi do całego rozdziału

Sprawdź, że `querySelector('.czas')` jest wywołane na `root`, nie na `document` (inaczej oba liczniki znalazłyby ten sam element, co widać dopiero w kroku 5). Sprawdź, że `stop()` kończy się `render()`. Jeśli drugi licznik z kroku 5 został w kodzie razem ze stoperem z zadania 1, to w porządku; rozdział 4 i tak czyści HTML.
