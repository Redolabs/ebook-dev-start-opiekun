# 02-javascript-react: materiały opiekuna

Rozwiązania zadań z kursu „JavaScript, TypeScript i React”, po jednym pliku na rozdział, z akapitem „Na co patrzeć przy review” przy każdym zadaniu. Szkice review uczniów lądują w `review/<login>/`.

Różnice względem kursu PHP, które wpływają na review:

- Każdy rozdział ma trzy zadania, trzecie zawsze „wyjaśnij własnymi słowami”. Brak odpowiedzi na pytania to **Request changes**, nie uwaga na marginesie. To świadoma decyzja po doświadczeniach z kursem PHP, gdzie kod wyprzedzał rozumienie.
- Przed commitem uczeń ma uruchamiać `npm run lint`. PR z błędami lintera wraca bez czytania reszty.
- Repozytorium ucznia to `frontend` z katalogami `licznik/`, `fiszki/`, `repertuar/`, `kino/`. Każdy katalog ma własny `package.json`; `npm install` i `npm run dev` uruchamiasz w katalogu projektu, nie w głównym.
- Rozdział 0 oddawany jest jako katalog `js-rozgrzewka/` bez Gita, z plikiem `odpowiedzi.md`.

Skill `/review` jest napisany pod kurs PHP (ścieżki, gałęzie, `php -l`). Do kursu JS trzeba go rozszerzyć: tabela gałęzi z rozdziałów, `npm run lint` zamiast `php -l`, `npm run dev` zamiast `php -S`, sprawdzanie sekcji „Odpowiedzi na pytania” w opisie PR. To zadanie na moment, gdy Kuba dojdzie do rozdziału 1.

Gałęzie z książki:

| Gałąź | Rozdział |
| --- | --- |
| katalog `js-rozgrzewka/` | 0 |
| `feature/setup` | 1 |
| `feature/countdown-dom` | 2 |
