# Zaległe uwagi do przeniesienia do następnego review

Przy najbliższym review (PR #17, rozdział 17): dołącz poniższe do sekcji „Komentarz do PR”, sprawdź na gałęzi PR, co zostało już zrobione, i usuń z tego pliku punkty załatwione. Gdy plik będzie pusty, usuń go.

## Z PR #16 (Tailwind), stan na `main` c582285

**Przeniesione do review PR #17 (punkty 5 i 6, 2026-10-30).** Przy sprawdzaniu poprawek PR #17 zobacz, czy są zrobione, i wtedy usuń ten plik.

Tekst do ucznia (do wklejenia jako osobna podsekcja, np. „### Zaległe z PR #16”):

> Z poprawek do PR #16 zrobiłeś najważniejsze: XSS w `404.php`, powrót „Edytuj” i filtry przy pustej liście. Dzięki. Kilka rzeczy zostało, zrób je na tej gałęzi:
>
> 1. `views/tasks/index.php:56`: „Edytuj” jest gołym `<button>` z `style="display: inline"`, bez klas Tailwinda, więc wygląda inaczej niż „Zrobione” i „Usuń” obok. Ostyluj go tak jak sąsiednie przyciski (najprościej jako `<a>`).
> 2. Zadanie 1 z rozdziału 16: `tasks/show.php`, `admin/_nav.php` (dalej `|` i `<hr>`), `admin/edit.php` (dalej `style="color: red"`) i strony w `views/errors/` są bez Tailwinda. Wejdź na `/tasks/1`, `/admin/users/1/edit` i `/nie-ma` i porównaj z resztą aplikacji.
> 3. Zadanie 3 z rozdziału 16 (tryb ciemny): jest tylko `darkMode: 'class'` w `layout.php`. Brakuje klasy `dark` na `<html>` i klas `dark:` w widokach. Sprawdzenie: `grep -rn 'bg-white' views/ | grep -v 'dark:'` ma nic nie wypisać.
> 4. Opis PR: dodawaj „Jak sprawdzić” z listą stron do przeklikania. W PR #16 tego zabrakło i stąd zgubiony „Edytuj”.

Dla opiekuna: drugi PR z rzędu, w którym poprawił tylko najgłośniejsze uwagi, a resztę pominął bez słowa i zmergował. Jeśli punkty 2–4 znowu przepadną, omówić na żywo.
