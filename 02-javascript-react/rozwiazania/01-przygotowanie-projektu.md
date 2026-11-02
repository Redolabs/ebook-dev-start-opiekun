# Rozdział 1: rozwiązania

## Zadanie 1: `lint:fix`

W `package.json`:

```json
    "lint:fix": "eslint . --fix"
```

`--fix` zamieni `let y = 5;` na `const y = 5;` (reguła `prefer-const`), bo to zmiana mechaniczna i bezpieczna. Nie naprawi nieużywanej zmiennej (`no-unused-vars`): ESLint nie wie, czy ją usunąć, czy jej użyć, i nie zgaduje. Nie naprawi też `==` na `===` w ogólnym przypadku, bo zmiana `==` na `===` może zmienić wynik programu (dla `x == null` ESLint celowo zostawia).

**Na co patrzeć przy review.** Czy uczeń faktycznie uruchomił i opisał, co się zmieniło, a nie tylko dopisał skrypt. Odpowiedź „nie naprawi wszystkiego” bez przykładu to za mało.

## Zadanie 2: `no-console` jako warning

W `eslint.config.js`, w `rules`:

```js
      'no-console': 'warn',
```

`npm run lint` wypisze `warning  Unexpected console statement  no-console` i zakończy się kodem 0, więc commit przechodzi. Ostrzeżenie zamiast błędu, bo przez pierwsze rozdziały `console.log` jest narzędziem nauki, a ostrzeżenie przypomina, żeby przed oddaniem rozdziału posprzątać. W kinie (część IV) można podnieść do `'error'`.

**Na co patrzeć przy review.** Reguła wpisana w obiekt `rules` istniejącego wpisu, nie jako nowy element tablicy po `prettier` (zadziała, ale pokazuje, że uczeń nie zrozumiał kolejności). Jeśli wpisał `'error'`, zapytaj, jak zrobi commit w rozdziale 2, gdzie `console.log` jest w krokach.

## Zadanie 3: wyjaśnienia

1. **ESLint a Prettier.** ESLint szuka błędów i złych wzorców w kodzie (nieużyta zmienna, `==`, `var`); Prettier ustawia wygląd kodu (wcięcia, cudzysłowy, długość linii) i nie ocenia jego sensu. Potrzebne oba, bo ładnie sformatowany kod może mieć błąd, a poprawny kod może być nieczytelny. `eslint-config-prettier` wyłącza w ESLint reguły o wyglądzie, żeby nie dublowały Prettiera.
2. **`node_modules/` a `package-lock.json`.** `node_modules/` jest ogromny i odtwarzalny z `package.json` przez `npm install`. `package-lock.json` zapisuje dokładne wersje wszystkich pakietów (także zależności zależności), żeby każdy, kto zrobi `npm install`, dostał identyczny zestaw. To samo co `composer.lock`.
3. **Jedno repo dla `kino-api` i `frontend`.** Mieszałyby się dwa języki, dwa menedżery pakietów i dwa rytmy zmian; PR z rozdziału JS pokazywałby też pliki PHP; uczeń od Reacta musiałby klonować backend, którego nie zmienia; wdrożenie frontu (Vercel) i API (serwer PHP) idzie w dwa różne miejsca. Plus za zauważenie, że takie wspólne repozytoria istnieją (monorepo) i mają własne narzędzia.

**Na co patrzeć przy review.** Odpowiedź 2 często myli `package-lock.json` z `package.json`; dopytaj o różnicę. Odpowiedź 3 jest pytaniem otwartym, liczy się argumentacja, nie jedna poprawna odpowiedź.

## Uwagi do całego rozdziału

Sprawdź w PR, że `node_modules/` nie trafił do repozytorium (zakładka Files changed nie powinna mieć tysięcy plików) i że `.vscode/settings.json`, `.prettierrc` i szablon PR są w katalogu głównym `frontend/`, nie w `licznik/`. Pierwszy PR to dobry moment, żeby ustalić, że sekcja „Odpowiedzi na pytania” jest obowiązkowa.
