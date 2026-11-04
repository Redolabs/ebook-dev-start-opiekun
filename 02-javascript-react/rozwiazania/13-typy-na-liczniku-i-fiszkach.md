# Rozdział 13: rozwiązania

## Zadanie 1: `no-shadow` i jedna reguła z `recommended`

W `eslint.config.js`, w `rules`:

```js
      '@typescript-eslint/no-shadow': 'error',
```

Nie `no-shadow` (bazowa reguła ESLinta zgłasza fałszywe alarmy dla `interface X` i `const X`, czyli typu i wartości o tej samej nazwie, co w TypeScripcie jest legalne). W liczniku z książki reguła nie zgłasza nic po rozdziale 5 (parametr `s` w `start`); w fiszkach też nic. Jeśli uczeń robił własne rozwiązania, mogą wyjść przypadki typu `const card = cards.find((card) => ...)`.

Reguły z `tseslint.configs.recommended`, które najczęściej zaskakują, i czego się spodziewać w opisie:

- `@typescript-eslint/no-explicit-any`: jawne `any` to błąd. Uczeń może trafić na nią w zadaniu 2 przy `deepFreeze`; właściwa odpowiedź to `unknown`.
- `@typescript-eslint/no-unused-vars`: zastępuje bazową regułę i rozumie typy (nieużywany import typu też zgłasza; `tsc` z `noUnusedLocals` zgłosi to samo, więc uczeń zobaczy błąd dwa razy).
- `@typescript-eslint/no-non-null-assertion` **nie** jest w `recommended` (jest w `strict`); jeśli uczeń ją włączył, musiał zamienić `!` w `main.ts` na coś innego; zapytaj, na co.
- `@typescript-eslint/ban-ts-comment`: `// @ts-ignore` to błąd. Dobrze, żeby uczeń ją zauważył i zrozumiał, dlaczego istnieje.

**Na co patrzeć przy review.** Reguła z przedrostkiem `@typescript-eslint/`, nie bazowa. Czy `npm run lint` nadal przechodzi w obu projektach. Opis reguły własnymi słowami, nie skopiowany z dokumentacji; wystarczy zdanie.

## Zadanie 2: typy w rozwiązaniach z części I

Najczęstsze:

```ts
export function shuffle<T>(items: T[], random: () => number = Math.random): T[]
```

Uczeń nie zna jeszcze generyków (rozdział 15), więc zaakceptuj też `shuffle(cards: Card[], random: () => number = Math.random): Card[]`. Jeśli napisał wersję z `T` sam, pochwal i zapytaj, czy wie, co zrobił.

```ts
export function countByCategory(cards: Card[]): Record<string, number> {
  return cards.reduce<Record<string, number>>((counts, card) => {
    counts[card.category] = (counts[card.category] ?? 0) + 1;
    return counts;
  }, {});
}
```

`reduce` z pustym obiektem jako wartością początkową wnioskuje typ `{}` i każde przypisanie klucza jest błędem; trzeba podać typ akumulatora albo przez `reduce<Record<string, number>>(...)`, albo przez `{} as Record<string, number>`. Pierwsze jest czystsze; drugie to rzutowanie, do którego rozdział 14 ma zastrzeżenia. Obie wersje akceptuj, ale przy `as` zapytaj w komentarzu, co ono robi.

```ts
export function deepFreeze(value: unknown): unknown {
  if (typeof value === 'object' && value !== null) {
    Object.values(value).forEach(deepFreeze);
    Object.freeze(value);
  }
  return value;
}
```

Po `typeof value === 'object' && value !== null` TypeScript zawęża `unknown` do `object` i `Object.values` działa. W teście `deepFreeze(createState(cards))` zwraca `unknown`, więc `addCard(frozen(), ...)` nie skompiluje się; rozwiązanie: `deepFreeze(state) as State` w teście (tu rzutowanie jest uczciwe, bo funkcja zwraca to, co dostała) albo sygnatura `deepFreeze<T>(value: T): T` (znów generyk). Uczeń, który dojdzie do tego miejsca, zobaczy pierwszy raz, że `unknown` jest niewygodne; to celowe, rozdział 14 wyjaśni, dlaczego jest warte tej niewygody.

`markKnown(state: State, id: string): State` wymaga pola `known?: boolean` w `Card` (opcjonalne, bo stare fiszki w `localStorage` go nie mają). Stoper: `interface StopwatchApi { start(): void; stop(): void; lap(): void; laps(): number[]; }` w `types.ts`, `startedAt: number | null`, `laps: number[]`.

