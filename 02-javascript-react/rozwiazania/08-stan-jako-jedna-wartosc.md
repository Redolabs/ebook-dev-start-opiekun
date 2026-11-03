# Rozdział 8: rozwiązania

## Zadanie 1: „Umiem” i statystyki

W `state.js`:

```js
export function markKnown(state, id) {
  return {
    ...state,
    cards: state.cards.map((card) => (card.id === id ? { ...card, known: true } : card)),
  };
}

export function markCurrentKnownAndNext(state) {
  const card = currentCard(state);
  if (card === null) {
    return state;
  }
  return next(markKnown(state, card.id));
}

export function stats(state) {
  const visible = visibleCards(state);
  const known = visible.filter((card) => card.known).length;
  const total = visible.length;
  return { known, total, percent: total === 0 ? 0 : Math.round((known / total) * 100) };
}
```

W `main.js` do `actions` dochodzi `known: markCurrentKnownAndNext`, w HTML przycisk `data-action="known"`. W `ui.js` `renderStudy` dostaje `stats(state)` (albo `render` wpisuje je w osobny `<p>`), a `createCardElement` robi `li.classList.toggle('znana', card.known === true)`.

**Na co patrzeć przy review.** `map` z nowym obiektem dla trafionej fiszki i tą samą referencją dla pozostałych; `card.known = true` w środku `map` to mutacja i Request changes (cały rozdział jest o tym). `stats` jako funkcja ze stanu, nie pole w stanie. Składanie `next(markKnown(...))` zamiast kopiowania logiki `move`. Czy przy pustej kategorii „Umiem” nie wywala się na `currentCard(state).id` (null).

## Zadanie 2: cofnij

```js
const history = [];

function setState(nextState, { record = true } = {}) {
  if (nextState === state) {
    return;
  }
  if (record) {
    history.push(state);
    if (history.length > 20) {
      history.shift();
    }
  }
  state = nextState;
  saveCards(state.cards);
  render(state, elements, handlers);
}

function undo() {
  const previous = history.pop();
  if (previous) {
    setState(previous, { record: false });
  }
}
```

Plus przycisk `data-action="undo"` (nie w `actions`, bo `undo` nie jest przejściem `state => state`; osobny `if` albo osobny listener) i w `keydown`: `if (event.ctrlKey && event.key === 'z') { event.preventDefault(); undo(); }`. Na macOS uczeń może dodać `event.metaKey`.

Sprawdzenie `history[0].cards === history[1].cards` po `flip`: `flip` kopiuje tylko wierzchni obiekt, tablica `cards` jest współdzielona, więc porównanie daje `true`. To dowód, że niemutowalność nie oznacza kopiowania wszystkiego.

**Na co patrzeć przy review.** Przywrócenie bez dopisywania do historii (objaw błędu w tabeli „Częste błędy”). Limit 20 przez `shift`, nie przez `slice` z nadpisaniem `const` (błąd składni, uczeń zobaczy sam). Czy `undo` na pustej historii nic nie robi. Uczeń może zapytać, czemu `history` to `const`, skoro się zmienia: `const` blokuje przypisanie do zmiennej, nie zmiany w tablicy; to wprost z rozdziału 0 i fiszka `k5`.

## Zadanie 3: wyjaśnienia

1. **`push` w `addCard`.** Trzy skutki: (a) `setState` dostaje ten sam obiekt, jeśli przejście zwraca `state`, i nic nie rysuje; jeśli zwraca `{ ...state }`, rysuje, ale (b) historia z zadania 2 pokazuje w każdym wpisie tę samą, już zmienioną tablicę, więc cofnięcie nie cofa dodania; (c) w React porównanie `===` tablicy przed i po da `true` i komponent się nie przerysuje. Wystarczą dwa z trzech, ale (c) powinno być, bo rozdział wprost o nim mówi.
2. **`visibleCards` jako funkcja.** Pole trzeba by aktualizować w każdym przejściu, które zmienia `cards` lub `category`; pominięcie jednego daje stan niespójny sam ze sobą (np. po `removeCard` lista widocznych zawiera usuniętą fiszkę). Funkcja liczy z aktualnych danych, nie da się jej „zapomnieć zaktualizować”.
3. **`return state` w `move`.** Ten sam obiekt to sygnał „bez zmian”; `setState` porównuje `===` i wychodzi bez zapisu i bez rysowania. Kopia `{ ...state }` byłaby nowym obiektem, `setState` zapisałby do `localStorage` i przerysował całą stronę mimo braku zmian. Działałoby, tylko niepotrzebnie.

**Na co patrzeć przy review.** W odpowiedzi 1 słowo „React” albo „`===`”. W odpowiedzi 3 zrozumienie, że `===` na obiektach porównuje tożsamość, nie zawartość (fiszka `k4` mówi o typach, tu chodzi o referencje; jeśli uczeń pomylił jedno z drugim, wyjaśnij w komentarzu).

## Uwagi do całego rozdziału

To najważniejszy rozdział części I: model, który tu powstaje, wraca w rozdziale 12 (refaktor repertuaru), 15 (typ stanu), 18 (`useState`) i 29 (Redux). Warto poświęcić na review więcej czasu niż zwykle. Lista kontrolna:

- `main.js` ma dokładnie jedną zmienną `let` (`state`); `history` z zadania 2 to `const`.
- Żadne przejście w `state.js` nie przypisuje do `state.*` ani nie woła `push`/`splice`/`sort` na `state.cards`.
- `ui.js` nie importuje `main.js` i nie czyta `localStorage`.
- `setState` jest jedynym miejscem z `saveCards` i `render`.
- Lista nie ma już listenera `click` na `li` (odwracanie zostało w trybie nauki).

Jeśli uczeń zostawił `renderSelect` z zapamiętywaniem `selected` z rozdziału 7, zapytaj w komentarzu, czy jest jeszcze potrzebne; dobra odpowiedź to „nie, bo `state.category` jest źródłem prawdy”.
