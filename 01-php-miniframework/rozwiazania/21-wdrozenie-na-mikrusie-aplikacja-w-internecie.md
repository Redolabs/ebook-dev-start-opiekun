# Rozdział 21: wdrożenie na Mikrusie: aplikacja w internecie

## Zadanie 1: strona `/health` i monitoring

Plik `app/Controllers/HealthController.php`:

```php
<?php

declare(strict_types=1);

namespace App\Controllers;

use Core\Controller;
use Core\Database;
use Core\Request;
use Core\Response;

class HealthController extends Controller
{
    public function show(Request $request): Response
    {
        try {
            Database::connection()->query('SELECT 1');
        } catch (\Throwable $e) {
            error_log('Health: baza nie odpowiada: ' . $e->getMessage());

            return $this->plain('DB ERROR', 503);
        }

        return $this->plain('OK', 200);
    }

    private function plain(string $body, int $status): Response
    {
        return (new Response($body, $status))
            ->header('Content-Type', 'text/plain; charset=UTF-8')
            ->header('Cache-Control', 'no-store');
    }
}
```

W `routes.php`, bez middleware:

```php
$router->get('/health', [HealthController::class, 'show']);
```

plus `use App\Controllers\HealthController;` na górze pliku.

Koniec `deploy.sh`:

```bash
echo "==> Sprawdzam /health"
APP_URL=$(grep '^APP_URL=' .env | cut -d= -f2-)
curl -fsS --max-time 10 "$APP_URL/health"
echo

echo "==> Gotowe: $(git log -1 --oneline)"
```

`curl -f` kończy się kodem błędu przy statusie 400 i wyższym, więc `set -e` przerwie skrypt, a workflow „Wdrożenie” zaświeci się na czerwono. `-sS` wycisza pasek postępu, ale zostawia komunikaty błędów. Na Mikrusie żądanie idzie z serwera na zewnątrz, do proxy subdomeny, i wraca do Twojego nginx. To dobrze: sprawdza całą drogę, którą idzie użytkownik, łącznie z HTTPS.

Monitor w UptimeRobot: typ „Keyword”, URL `https://kuba-zadania.byst.re/health`, słowo `OK`, „alert when keyword not exists”, interwał 5 minut, kontakt e-mail. Test: `sudo systemctl stop mariadb`, po kilku minutach mail „Monitor is DOWN”, `sudo systemctl start mariadb`, mail „Monitor is UP”.

**Na co patrzeć przy review:**

- Trasa bez `auth`: monitoring się nie loguje. Bez `guest` też, bo zalogowany admin też może chcieć ją otworzyć.
- Odpowiedź nie zdradza nic: ani nazwy bazy, ani wersji PHP, ani treści wyjątku. Treść wyjątku idzie do `error_log()` (czyli do logu nginx), nie do przeglądarki.
- `catch (\Throwable $e)` w tym jednym miejscu jest celowy: wyjątek z bazy to tu oczekiwana odpowiedź („baza leży”), nie awaria do obsłużenia przez `ErrorHandler`. Gdyby uczeń tego nie łapał, dostałby 500 zamiast 503 i stronę HTML zamiast `DB ERROR`. Monitor ze słowem kluczowym i tak wykryłby problem, ale 503 to poprawny status.
- `Cache-Control: no-store`: żaden pośrednik (także proxy Mikrusa) nie może zapamiętać „OK” sprzed awarii.
- W `deploy.sh` `APP_URL` czytany z `.env`, nie wpisany na sztywno. Zapytaj, co się stanie, gdy `/health` odpowie 503 po wdrożeniu (skrypt kończy się błędem, ale nowy kod już działa: to alarm, nie wycofanie zmian).
- Pytanie kontrolne: czym różni się od `/api/health` z [rozdziału 19](../../../ebook-dev-start/01-php-miniframework/rozdzialy/19-e-maile-przez-api-resend.md)? Tamto pokazywało konfigurację maili, to celowo nie pokazuje niczego, bo jest publiczne i odpytywane przez obcą usługę.

## Zadanie 2: tryb przerwy technicznej

`public/index.php` po zmianie (pełny plik):

