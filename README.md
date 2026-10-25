# ebook-dev-start-opiekun

Materiały dla opiekuna do kursów z repozytorium `ebook-dev-start`. To repo jest prywatne i uczeń nie ma do niego dostępu.

- [01-php-miniframework/](01-php-miniframework/README.md): rozwiązania wszystkich zadań, po jednym pliku na rozdział, plus miejsce na notatki z przejścia z uczniem.
- [pomysly-na-kolejne-ebooki.md](pomysly-na-kolejne-ebooki.md): plany na kursy o React i Pythonie, dodatek o Gicie w zespole, Docker, SQL.
- [uczniowie.md](uczniowie.md): lista uczniów i ich repozytoriów, z której korzysta skill `/review`.
- [.claude/skills/review/](.claude/skills/review/SKILL.md): skill Claude Code, który przygotowuje review pull requesta ucznia.

## Review pull requestów przez Claude Code

Uczeń oddaje każdy rozdział przez pull request w swoim repozytorium (rytuał z rozdziału 1 książki). Review przygotowuje Claude Code, a opiekun je czyta, poprawia i wysyła.

Wymagania: `gh` zainstalowane i zalogowane (`brew install gh && gh auth login`), zaproszenie do repo ucznia przyjęte, a repo z książką `ebook-dev-start` sklonowane obok tego repo (skill czyta z niego treść rozdziałów).

Użycie, w katalogu tego repo:

```
claude
/review 4          # najnowszy otwarty PR ucznia, rozdział 4
/review 4 12       # PR numer 12
/review 0 ~/obiekty  # rozdział 0 z lokalnego katalogu, bez GitHuba
```

Skill czyta rozdział z książki, akapity „Na co patrzeć przy review” z rozwiązań, pobiera PR, uruchamia `php -l`, testy i adresy z „Sprawdź, czy działa”, a potem pokazuje szkic review. Szkic ląduje w `01-php-miniframework/review/<login>/`. Na GitHuba trafia dopiero po Twojej decyzji, jako komentarz albo „Request changes”. „Approve” i merge robisz sam na GitHubie.

Konwencja: ścieżki katalogów są takie same jak w `ebook-dev-start`, więc rozwiązania do `01-php-miniframework` leżą pod tą samą nazwą co książka.
