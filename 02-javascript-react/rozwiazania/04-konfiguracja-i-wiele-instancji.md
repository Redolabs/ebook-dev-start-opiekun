# Rozdział 4: rozwiązania

## Zadanie 1: brak `data-seconds`

Dwa poprawne miejsca, oba do przyjęcia, ważne jest uzasadnienie:

```js
// W main.js: HTML to „wejście z zewnątrz”, więc sprzątamy przy odczycie
const seconds = Number(root.dataset.seconds);
return new Countdown(root, {
  seconds: Number.isFinite(seconds) && seconds > 0 ? seconds : undefined,
  ...
});
```

`undefined` sprawia, że w konstruktorze zadziała `?? 60`. Alternatywa w konstruktorze:

```js
    const seconds = Number(options.seconds);
    this.seconds = Number.isFinite(seconds) && seconds > 0 ? seconds : 60;
```

Argument za `main.js`: klasa powinna zakładać, że dostaje poprawne dane, a sprzątanie należy do tego, kto czyta z HTML-u. Argument za konstruktorem: klasa broni się sama niezależnie od tego, kto jej użyje. W części II TypeScript rozstrzygnie to typami (`seconds: number`), a walidacja danych z zewnątrz trafi na granicę, czyli przy odczycie z HTML-u.

**Na co patrzeć przy review.** `Number.isNaN`, nie globalne `isNaN` (to drugie zamienia argument na liczbę i `isNaN('abc')` daje `true`, ale `isNaN('')` daje `false`). Czy uczeń sprawdził też `0` i liczbę ujemną, czy tylko `NaN`.

## Zadanie 2: łańcuch liczników

HTML: `<div class="licznik" id="herbata" data-seconds="3" data-next="jajko" ...>`, i tak dalej. W `main.js`:

```js
const byId = new Map();

const countdowns = Array.from(document.querySelectorAll('.licznik')).map((root) => {
  const countdown = new Countdown(root, {
    seconds: Number(root.dataset.seconds),
    label: root.dataset.label,
    onFinish: () => {
      document.title = `Koniec: ${root.dataset.label}`;
      const next = byId.get(root.dataset.next);
      if (next) {
        next.start();
      }
    },
  });
  byId.set(root.id, countdown);
  return countdown;
});
```

`root` jest dostępny w `onFinish` przez domknięcie (zapowiedź rozdziału 5), więc `this.root` w klasie nie jest konieczne. `Map` to słownik `id → licznik`; `byId.get(undefined)` dla ostatniego licznika daje `undefined`, stąd `if (next)`.

**Na co patrzeć przy review.** Zero zmian w klasie poza ewentualnym `this.root = root`. Jeśli uczeń dopisał do klasy `next` albo `startNext()`, to zrobił dokładnie to, przed czym ostrzega pojęcie callbacku: klasa wie o innych licznikach. Request changes z odesłaniem do pojęcia. Sprawdź, że łańcuch nie zapętla się (ostatni bez `data-next`).

## Zadanie 3: wyjaśnienia

1. **`this` w `setInterval`.** `this.tick` bez nawiasów wyjmuje funkcję z obiektu; `setInterval` wywołuje ją po 250 ms jako zwykłą funkcję, bez obiektu przed kropką, więc `this` jest `undefined` i `this.render` wybucha. Strzałka `() => this.tick()` nie ma własnego `this`, bierze go z `start()`, gdzie `this` to licznik, i wywołuje `tick` **na** liczniku, z kropką.
2. **Tytuł w `onFinish`.** Klasa nie wie, gdzie jest używana; tytuł karty to decyzja strony. W innym projekcie (np. licznik w aplikacji kina) zmiana tytułu byłaby błędem, a każdy z trzech liczników nadpisywałby tytuł. Callback przenosi decyzję do tego, kto licznik tworzy.
3. **`??` a `||`.** `||` traktuje każdą wartość „fałszywą” (`0`, `''`, `false`, `NaN`, `null`, `undefined`) jako brak; `??` tylko `null` i `undefined`. Dla `seconds: 0` dają różne wyniki: `0 || 60` to 60, `0 ?? 60` to 0. Plus za zauważenie, że dla `NaN` oba przepuszczają (`NaN ?? 60` to `NaN`), co było w punkcie 4.

**Na co patrzeć przy review.** W odpowiedzi 1 ma być słowo „wywołanie” albo „przed kropką”; „bo strzałka ma this” to nie wyjaśnienie. W odpowiedzi 3 konkretna wartość, dla której wyniki się różnią.

## Uwagi do całego rozdziału

Uruchom stronę i zrób punkt 2 ze „Sprawdź, czy działa” (herbata na 5 sekund). Sprawdź, że `onFinish` jest po `stop()`. Sprawdź, czy `querySelectorAll` jest owinięte w `Array.from` (albo spread). Jeśli w kodzie jest `this.tick.bind(this)` albo pole-strzałka zamiast strzałki w `setInterval`, to w porządku, byle uczeń umiał powiedzieć, dlaczego to działa.