```php
<?php

declare(strict_types=1);

// Przerwa techniczna. Sprawdzana przed autoloaderem, .env, bazą i sesją,
// bo w czasie wdrożenia każda z tych rzeczy może być w połowie zmiany.
if (file_exists(__DIR__ . '/../storage/maintenance')) {
    http_response_code(503);
    header('Retry-After: 60');
    header('Content-Type: text/html; charset=UTF-8');
    require __DIR__ . '/../views/errors/maintenance.php';
    exit;
}

require __DIR__ . '/../vendor/autoload.php';

use App\Models\User;
use Core\Auth;
use Core\Env;
use Core\ErrorHandler;
use Core\Request;
use Core\Router;
use Core\Session;

// APP_ENV=testing ustawia serwer do testów: ServerTestCase albo Playwright
$envFile = getenv('APP_ENV') === 'testing' ? '.env.testing' : '.env';
Env::load(__DIR__ . '/../' . $envFile);
ErrorHandler::register(Env::get('APP_DEBUG') === 'true');

Session::start();
Auth::useModel(User::class);

$router = new Router();
require __DIR__ . '/../routes.php';

$response = $router->dispatch(Request::fromGlobals());
$response->send();
```

Plik `views/errors/maintenance.php` (pełna strona, bez layoutu, bez zmiennych i bez `e()`, bo helpery jeszcze nie są załadowane):

```php
<!DOCTYPE html>
<html lang="pl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Przerwa techniczna</title>
    <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gray-100 min-h-screen flex items-center justify-center">
    <div class="bg-white rounded-lg shadow p-8 max-w-md text-center">
        <h1 class="text-2xl font-bold mb-2">Trwa aktualizacja</h1>
        <p class="text-gray-600">Wgrywamy nową wersję. Wróć za minutę.</p>
    </div>
</body>
</html>
```

`deploy.sh`:

```bash
#!/usr/bin/env bash
# Wdrożenie nowej wersji. Uruchamiaj na serwerze jako deploy: ./deploy.sh
set -euo pipefail

cd "$(dirname "$0")"

MAINTENANCE=storage/maintenance

trap 'echo "!! Wdrożenie przerwane. Przerwa techniczna zostaje WŁĄCZONA." >&2
      echo "!! Napraw problem i uruchom ./deploy.sh jeszcze raz albo: rm $MAINTENANCE" >&2' ERR

echo "==> Pobieram kod z GitHuba (aplikacja dalej działa)"
git fetch origin main

echo "==> Włączam przerwę techniczną"
touch "$MAINTENANCE"

echo "==> Podmieniam kod"
git merge --ff-only origin/main

echo "==> Instaluję zależności"
composer install --no-dev --optimize-autoloader --no-interaction

echo "==> Przeładowuję PHP-FPM"
sudo systemctl reload php8.3-fpm

echo "==> Wyłączam przerwę techniczną"
rm -f "$MAINTENANCE"

echo "==> Gotowe: $(git log -1 --oneline)"
```

W `.gitignore`:

```
storage/maintenance
```

**Na co patrzeć przy review:**

- Sprawdzenie stoi **przed** `require vendor/autoload.php`. W czasie `composer install` autoloader jest przepisywany, więc strona przerwy nie może od niego zależeć. Dlatego `require` zwykłego pliku zamiast `View::render()` i `Response`, i dlatego widok nie używa `e()` ani layoutu (layout woła `Auth::check()`, czyli bazę). Jeśli uczeń użył `View::renderPartial()` po autoloaderze, to działa w praktyce, ale zapytaj, co się stanie w tej sekundzie, gdy `vendor/` jest w połowie podmiany.
- Status 503 z `Retry-After`, nie 200: wyszukiwarki i monitoring z zadania 1 wiedzą, że to przerwa, a nie nowa treść strony.
- Zachowanie przy błędzie: po `set -e` plik zostaje, użytkownicy widzą spokojną stronę przerwy zamiast pół starej, pół nowej aplikacji, a `trap ... ERR` mówi wprost, co zrobić. Wersja z `trap ... EXIT`, która zawsze usuwa plik, też jest do obrony, jeśli uczeń umie powiedzieć, czemu wybrał wpuszczanie ludzi do możliwie zepsutej wersji.
- `git fetch` przed włączeniem przerwy: pobieranie z sieci trwa najdłużej i może się nie udać, a aplikacja w tym czasie działa normalnie. Przerwa obejmuje tylko podmianę plików. Wersja z `touch` przed `git pull` jest poprawna, tylko przerwa trwa dłużej.
- Plik `storage/maintenance` należy do `deploy`, ale `www-data` musi go tylko zobaczyć (`file_exists`), do czego wystarcza prawo czytania katalogu. Plik jest w `.gitignore`, inaczej `git status` na serwerze nie byłby czysty, a `git merge --ff-only` mógłby odmówić.
- Brak furtki dla admina w czasie przerwy jest w porządku na tym etapie. Laravel ma do tego `php artisan down --secret=...`; można zapytać, jak by to zrobić (ciasteczko albo parametr w adresie sprawdzany przed 503).

## Zadanie 3: kopia poza serwerem albo rotacja logów

**Wariant (a).** Plik `~/pobierz-backup.sh` na WSL:

