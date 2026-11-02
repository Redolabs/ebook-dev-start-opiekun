# Rozdział 0: rozwiązania

## Zadanie 1: `isLong` i dwa sposoby filtrowania

```js
function isLong(movie) {
  return movie.duration > 150;
}

for (const movie of movies) {
  if (isLong(movie)) {
    console.log(movie.title);
  }
}

console.log(movies.filter(isLong).map((m) => m.title));
```

**Na co patrzeć przy review.** `isLong(movie)` zwraca wynik porównania wprost, nie `if (...) return true; else return false;`. W `filter(isLong)` jest sama nazwa funkcji, bez nawiasów i bez strzałki `(m) => isLong(m)` (ta druga wersja działa, ale uczeń ma zobaczyć, że funkcję podaje się jak wartość). Jeśli użył `>=`, zapytaj, czy film trwający dokładnie 150 minut jest „dłuższy niż”.

## Zadanie 2: `formatTime`

```js
function formatTime(totalSeconds) {
  const minutes = Math.floor(totalSeconds / 60);
  const seconds = totalSeconds % 60;
  return `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`;
}
```

`formatTime(0)` daje `'00:00'`, `formatTime(9)` `'00:09'`, `formatTime(75)` `'01:15'`, `formatTime(600)` `'10:00'`, `formatTime(3600)` `'60:00'`.

**Na co patrzeć przy review.** `Math.floor`, nie `parseInt(totalSeconds / 60)` ani `Math.round` (dla 119 sekund `round` da 2 minuty). `padStart` na tekście, czyli `String(x).padStart(...)`; `minutes.padStart` rzuci `TypeError`, bo liczba nie ma tej metody. Jeśli uczeń użył `toString()`, w porządku. Częsty błąd: `${minutes}:${seconds}` bez wyrównania, co daje `1:5` zamiast `01:05`; sprawdź wynik dla `65`.

## Zadanie 3: wyjaśnienia

Dobre odpowiedzi zawierają:

1. **Referencja.** `const b = a` nie tworzy nowej tablicy, tylko drugą nazwę na tę samą tablicę w pamięci; zmiana przez `b` jest widoczna przez `a`. Sprawdzenie: `b.push(...)` i odczyt `a`. Plus za porównanie z PHP, gdzie `$b = $a` kopiuje. Plus za `const b = [...a]` jako kopię, choć tego jeszcze nie było.
2. **`===`.** `==` zamienia typy przed porównaniem według reguł, których się nie pamięta (`0 == ''`, `1 == '1'`), a dane z formularzy i adresu są zawsze tekstem, więc łatwo porównać tekst z liczbą i dostać przypadkowe `true`. `===` sprawdza wartość i typ.
3. **`undefined` vs `null`.** `undefined` to brak wartości, którego nikt nie ustawił (zmienna bez wartości, brak pola w obiekcie, funkcja bez `return`); `null` to pustka wpisana celowo. W kroku 3 pojawiło się tylko `undefined`, bo nikt nic nie ustawiał; `null` byłoby, gdyby ktoś napisał `user.email = null`.

**Na co patrzeć przy review.** Odpowiedzi mają być własnymi słowami, nie przeklejonym fragmentem rozdziału. Jeśli odpowiedź na 1 brzmi „bo tak działa JavaScript”, to nie jest odpowiedź; dopytaj, co dokładnie zobaczył w konsoli. Jeśli uczeń pomylił kierunek (twierdzi, że `null == undefined` jest `false`), skieruj do konsoli, nie poprawiaj sam.

## Uwagi do całego rozdziału

Rozdział 0 oddawany jest jako katalog, więc nie ma PR. Przejrzyj `main.js` i `odpowiedzi.md`. Jeśli odpowiedzi nie ma, nie czytaj kodu, tylko poproś o uzupełnienie: ten nawyk ma wejść od pierwszego rozdziału.
