# Rozdział 7: rozwiązania

## Zadanie 1: usuwanie fiszki

W `ui.js`:

```js
export function createCardElement(card, onRemove) {
  // ...jak w rozdziale, plus:
  const remove = document.createElement('button');
  remove.type = 'button';
  remove.textContent = 'Usuń';
  remove.addEventListener('click', (event) => {
    event.stopPropagation();
    onRemove(card.id);
  });
  li.append(front, back, category, remove);
  return li;
}

export function renderList(listEl, cards, onRemove) {
  listEl.replaceChildren(...cards.map((card) => createCardElement(card, onRemove)));
}
```

W `main.js`:

```js
function removeCard(id) {
  cards = cards.filter((card) => card.id !== id);
  saveCards(cards);
  renderSelect();
  const count = visibleCards().length;
  index = count === 0 ? 0 : Math.min(index, count - 1);
  render();
}

// w render():
renderList(listEl, visible, removeCard);
```

`stopPropagation()` zatrzymuje bąbelkowanie: kliknięcie w przycisk nie dociera do listenera na `li`, więc fiszka się nie odwraca. Bez tego najpierw odpala się listener przycisku (usuwa), potem `li` (przełącza klasę na elemencie, który za chwilę zniknie z DOM-u po `render`), czyli efekt i tak byłby niewidoczny; uczeń może to zauważyć i napisać, że „działa bez”. Działa przypadkiem, bo `render` tworzy listę od nowa. Poproś o `stopPropagation` i tak: w rozdziale 8 lista nie będzie odwracalna, ale zasada „przycisk w klikalnym rodzicu zatrzymuje zdarzenie” zostaje.

**Na co patrzeć przy review.** Czy po usunięciu jest `renderSelect()` (kategoria mogła zniknąć). Czy poprawka `index` jest przed `render()`. Czy `onRemove` idzie przez argumenty, a nie przez import `main.js` do `ui.js` (cykliczny import; Vite go przepuści, ale to zły kierunek zależności: DOM nie może wiedzieć o stanie).

## Zadanie 2: eksport i import JSON

```js
function exportCards() {
  const blob = new Blob([JSON.stringify(cards, null, 2)], { type: 'application/json' });
  const url = URL.createObjectURL(blob);
  const link = document.createElement('a');
  link.href = url;
  link.download = 'fiszki.json';
  link.click();
  URL.revokeObjectURL(url);
}

async function importCards(file) {
  try {
    const parsed = JSON.parse(await file.text());
    if (!Array.isArray(parsed)) {
      throw new Error('Plik nie zawiera tablicy');
    }
    const known = new Set(cards.map((card) => card.id));
    const fresh = parsed.filter(
      (card) => typeof card.front === 'string' && typeof card.back === 'string' && !known.has(card.id),
    );
    cards = [...cards, ...fresh];
    saveCards(cards);
    renderSelect();
    render();
  } catch (error) {
    console.error('Import nie powiódł się', error);
  }
}

document.querySelector('#eksport').addEventListener('click', exportCards);
document.querySelector('#import').addEventListener('change', (event) => {
  importCards(event.target.files[0]);
});
```

`async`/`await` jest w książce dopiero w rozdziale 10; zadanie o tym mówi i każe potraktować `await` jako „poczekaj”. Uczeń może też użyć `file.text().then(...)`, równie dobrze. Alternatywa bez `await`: `FileReader` z `onload`, starsze, ale poprawne.

**Na co patrzeć przy review.** `JSON.parse` w `try`/`catch`. Sprawdzenie `Array.isArray`, bo `JSON.parse('{"a":1}')` to poprawny JSON, który wywaliłby `filter`. Pominięcie duplikatów po `id`. Brak `id` w importowanym pliku (ktoś edytował ręcznie): dobre rozwiązanie dopisuje `id: card.id ?? crypto.randomUUID()`; nie wymagaj, pochwal. `URL.revokeObjectURL` po `click()`.

## Zadanie 3: wyjaśnienia

1. **`localStorage` a sesja.** Sesja z ebooka 1 leży na serwerze (plik lub baza), a przeglądarka ma tylko ciasteczko z identyfikatorem; serwer decyduje, co w niej jest, i widzi ją z każdej przeglądarki, która ma to ciasteczko. `localStorage` leży w przeglądarce, serwer go nie widzi, zmiana przeglądarki lub komputera to pusty magazyn. Nie wolno trzymać: haseł, danych wrażliwych, niczego, czego utrata lub podmiana przez użytkownika byłaby problemem (każdy skrypt na stronie czyta `localStorage`, każdy użytkownik może go edytować). Plus za „tokeny tylko ostrożnie, rozdział 27”.
2. **`keydown` na `document`.** Zdarzenia klawiatury powstają w elemencie, który ma fokus. Gdyby listener wisiał na `#nauka`, działałby tylko, gdy fokus jest wewnątrz sekcji (np. na jej przycisku po kliknięciu); po kliknięciu gdziekolwiek indziej strzałki by umarły. `document` dostaje wszystko przez bąbelkowanie.
3. **Nieodwrócone po `renderList`.** `renderList` tworzy nowe elementy `li` od zera przez `createElement`; klasa `odwrocona` była na starych, które `replaceChildren` usunęło. Nikt jej nie zdejmował, bo nie ma już elementu, na którym była. Dobra odpowiedź nazywa to „stanem w DOM, którego nikt nie pamięta” i wskazuje, że to zapowiedź rozdziału 8.

**Na co patrzeć przy review.** W odpowiedzi 1 słowa „serwer” i „przeglądarka” po właściwych stronach. W odpowiedzi 2 słowo „fokus”.

## Uwagi do całego rozdziału

Rozdział ma celowo „brzydki” `main.js`: cztery kawałki stanu i ręczne odświeżanie. Nie poprawiaj tego w review, rozdział 8 to robi. Sprawdź natomiast: `event.preventDefault()` w `submit`; `trim()` na polach; `cards = [...cards, card]` zamiast `push` (jeśli `push`, uwaga bez Request changes, z odesłaniem do rozdziału 8); `try`/`catch` w `loadCards`; `event.target.matches(...)` w `keydown`. Przycisk „Dodaj” bez `type="button"`, pozostałe z, i czy uczeń rozumie różnicę (często pyta w PR).
