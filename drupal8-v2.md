---
title: Oppsett av Drupal 8
---

## {{ page.title }}

Denne guiden skal hjelpe deg med å få satt opp dev-instanser av utdanning.no inkludert søk, min.utdanning.no og data.utdanning.no.

Det antas at `uno_beta` er et gyldig  alias for beta-serveren til utdanning.no i oppsettet ditt. Bytt ut dette med fullstendig servernavn eller med tilsvarende for prod-server hvis du laster ned databaser derfra.

### 1. Legg inn domener i /etc/hosts

Legg inn dev.utdanning.no, dev.min.utdanning.no og dev.data.utdanning.no i `/etc/hosts`.

<pre>
127.0.0.1	dev.utdanning.no
127.0.0.1	dev.min.utdanning.no
127.0.0.1	dev.data.utdanning.no
</pre>

### 2. Installer Docker 

- installer Docker. Se [https://docs.docker.com/install/linux/docker-ce/ubuntu/](https://docs.docker.com/install/linux/docker-ce/ubuntu/) for instruksjoner for Ubuntu
- Installer docker-compose: `sudo apt install docker-compose`
- Meld deg inn i docker-gruppa. Etter dette har det vist seg nødvendig å restarte pc-en.
- For Mac: Installer Docker Desktop
- For Mac: Kildekoden må være sjekket ut i en mappe som ligger inne i 'File sharing'-innstillingene til Docker. Hvis ikke så får ikke Mac mountet filsystemet inne i Docker.

### 3. Forbered eventuell lokal Apache

Hvis du har Apache (eller noe annet) kjørende på port 80 så vil det stå i veien for installasjonen vi skal gjøre. Du kan enten stoppe Apache (`sudo service apache2 stop`) eller sette opp dev-miljøet til å kjøre på en annen port, f.eks. port 81. Du må da endre `varnish.ports` i `docker-compose.yml` til `81:80` og bruke det følgende i en vhost-fil til Apache (NB! ikke testet):

<pre>
<VirtualHost *:80>
    ServerName dev.utdanning.no
    ProxyPass "/"  "http://dev.utdanning.no:81/"
    ProxyPassReverse "/"  "http://dev.utdanning.no:81/"
</VirtualHost>
<VirtualHost *:80>
    ServerName dev.min.utdanning.no
    ProxyPass "/"  "http://dev.min.utdanning.no:81/"
    ProxyPassReverse "/"  "http://dev.min.utdanning.no:81/"
</VirtualHost>
<VirtualHost *:80>
    ServerName dev.data.utdanning.no
    ProxyPass "/"  "http://dev.data.utdanning.no:81/"
    ProxyPassReverse "/"  "http://dev.data.utdanning.no:81/"
</VirtualHost>
</pre>

For at dette skal fungere så må proxy og proxy_http-modulene aktiveres:

> sudo a2enmod proxy proxy_http

Hvis du vil gjøre dette og det skaper problemer så er det bare å stoppe Apache, kjøre gjennom installasjonen og så komme tilbake til dette etterpå.


### 4. Sjekke ut dev-prosjektet

Gjør dette fra `/srv`-mappa eller et tilsvarende sted.

- `git clone --recurse-submodules git@github.com:utdanningno/dev.utdanning.no.git --branch development`
- `cd dev.utdanning.no`
- `git submodule foreach --recursive git checkout development`
- `git submodule foreach --recursive git pull`

Vi bruker `development` som utviklingsbranch så arbeid bør gjøres i den eller i feature-brancher utledet fra den.


### 5. Installere kontainere


- `./robo.phar install:utd`
- `./robo.phar install:data`
- `./robo.phar install:ms`

Etter dette så har du databaseoppsett i filene `dbpwfile.txt`, `data_dbpwfile.txt` og `minside_dbpwfile.txt`.

### 6. Eksportere databaser

Du kan enten eksportere databasene ved hjelp av et skript på prod-serveren eller gjøre det med `mysqldump`-kommandoer fra beta (eller prod). beta-databaser er bedre synkronisert med development-branchene våre, prod-databaser har mer oppdatert innhold. Det har også oppstått problemer ved bruk av beta-database for data.utdanning.no så det kan være lurt å hente den fra prod.


#### 6.1 Eksportere databaser fra beta (mysqldump)


- ssh til beta-serveren.
- `sudo -u uno -i`
- `cd /srv/beta.utdanning.no`
- utdanning.no: `mysqldump --column-statistics=0 -h utdanning-dbaas01-beta.iktsenteret.c.bitbit.net uno_beta -u uno -p | gzip > /srv/tmp/beta.utdanning.no.yyyymmdd.sql.gz`
- utdanning.no-datakollektivet: `mysql -h utdanning-dbaas01-beta.iktsenteret.c.bitbit.net uno_data_beta -u uno_data -p -N -e 'show tables like "mv\_%"' | xargs mysqldump --column-statistics=0 -h utdanning-dbaas01-beta.iktsenteret.c.bitbit.net uno_data_beta -u uno_data -p  red_korrigering x_geonorge_kommuner z_undervisningssteder z_fagskole_tilbud z_folkehogskole_tilbud z_so_kravkoder z_organisasjoner z_ssb_nus z_ssb_styrk98 z_so_uh z_uh_tilbud z_vgs_tilbud z_annen_utdanning z_bedrifter z_so_fs z_privat_vgs_tilbud | gzip > /srv/tmp/beta.dk-lite.yyymmdd.sql.gz`
- `cd /srv/beta.min.utdanning.no`
- minside: `mysqldump --column-statistics=0 -h utdanning-dbaas01-beta.iktsenteret.c.bitbit.net minside_beta -u uno -p | gzip > /srv/tmp/beta.min.utdanning.no.yyyymmdd.sql.gz`
- `cd /srv/beta.data.utdanning.no`
- data: `mysqldump --column-statistics=0 -h utdanning-dbaas01-beta.iktsenteret.c.bitbit.net beta_data_utdanning_no -u beta_data_utdanning_no -p | gzip > /srv/tmp/beta.dk-lite.utdanning.no.yyyymmdd.sql.gz`

#### 6.2 Eksportere databaser fra prod (skriptet)

Vi har et skript for database-eksport som per nå bare er tilgjengelig fra en tmp-katalog på prod-serveren.

> cd /srv/tmp/backup-command/

Kjør følgende kommando 4 ganger, en gang for hver database:

> php bkp.php

Dette vil spørre om hvilken database du vil eksportere. Velg nummer og trykk enter. Vi vil eksportere disse databasene:

```
 - 1: utdanning.no - drupal-database
 - 2: utdanning.no - datakollektivet
 - 3: data.utdanning.no
 - 4: min.utdanning.no
```
Etter å ha trykket enter så får du et spørsmål om suffix. Dette kan overses.

Forventede filnavn:

- `/srv/tmp/prod.utdanning.no.YYYYMMDD.sql.gz` <-- dette er utdanning.no sin drupal-database.
- `/srv/tmp/prod.dk-lite.YYYYMMDD.sql.gz` <-- dette er en minimal datakollektivet-database som vil brukes av dev.utdanning.no og dev.data.utdanning.no.
- `/srv/tmp/prod.data.utdanning.no.YYYYMMDD.sql.gz` <-- data.utdanning.no sin drupal-database.
- `/srv/tmp/prod.min.utdanning.no.YYYYMMDD.sql.gz` <-- min.utdanning.no sin drupal-database.

Hvis filene allerede eksisterer så vil skriptet legge til et numerisk suffiks for å unngå å overskrive eksisterende filer. F.eks. `prod.utdanning.no.YYYYMMDD-2.sql.gz`



### 7. Last ned databasene

Legg dem i rot-katalogen til prosjektet. Det er der importkommandoene forventer å finne dem.

Gå til nevne rot-katalog og gjør dette for alle 4 database-dumpene:

- `scp uno-beta:/srv/tmp/filnavn.sql.gz .`


### 8. Forberede import av databaser


#### 8.1 Alternativ 1: Manuelle endringer i konfigurasjonsfiler


##### dev.utdanning.no

Endringer skal gjøres i `source/utdanning.no/web/sites/default/settings.php` og du behøver verdier fra fila `dbpwfile.txt`. Du må også lage et passord som skal brukes for tilgang til datakollektivet-databasen. Husk dette passordet, det vil også behøves til databaseimporten.

Legg inn en tilfeldig tekst i `$settings['hash_salt']`, f.eks.

> $settings['hash_salt'] = 'skldzfhnlcahegmhrmdlckggmaudhrcghugl84y8gae4cgmloz8glmo8';

Legg inn dette nederst i settings-fila:

<pre>
$databases['default']['default'] = [
  'database' => 'Database fra dbpwfile.txt',
  'username' => 'User fra dbpwfile.txt',
  'password' => 'Password fra dbpwfile.txt',
  'host' => 'Host fra dbpwfile.txt',
  'port' => '3306',
  'driver' => 'mysql',
  'prefix' => '',
  'collation' => 'utf8mb4_danish_ci',
];
$databases['datakollektivet']['default'] = [
  'database' => 'datakollektivet',
  'username' => 'uno_data',
  'password' => 'Et passord du finner på selv',
  'host' => 'db',
  'port' => '3306',
  'driver' => 'mysql',
  'prefix' => '',
  'collation' => 'utf8mb4_danish_ci',
];

$settings['config_sync_directory'] = '../config/sync';

$config['config_split.config_split.dev']['status'] = 1;
$config['config_split.config_split.alfa']['status'] = 0;
$config['config_split.config_split.beta']['status'] = 0;
$config['config_split.config_split.prod']['status'] = 0;
</pre>

##### dev.min.utdanning.no

Endringer skal gjøres i `source/min.utdanning.no/web/sites/default/settings.php` og du behøver verdier fra fila `minside_dbpwfile.txt`.

Legg inn en tilfeldig tekst i `$settings['hash_salt']`, f.eks.

> $settings['hash_salt'] = 'skldzfhnlcahegmhrmdlckggmaudhrcghugl84y8gae4cgmloz8glmo8';

Legg inn dette nederst i settings-fila:

<pre>
$databases['default']['default'] = [
  'database' => 'Database fra minside_dbpwfile.txt',
  'username' => 'User fra minside_dbpwfile.txt',
  'password' => 'Password fra minside_dbpwfile.txt',
  'host' => 'Host fra minside_dbpwfile.txt',
  'port' => '3306',
  'driver' => 'mysql',
  'prefix' => '',
  'collation' => 'utf8mb4_danish_ci',
];

$settings['config_sync_directory'] = '../config/sync';

$config['config_split.config_split.dev']['status'] = 1;
$config['config_split.config_split.alfa']['status'] = 0;
$config['config_split.config_split.beta']['status'] = 0;
$config['config_split.config_split.prod']['status'] = 0;
</pre>


##### dev.data.utdanning.no

Endringer skal gjøres i `source/data.utdanning.no/web/sites/default/settings.php`.

Legg inn database-adapteren (uten å endre passord) fra `source/utdanning.no/web/sites/default/settings.php` som har navn `datakollektivet`. Denne databasen blir delt mellom dev.utdanning.no og dev.data.utdanning.no.

<pre>
$databases['datakollektivet']['default'] = [
  'database' => 'datakollektivet',
  'username' => 'uno_data',
  'password' => 'Et passord du finner på selv',
  'host' => 'db',
  'port' => '3306',
  'driver' => 'mysql',
  'prefix' => '',
  'collation' => 'utf8mb4_danish_ci',
];
</pre>


#### 8.2 Alternativ 2: Via nettleseren


Dette kan gjøres ved å gå til dev-nettstedene dine:

- http://dev.utdanning.no
- http://dev.min.utdanning.no
- http://dev.data.utdanning.no (ser ikke ut som om det er nødvendig)

Databasekonfigurasjon som installasjonen ber om ligger i filene `dbpwfile.txt`, `minside_dbpwfile.txt` og `data_dbpwfile.txt`. Dette tar seg av databasekonfigurasjonen til Drupal-databasene i `settings.php`-fila. Se eksemplene over for å få inn `hash_salt`, `config_sync` og `config_split` i settings.php-filene. Lim også inn adaptere for datakollektivet-databasen (se dev.utdanning.no over) i `source/utdanning.no/web/sites/default/settings.php` og `source/data.utdanning.no/web/sites/default/settings.php`.


### 9. Importere databaser

Kjør følgende kommandoer fra rot-katalogen til prosjektet (`/srv/dev.utdanning.no`).

#### 9.1 Datakollektivet (dev.utdanning.no)

> ./robo.phar dbcustom datakollektivet uno_data beta.dk-lite.yyyymmdd.sql.gz

Denne kommandoen ber deg om et passord. Bruk passordet du la inn i oppsettet til datakollektivet-databasen til dev.utdanning.no.

#### 9.2 dev.utdanning.no

> ./robo.phar db:restore-utd beta.utdanning.no.yyyymmdd.sql.gz

#### 9.3 min.utdanning.no

> ./robo.phar db:restore-ms beta.min.utdanning.no.yyyymmdd.sql.gz

#### 9.4 data.utdanning.no

> ./robo.phar db:restore-data beta.data.utdanning.no.yyyymmdd.sql.gz


### 10. Avslutning

Litt opprydning for å blidgjøre Drupal og for å synkronisere dev-nettstedene med siste status i kodebasene.

<pre>
./robo.phar drush:utd cr
./robo.phar drush:utd cim
./robo.phar drush:utd cr
</pre>

<pre>
./robo.phar drush:data cr
./robo.phar drush:data cim
./robo.phar drush:data cr
</pre>

<pre>
./robo.phar drush:ms cr
./robo.phar drush:ms cim
./robo.phar drush:ms cr
</pre>

Hvis `drush cim` foreslår omfattende endringer så tyder det på at noe har blitt feil. Endringene du skal få er i hovedsak forskjeller mellom beta/prod- og dev-miljø.

Sjekk deretter at følgende nettsteder er operative:

- [http://dev.utdanning.no](http://dev.utdanning.no)
- [http://dev.min.utdanning.no](http://dev.min.utdanning.no)
- [http://dev.data.utdanning.no](http://dev.data.utdanning.no)