**Na co patrzeć przy review.** Brak `any` (poza `JSON.parse`, którego nie da się uniknąć; to temat rozdziału 14). `Record<string, number>` zamiast `object` czy `{}`. Opcjonalne `known?` zamiast obowiązkowego z dopisaniem `known: false` do wszystkich danych (obie wersje działają; opcjonalne lepiej znosi stare dane w `localStorage`). `npm run check` zielone w obu projektach, co sprawdzisz sam na gałęzi.

## Zadanie 3: wyjaśnienia

1. **Trzy błędy z grupami.** Nie ma jednej dobrej odpowiedzi; sprawdź, czy przypisanie do grupy jest uzasadnione. Typowy dobry zestaw: `Parameter 'root' implicitly has an 'any' type` (grupa 1, brak informacji); `Property 'trim' does not exist on type 'FormDataEntryValue'` (grupa 2, założenie niesprawdzane: pole formularza może być plikiem); `Argument of type '{ id: string; }' is not assignable to parameter of type 'Card'` (grupa 3, test podawał dane, które w aplikacji nie mogą zajść). Częsty błąd w klasyfikacji: `event.target` bez `instanceof` wrzucone do grupy 3; to grupa 2, bo kod działał poprawnie dla każdego realnego zdarzenia. Jeśli uczeń broni grupy 3 argumentem o `target` będącym np. węzłem tekstowym, zaakceptuj; granica między 2 i 3 bywa płynna i dyskusja o tym jest wartościowsza niż „poprawna” odpowiedź.
2. **`!` a `requireElement`.** `querySelector<HTMLFormElement>('#f')!` to dwie obietnice (typ i nie-`null`), zero sprawdzeń w czasie wykonania; przy pomyłce błąd wygląda jak w JavaScripcie (`Cannot read properties of null`), tylko gdzieś dalej. `requireElement` sprawdza `instanceof HTMLElement`, rzuca z nazwą selektora, ale zwraca ogólny `HTMLElement` (nie wie o `reset()` formularza). Obietnica: elementy z własnego `index.html`, pobierane raz przy starcie, bo pomyłka wyjdzie natychmiast. Sprawdzenie: elementy szukane w cudzym `root`, w pętli, w komponencie wielokrotnego użytku.
3. **`Number(root.dataset.seconds)`.** `dataset.seconds` jest `string | undefined`; `Number` przyjmuje dowolny argument i zwraca `number`, a `NaN` jest poprawną wartością typu `number`. TypeScript sprawdza, czy **typy** się zgadzają (zgadzają się), a nie czy **wartość** ma sens. Wniosek, którego oczekujesz: typ opisuje kształt, nie poprawność; dane z HTML-a, formularza, sieci i `localStorage` trzeba sprawdzać kodem, który działa w czasie wykonania. To jest pomost do rozdziału 14; jeśli uczeń sam napisze „to samo z `JSON.parse` w `loadCards`”, jest przygotowany.

**Na co patrzeć przy review.** W odpowiedzi 1 numer linii i dosłowny komunikat (świadczy o tym, że uczeń czytał wyjście `tsc`, a nie tylko kolorowe podkreślenia). W odpowiedzi 3 słowo „kształt” albo „w czasie wykonania”.

## Uwagi do całego rozdziału

Przed review uruchom w obu projektach `npm run check`, `npm run lint`, `npm test`. Sprawdź `package.json`: `typescript@6` (nie 7, bo `typescript-eslint` 8.x wymaga `<6.1`; jeśli do czasu review `typescript-eslint` zaczął wspierać 7, uczeń może mieć 7 i to jest w porządku, zaktualizuj wtedy akapit w rozdziale). `tsconfig.json` ze `strict: true` wpisanym jawnie, mimo że to domyślne od TypeScriptu 6.

Czego nie wymagać: typów zwracanych na każdej strzałce (wnioskowanie wystarcza), `readonly`, `private` w klasie, `enum`, `namespace`. Czego wymagać: `import type` dla typów (inaczej kompilator i tak nie przepuści), `Record<string, ...>` dla map akcji, `instanceof` przed `dataset`/`matches`/`closest`, brak `as` (jedyne akceptowalne miejsca: test `deepFreeze` i ewentualnie akumulator `reduce`).

Najczęstsze obejście, które trzeba zatrzymać: `// @ts-ignore` albo `any` w parametrze, żeby „zamknąć” błąd grupy 2. Komentarz w review: „kompilator miał rację, że tu czegoś brakuje; co dokładnie?”. Drugie obejście: `strict: false`. Nie.

Jeśli uczeń robił wszystko w jednym commicie zamiast po krokach, nie blokuj, ale przypomnij, że pierwszy commit (czerwony, same zmiany nazw) jest w książce celowo: pokazuje w historii, że zmiana rozszerzeń i dopisanie typów to dwa osobne kroki.
