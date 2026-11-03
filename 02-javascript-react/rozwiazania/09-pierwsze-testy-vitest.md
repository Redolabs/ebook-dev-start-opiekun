# Rozdział 9: rozwiązania

## Zadanie 1: test, który łapie każdą mutację

```js
function deepFreeze(value) {
  if (value !== null && typeof value === 'object') {
    Object.values(value).forEach(deepFreeze);
    Object.freeze(value);
  }
  return value;
}

describe('przejścia nie mutują', () => {
  const frozen = () => deepFreeze(createState(cards));

  it('addCard', () => {
    expect(() => addCard(frozen(), { id: 'd' })).not.toThrow();
  });
  it('removeCard', () => {
    expect(() => removeCard(frozen(), 'a')).not.toThrow();
  });
  it('selectCategory', () => {
    expect(() => selectCategory(frozen(), 'Y')).not.toThrow();
  });
  it('flip, next, prev', () => {
    expect(() => prev(next(flip(frozen())))).not.toThrow();
  });
  it('markKnown', () => {
    expect(() => markKnown(frozen(), 'a')).not.toThrow();
  });
});
```

Zepsute `addCard` przez `push`: `TypeError: Cannot add property 1, object is not extensible` (sprawdzone na Vitest 5). Zepsute `markKnown` przez `card.known = true`: łapie **tylko** test z `deepFreeze` (`Cannot add property known, object is not extensible`); test „nie zmienia starego stanu” z kroku 3 sprawdza długość tablicy, a ta się nie zmienia. To jest odpowiedź na pytanie z zadania: test z długością łapie dodanie i usunięcie, test z zamrożeniem łapie każdą zmianę na każdym poziomie, także zmianę pola w fiszce.

**Na co patrzeć przy review.** `value !== null` przed `typeof` (tabela błędów). `deepFreeze` wywoływane na **świeżym** stanie w każdym teście (stąd `frozen()` jako funkcja), bo zamrożonego nie da się odmrozić i testy by na sobie wisiały. Czy uczeń zrobił test dla każdego przejścia, w tym własnych z rozdziału 8 (`markKnown`, `markCurrentKnownAndNext`).

## Zadanie 2: test losowania

W `cards.js`:

```js
export function shuffle(cards, random = Math.random) {
  const copy = [...cards];
  for (let i = copy.length - 1; i > 0; i--) {
    const j = Math.floor(random() * (i + 1));
    [copy[i], copy[j]] = [copy[j], copy[i]];
  }
  return copy;
}
```

Test:

```js
describe('shuffle', () => {
  it('z generatorem zwracającym 0 daje przewidywalną kolejność', () => {
    expect(shuffle(['a', 'b', 'c'], () => 0)).toEqual(['b', 'c', 'a']);
  });

  it('z generatorem bliskim 1 zostawia kolejność', () => {
    expect(shuffle(['a', 'b', 'c'], () => 0.99)).toEqual(['a', 'b', 'c']);
  });

  it('zachowuje elementy i nie zmienia wejścia', () => {
    const input = ['a', 'b', 'c', 'd'];
    const result = shuffle(input);
    expect([...result].sort()).toEqual([...input].sort());
    expect(input).toEqual(['a', 'b', 'c', 'd']);
  });
});

describe('countByCategory', () => {
  it('liczy fiszki w kategoriach', () => {
    expect(countByCategory(cards)).toEqual({ X: 2, Y: 1 });
  });

  it('dla pustej tablicy daje pusty obiekt', () => {
    expect(countByCategory([])).toEqual({});
  });
});
```

Wyniki `['b', 'c', 'a']` i `['a', 'b', 'c']` sprawdzone w Node. Uczeń ma to policzyć na kartce: z `j = 0` pętla zamienia `copy[2]` z `copy[0]` (daje `c b a`), potem `copy[1]` z `copy[0]` (daje `b c a`).

**Na co patrzeć przy review.** Parametr z wartością domyślną, nie globalna zmiana `Math.random`. Czy w `shuffle` nie został `Math.random()` (objaw: test losowo pada). Test trzeci używa prawdziwego losowania, ale sprawdza tylko to, co jest prawdą dla każdej permutacji; to dobry wzór na testowanie niedeterministycznych funkcji.

## Zadanie 3: wyjaśnienia

1. **`toBe` a `toEqual`.** `toBe` to `Object.is`, czyli praktycznie `===`: dla obiektów ten sam obiekt. `toEqual` porównuje rekurencyjnie zawartość. Test „na pustej liście zwracają ten sam stan” z `toEqual` przechodziłby też dla kopii, czyli przestałby sprawdzać to, o co chodzi. W drugą stronę: `expect(currentCard(state)).toBe(cards[0])` by **przeszedł**, bo `visibleCards` przy `wszystkie` zwraca tę samą tablicę, a więc te same obiekty; ale `expect(visibleCards(state)).toBe([cards[2]])` dla kategorii `Y` by padł, bo `filter` zwraca nową tablicę. Wystarczy, że uczeń wskaże jeden konkretny test.
2. **Dlaczego `state.js`, a nie `ui.js`.** `state.js` nie dotyka DOM-u, `localStorage` ani sieci; `ui.js` wywołuje `document.createElement`, którego w Node nie ma. Żeby przetestować obsługę `submit`, trzeba by wyciągnąć z handlera funkcję czystą, np. `cardFromFormData(data)` przyjmującą obiekt zamiast formularza, i testować ją; sam `submit` zostaje w `main.js` i wymaga przeglądarki lub `jsdom`.
3. **Parametr zamiast nadpisania `Math.random`.** Nadpisanie jest globalne: zmienia zachowanie dla wszystkich testów w procesie, dopóki ktoś nie przywróci, i działa też na kod, który testu nie dotyczy. Parametr dotyka jednego wywołania. Plus za słowo „wstrzykiwanie zależności” albo odwołanie do kontenera z ebooka 1.

**Na co patrzeć przy review.** W odpowiedzi 1 konkretny test. W odpowiedzi 3 słowo „globalne” lub „inne testy”.

## Uwagi do całego rozdziału

Vitest 5 w `npm install -D vitest` (listopad 2026). W trybie nie-TTY (CI, niektóre terminale edytora) domyślny reporter nie wypisuje linii `✓ plik (n tests)`, tylko podsumowanie; jeśli uczeń pyta, czemu u niego wygląda inaczej niż w książce, to to. Sprawdź `package.json`: skrypt `test` ma być `vitest`, nie `vitest run` (książka chce trybu watch jako domyślnego). Pliki testów w `src/` obok kodu. Importy z `'vitest'` jawne; jeśli uczeń ustawił `globals: true` w konfiguracji, ESLint zgłosi `describe is not defined` i uczeń doda `globals.vitest` lub podobne do configu; akceptowalne, ale zapytaj, dlaczego.