```bash
#!/usr/bin/env bash
# Pobiera najnowszą kopię bazy z serwera na WSL. Trzyma 30 dni.
set -euo pipefail

# Skrót z ~/.ssh/config: srv20.mikr.us, port 10123, użytkownik deploy
SERVER=mikrus
DEST="$HOME/backupy-miniframework"

mkdir -p "$DEST"
chmod 700 "$DEST"

LATEST=$(ssh "$SERVER" 'ls -1t backups/miniframework-*.sql.gz | head -n 1')
if [ -z "$LATEST" ]; then
    echo "Na serwerze nie ma żadnej kopii" >&2
    exit 1
fi

scp "$SERVER:$LATEST" "$DEST/"
gzip -t "$DEST/$(basename "$LATEST")"

find "$DEST" -name 'miniframework-*.sql.gz' -mtime +30 -delete

echo "$(date '+%F %T') Pobrano $(basename "$LATEST")"
```

`chmod +x ~/pobierz-backup.sh`, uruchomienie ręczne albo w crontabie na WSL, np. `0 20 * * * /home/kuba/pobierz-backup.sh >> /home/kuba/backupy-miniframework/pobieranie.log 2>&1`. Uwaga: na WSL cron działa tylko wtedy, gdy WSL jest uruchomiony i usługa `cron` wystartowała (`sudo service cron start` albo systemd włączony w `/etc/wsl.conf`). Dlatego ręczne uruchamianie raz w tygodniu jest tu uczciwszą odpowiedzią. Skrót `mikrus` działa z crona, bo cron uruchamia skrypt jako `kuba` i SSH czyta jego `~/.ssh/config`.

Wersja z `rclone` (chętni): `rclone config` raz, interaktywnie, na serwerze (zdalne miejsce np. o nazwie `gdrive`), potem na końcu `backup-db.sh` linia `rclone copy "$FILE" gdrive:miniframework-backup/`. Alternatywa bez chmury: Strych Mikrusa (200 MB, osobny serwer, opis na https://wiki.mikr.us/strych_backupy).

**Wariant (b).** Plik `/etc/logrotate.d/miniframework`:

```
/var/www/miniframework/storage/logs/*.log {
    weekly
    rotate 8
    compress
    delaycompress
    missingok
    notifempty
    su www-data www-data
    create 0664 www-data www-data
}
```

```bash
sudo logrotate -d /etc/logrotate.d/miniframework
sudo logrotate -f /etc/logrotate.d/miniframework
ls -l /var/www/miniframework/storage/logs
```

**Na co patrzeć przy review:**

- (a) Kopia ląduje poza serwerem, który może paść. To jest cały sens zadania. Kopia w innym katalogu tego samego Mikrusa się nie liczy.
- (a) Port i nazwa serwera są w `~/.ssh/config`, nie w skrypcie. Jeśli uczeń wpisał `ssh -p 10123 deploy@srv20.mikr.us` i `scp -P 10123 ...`, to też działa; sprawdź tylko, czy w `scp` jest duże `-P`.
- (a) `gzip -t` sprawdza, że plik nie jest ucięty. To jeszcze nie test odtworzenia, ale łapie najczęstszą awarię (przerwany transfer, pusty plik po błędzie `mysqldump`). Zapytaj, kiedy ostatnio robił prawdziwe odtworzenie z Kroku 7.
- (a) `chmod 700` na katalogu: kopia zawiera dane osobowe użytkowników (e-maile) i hashe haseł. Zapytaj, czy przechowywanie cudzych danych na prywatnym laptopie jest w porządku i co by zrobił, gdyby laptop zginął (szyfrowanie dysku).
- (a) Skrypt nie ma w sobie żadnego hasła: używa tego samego klucza SSH co Kuba.
- (b) `su www-data www-data` jest wymagane: katalog `storage/logs` jest zapisywalny dla grupy innej niż root i bez tej linii logrotate odmówi z komunikatem o niebezpiecznych uprawnieniach katalogu nadrzędnego. Rotacja jako `www-data` może też utworzyć nowy plik z właścicielem `www-data`.
- (b) `create 0664 www-data www-data`: bez tego nowy `app.log` mógłby powstać z właścicielem, do którego php-fpm nie zapisze, i od tej chwili `ErrorHandler` milczy. `delaycompress` zostawia ostatni plik nieskompresowany, bo proces może jeszcze do niego dopisywać.
- (b) `missingok` i `notifempty`: brak `mail.log` na produkcji (tam `MAIL_DRIVER=resend`) i pusty `app.log` nie są błędem.
- (b) Logrotate uruchamia się z timera systemd albo z crona (`/etc/cron.daily/logrotate`). W kontenerze sprawdź, że któryś działa: `systemctl list-timers | grep logrotate`.
