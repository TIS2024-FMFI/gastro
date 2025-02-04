# Gastro

Tento repozitár obsahuje informácie k údržbe systémov Prítomnosť na Pracovisku a Pracovné cesty. Projekt vznikol v rámci predmetu Tvorba Informačných Systémov na Fakulte Matematiky, Fyziky a Informatiky Univerzity Komenského v Bratislave v akademickom roku 2024/2025.

## Prítomnosť na Pracovisku

Systém Prítomnosť na Pracovisku slúži na evidenciu dochádzky zamestnancov Katedry aplikovanej informatiky.
Systém bol udržiavaný v jazyku PHP `7.4.33` a využíva moduly implementované v jazyku Python `3.9.2`.
Projekt vznikol v rámci predmetu Tvorba informačných systémov na FMFI UK BA v akademickom roku 2017/2018.

### Inštalácia a konfigurácia
Pre inštaláciu z [repozitáru systému](https://github.com/TIS2017/PritomnostNaPracovisku) postupujte, prosím, podľa nasledujúcich inštrukcií.\
Aplikácia na správne fungovanie vyžaduje externú pythonovú knižnicu `openpyxl`, ktorú možno nainštalovať príkazom:
```zsh
python3 -m pip install openpyxl
```

V rámci inštalácie je potrebné stiahnuť si najnovšiu verziu projektu:
```sh
git pull origin main
```

Pre prevádzku v produkcii aplikácia potrebuje mať nastavené tieto parametre v `Aplikácia/include/conn.php`:
```php
$conn = new mysqli("hostname", "username", "password", "database", "port");
```
a tieto parametre v `Aplikácia/include/config.php`:
```php
$main_url = "URL";

$sending_mails = true/false;

// PID používateľov, ktorí schvaľujú prácu doma a pracovné cesty
$request_validators = [PID];

$printer_host = "hostname";
$printer = "printer";
$printer_options = ['page orientation', 'page size'];

$department = 'katedra';
$department_id = 'ID katedry';
$personal_id_prefix = 'PID prefix';
```

Ak existuje databáza podľa projektu z rokov 2017/2018, stačí spustiť:
```sql
ALTER TABLE dochadzka.absence
    ADD COLUMN `cesty_id` bigint unsigned NULL DEFAULT NULL AFTER `confirmation`;
```

Ak databáza ešte nebola vytvorená, je potrebné spustiť skript na jej vytvorenie:
```sql
/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8mb4 */;
/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;


DROP TABLE IF EXISTS `absence`;
CREATE TABLE `absence` (
  `id` int NOT NULL AUTO_INCREMENT,
  `user_id` int NOT NULL,
  `date_time` date NOT NULL,
  `from_time` time DEFAULT NULL,
  `to_time` time DEFAULT NULL,
  `description` text CHARACTER SET utf8mb3 COLLATE utf8mb3_slovak_ci NOT NULL,
  `type` int NOT NULL,
  `insert_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `public` int NOT NULL DEFAULT '1',
  `confirmation` int NOT NULL DEFAULT '1',
  `cesty_id` bigint unsigned DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `user_id_2` (`user_id`,`date_time`),
  KEY `user_id` (`user_id`),
  CONSTRAINT `absence_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `users` (`id`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB AUTO_INCREMENT=16052 DEFAULT CHARSET=utf8mb3 COLLATE=utf8mb3_slovak_ci;

DROP TABLE IF EXISTS `deadlines`;
CREATE TABLE `deadlines` (
  `year` int NOT NULL,
  `month` int NOT NULL,
  `day` int NOT NULL,
  UNIQUE KEY `year` (`year`,`month`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb3 COLLATE=utf8mb3_slovak_ci;

DROP TABLE IF EXISTS `holidays`;
CREATE TABLE `holidays` (
  `id` int NOT NULL AUTO_INCREMENT,
  `date_time` date NOT NULL,
  `description` varchar(255) CHARACTER SET utf8mb3 COLLATE utf8mb3_slovak_ci NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `date_time` (`date_time`)
) ENGINE=InnoDB AUTO_INCREMENT=112 DEFAULT CHARSET=utf8mb3 COLLATE=utf8mb3_slovak_ci;

DROP TABLE IF EXISTS `holidays_budget`;
CREATE TABLE `holidays_budget` (
  `user_id` int NOT NULL,
  `num` decimal(5,2) NOT NULL,
  `year` int NOT NULL,
  UNIQUE KEY `user_id` (`user_id`,`year`),
  KEY `user_id_2` (`user_id`),
  CONSTRAINT `uid` FOREIGN KEY (`user_id`) REFERENCES `users` (`id`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb3 COLLATE=utf8mb3_slovak_ci;

DROP TABLE IF EXISTS `users`;
CREATE TABLE `users` (
  `id` int NOT NULL AUTO_INCREMENT,
  `personal_id` int DEFAULT NULL,
  `username` varchar(255) CHARACTER SET utf8mb3 COLLATE utf8mb3_slovak_ci NOT NULL,
  `password` varchar(255) CHARACTER SET utf8mb3 COLLATE utf8mb3_slovak_ci NOT NULL,
  `name` varchar(63) CHARACTER SET utf8mb3 COLLATE utf8mb3_slovak_ci NOT NULL,
  `surname` varchar(63) CHARACTER SET utf8mb3 COLLATE utf8mb3_slovak_ci NOT NULL,
  `email` varchar(127) CHARACTER SET utf8mb3 COLLATE utf8mb3_slovak_ci NOT NULL,
  `status` int NOT NULL,
  `last_login` datetime DEFAULT NULL,
  `token` varchar(127) CHARACTER SET utf8mb3 COLLATE utf8mb3_slovak_ci DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `username` (`username`),
  UNIQUE KEY `email` (`email`),
  UNIQUE KEY `personal_id` (`personal_id`)
) ENGINE=InnoDB AUTO_INCREMENT=211 DEFAULT CHARSET=utf8mb3 COLLATE=utf8mb3_slovak_ci;



/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;
```

Nakoniec sa aplikácia spúšťa lokálne príkazom:
```sh
php -S localhost:8000
```

Ďalšie informácie k tomuto systému sú dostupné v [samostatnom repozitári](https://github.com/TIS2017/PritomnostNaPracovisku).

## Pracovné cesty

Systém Pracovné cesty slúži na evidenciu pracovných ciest pre Katedru aplikovanej informatiky FMFI UK BA.
Systém bol udržiavaný v jazyku PHP `8.2.27` vo frameworku Laravel `10.44.0`

Projekt vznikol v rámci predmetu Tvorba informačných systémov na FMFI UK BA v akademickom roku 2023/2024.

### Inštalácia a konfigurácia
Pre inštaláciu z [repozitáru systému](https://github.com/TIS2023-FMFI/pracovne-cesty) postupujte, prosím, podľa nasledujúcich inštrukcií.\
Aplikácia pre správne fungovanie vyžaduje balík `texlive`, ideálne `texlive-full` 
([web](https://www.tug.org/texlive/)).

Tento balík možno v Linuxe nainštalovať napríklad pomocou package managera `apt` nasledovne:
```sh
apt-get install texlive-full
```
V závislosti od platformy môže byť potrebné nainštalovať balík `php-bcmath`([web](https://www.php.net/manual/en/book.bc.php)):
```sh
apt-get install php-bcmath
```

V rámci inštalácie je potrebné stiahnuť si najnovšiu verziu projektu:
```sh
git pull origin main
```

Následne aplikáciu treba nakonfigurovať pre dané prostredie pomocou `src/.env`:
```sh
cd src
cp .env.example .env
```
Pre prevádzku v produkcii aplikácia potrebuje mať nastavené tieto parametre:
```dotenv
APP_NAME="Pracovné cesty"
APP_ENV=production
APP_DEBUG=false
APP_URL=

# DB connection for this app
DB_CONNECTION=mysql
DB_HOST=
DB_PORT=
DB_DATABASE=
DB_USERNAME=
DB_PASSWORD=

# DB connection for Pritomnost
PRITOMNOST_DB_CONNECTION=mysql
PRITOMNOST_DB_HOST=
PRITOMNOST_DB_PORT=
PRITOMNOST_DB_DATABASE=
PRITOMNOST_DB_USERNAME=
PRITOMNOST_DB_PASSWORD=

# Mail service configuration
MAIL_MAILER=smtp
MAIL_HOST=
MAIL_PORT=
MAIL_USERNAME=
MAIL_PASSWORD=
MAIL_ENCRYPTION=
MAIL_FROM_ADDRESS=
MAIL_FROM_NAME=
```

Pre správne fungovanie integrácie aplikácií je potrebné nastaviť atribút `REQUEST_VALIDATOR_ID` v súbore `src/app/Models/PritomnostUser.php` na rovnakú hodnotu, aká je nastavená v premennej `$request_validators` v súbore `Aplikácia/include/config.php` v systéme Prítomnosť na pracovisku.

```php
private const REQUEST_VALIDATOR_ID = PID;
```

Kvôli generovaniu PDF dokumentov je potrebné vytvoriť v adresári `src/storage/app/` adresár `pdf_exports`:
```sh
mkdir src/storage/app/pdf_exports
```

Ďalej je potrebné nainštalovať závislosti:
```sh
composer install --optimize-autoloader --no-dev
```

Pri problémoch s generovaním PDF dokumentov, aj keď by sa mal balík nainštalovať v predošlom kroku, môže pomôcť explicitná inštalácia príkazom:
```sh
composer require ismaelw/laratex
```

Ak existuje databáza podľa projektu z rokov 2023/2024, stačí spustiť:
```sql
ALTER TABLE cesty.countries
    ADD COLUMN `trips_count` bigint unsigned NOT NULL DEFAULT '0' AFTER `name`;

ALTER TABLE cesty.spp_symbols
    ADD COLUMN `agency` VARCHAR(100) AFTER `status`,
    ADD COLUMN `acronym` VARCHAR(10) AFTER `agency`,
    DROP COLUMN `fund`;

UPDATE cesty.business_trips
    SET `state` = `state` - 1
    WHERE `state` >= 2;

ALTER TABLE cesty.business_trips
    ADD COLUMN spp_symbol_id_2 BIGINT UNSIGNED NULL AFTER reimbursement_id,
    ADD COLUMN spp_symbol_id_3 BIGINT UNSIGNED NULL AFTER spp_symbol_id_2,
    ADD COLUMN amount_eur SMALLINT NULL AFTER spp_symbol_id_3,
    ADD COLUMN amount_eur_2 SMALLINT NULL AFTER amount_eur,
    ADD COLUMN amount_eur_3 SMALLINT NULL AFTER amount_eur_2,
    ADD COLUMN is_template BOOLEAN NOT NULL DEFAULT FALSE AFTER conclusion,
    ADD CONSTRAINT business_trips_spp_symbol_id_2_foreign 
        FOREIGN KEY (spp_symbol_id_2) REFERENCES spp_symbols(id),
    ADD CONSTRAINT business_trips_spp_symbol_id_3_foreign 
        FOREIGN KEY (spp_symbol_id_3) REFERENCES spp_symbols(id);

ALTER TABLE cesty.users
    ADD COLUMN iban VARCHAR(34) NULL AFTER remember_token,
    ADD COLUMN spp_user_type SMALLINT UNSIGNED NULL AFTER iban;

ALTER TABLE cesty.users
    ADD COLUMN personal_id_dochadzka int NULL AFTER updated_at;

ALTER TABLE cesty.business_trips
    # advance => participation (vlozne)
    RENAME COLUMN advance_expense_id TO participation_expense_id,
    # allowance => advance (zaloha)
    RENAME COLUMN allowance_expense_id TO advance_expense_id;
ALTER TABLE cesty.business_trips
    # allowance (vreckove)
    ADD COLUMN allowance_expense_id BIGINT UNSIGNED UNIQUE NULL AFTER advance_expense_id;
ALTER TABLE cesty.business_trips
    # insurance (poistenie)
    ADD COLUMN insurance_expense_id BIGINT UNSIGNED UNIQUE NULL AFTER participation_expense_id;
```

Treba ručne prepísať grantee na idčka z users tabulky a potom spustiť.
```sql
ALTER TABLE cesty.spp_symbols
    MODIFY COLUMN grantee BIGINT UNSIGNED NOT NULL,
    ADD CONSTRAINT fk_spp_symbols_grantee
    FOREIGN KEY (grantee) REFERENCES users(id)
    ON DELETE CASCADE ON UPDATE CASCADE;
```

Ak databáza ešte nebola vytvorená, je potrebné spustiť databázové migrácie:
```sh
php artisan migrate
php artisan db:seed
```

Po rozsiahlej aktualizácii systému je vhodné vyčistiť všetky vyrovnávacie pamäte príkazmi:
```sh
php artisan cache:clear
php artisan route:clear
php artisan view:clear
php artisan config:clear
```
prípadne príkazom:
```sh
php artisan optimize:clear
```

Na spustenie plánovaných úloh, ako je napríklad posielanie varovných emailov, je potrebné spustiť scheduler:
```sh
php artisan schedule:work
```

Voliteľne je možné v rámci optimalizácie uložiť aktuálnu konfiguráciu do cache:
```sh
php artisan config:cache
php artisan event:cache
php artisan route:cache
php artisan view:cache
```

Nakoniec sa aplikácia spúšťa lokálne príkazom:
```sh
php artisan serve
```

Všetky požiadavky z webového servera by mali byť smerované na `src/public/index.php`.

Ďalšie detaily ku konfigurácii je možné nájsť v [Laravel dokumentácii](https://laravel.com/docs/10.x/deployment).\
Ďalšie informácie k tomuto systému sú dostupné v [samostatnom repozitári](https://github.com/TIS2023-FMFI/pracovne-cesty).
