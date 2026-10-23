# Rozdział 10: sesje, logowanie i CSRF

## Zadanie 1: drugi użytkownik

```bash
php -r "echo password_hash('ola12345', PASSWORD_DEFAULT), PHP_EOL;"
```

```sql
INSERT INTO users (name, email, password, role)
VALUES ('Ola', 'ola@example.com', '<wklej hash>', 'user');
```

**Na co patrzeć przy review:** w bazie ma być hash zaczynający się od `$2y$`, nie jawne hasło. Kolumnę `role` można pominąć, bo ma domyślnie `user`. W commicie nie powinno być ani hasła, ani hasha (SQL nie idzie do repozytorium); jeśli uczeń zapisał go w pliku `.sql` w projekcie, poproś o usunięcie hasła z treści.

## Zadanie 2: e-mail `admin@example.com' OR '1'='1`

Bez kodu. `Auth::attempt()` woła `User::first('email', $email)`, które robi `SELECT ... WHERE email = ?` i przekazuje cały napis jako wartość. Baza szuka użytkownika, którego adres to dokładnie ten dziwny napis, nie znajduje go i `attempt()` zwraca `false`. Nawet gdyby zapytanie było podatne, hasło i tak sprawdza `password_verify` w PHP, więc samo „znalezienie” admina nie wystarczyłoby do zalogowania.

**Na co patrzeć przy review:** żeby w ogóle wysłać taki tekst, uczeń musi w DevTools zmienić `type="email"` na `text` (przeglądarka nie przepuści spacji w polu e-mail). Jeśli twierdzi, że „przeglądarka zablokowała atak”, to błędna lekcja: zablokowała wygodę, nie atak. Dobra odpowiedź wymienia oba zabezpieczenia: `?` w zapytaniu i `password_verify`.

## Zadanie 3: kto co widzi

To nie jest zadanie do zrobienia, tylko zapowiedź [rozdziału 12](../../../ebook-dev-start/01-php-miniframework/rozdzialy/12-middleware-i-role.md). Brak kodu. Jeśli uczeń zaczął sam dopisywać sprawdzanie logowania w kontrolerach, powiedz mu, że w [rozdziale 12](../../../ebook-dev-start/01-php-miniframework/rozdzialy/12-middleware-i-role.md) przeniesie to do middleware.
