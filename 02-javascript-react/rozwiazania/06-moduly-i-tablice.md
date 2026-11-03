# Rozdział 6: rozwiązania

## Zadanie 1: `shuffle`

```js
export function shuffle(cards) {
  const copy = [...cards];
  for (let i = copy.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [copy[i], copy[j]] = [copy[j], copy[i]];
  }
  return copy;
}
```

Zamiana przez destrukturyzację `[a, b] = [b, a]` jest w rozdziale tylko pośrednio (destrukturyzacja tablicy); uczeń może użyć zmiennej tymczasowej, to równie dobre. Przycisk „Pomieszaj” w `main.js` wymaga, żeby `render` umiało narysować inną kolejność; najprostsze rozwiązanie to `let order = cards` na poziomie modułu, `render` filtruje `order` zamiast `cards`, a przycisk robi `order = shuffle(cards); render();`. To dodaje piąty kawałek stanu, co jest dobrym paliwem do rozdziału 8.

`sort(() => Math.random() - 0.5)`: wynik zależy od algorytmu sortowania w silniku, część permutacji wypada częściej. Wystarczy, że uczeń napisze „nie daje równych szans każdej kolejności” i poda źródło.

**Na co patrzeć przy review.** Czy `shuffle` zaczyna od kopii i nie dotyka `cards` (sprawdzenie `cards[0].id === 'k1'` po wywołaniu). Czy `Math.random() * (i + 1)`, nie `* i` (bez `+ 1` ostatni element nigdy nie zostaje na miejscu).

## Zadanie 2: `countByCategory`

```js
export function countByCategory(cards) {
  return cards.reduce((acc, card) => {
    acc[card.category] = (acc[card.category] ?? 0) + 1;
    return acc;
  }, {});
}
```

W `renderCategories` opcje dostają trzeci argument albo funkcję etykiety, np. `renderCategories(selectEl, names, counts)` i `option.textContent = \`${name} (${counts[name] ?? cards.length})\``. Dla „wszystkie” trzeba osobno podać sumę; częste rozwiązanie to `{ ...countByCategory(cards), [ALL]: cards.length }`.

**Na co patrzeć przy review.** `option.value` musi zostać czystą nazwą, inaczej `byCategory` nic nie znajdzie (objaw: pusta lista po wyborze). `reduce` z `{}` jako wartością początkową; bez niej pierwszy element tablicy stałby się akumulatorem i wywaliłoby się na `acc[card.category]`. Jeśli uczeń zrobił to pętlą `for...of` i obiektem, zaakceptuj, ale poproś o wersję z `reduce` w komentarzu PR: zadanie wprost go wymaga, bo chodzi o umiejętność czytania.

## Zadanie 3: wyjaśnienia

1. **Tablica jako argument.** Funkcje działają na dowolnej tablicy, nie tylko tej z `data.js`. W rozdziale 7 fiszki przychodzą z `localStorage`, więc `byCategory` importujące `cards` z `data.js` filtrowałoby zawsze osiem fiszek startowych, a nie te, które użytkownik dodał. Plus za „łatwiej testować” (rozdział 9).
2. **`map` a `forEach`.** `map` zwraca nową tablicę wyników, `forEach` zwraca `undefined` i służy do efektów ubocznych. `map` bez użycia wyniku to błąd czytelności: czytelnik szuka, gdzie wynik idzie, a go nie ma; ESLint ma na to regułę `array-callback-return`. Jeśli uczeń napisał tylko „`map` zwraca tablicę”, dopytaj o drugą część.
3. **`sort` w `categories`.** Wywołany jest na `[...names]`, czyli na świeżej tablicy zrobionej z `Set`, nie na `cards`. `cards` nie jest nawet sortowane, więc nie ma czego zepsuć. Dobra odpowiedź nazywa po imieniu tablicę, na której stoi `sort`.

**Na co patrzeć przy review.** W odpowiedzi 1 słowo `localStorage` albo „rozdział 7”. W odpowiedzi 3 wskazanie konkretnej tablicy.

## Uwagi do całego rozdziału

Dwa projekty w jednym PR (podział licznika i start fiszek); sprawdź oba lintem, `npm run lint` w każdym katalogu osobno. W liczniku: czy `formatTime` nie zostało skopiowane do dwóch plików zamiast wyeksportowane z jednego. W fiszkach: `textContent`, nie `innerHTML` (to pierwszy rozdział, gdzie uczeń tworzy DOM z danych; XSS jest w tekście, więc `innerHTML` z danymi to Request changes). Każdy import z rozszerzeniem `.js`. `renderList` przez `map`, nie przez pętlę z `append` w środku; pętla działa, ale mija się z celem rozdziału.
