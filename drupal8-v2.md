---
title: Oppsett av Drupal 8 (v2, komplett?)
---

## {{ page.title }}

Denne guiden skal hjelpe deg med å få satt opp dev-instanser av utdanning.no, min.utdanning.no og data.utdanning.no

### Legg inn domener i /etc/hosts

Legg inn dev.utdanning.no, dev.min.utdanning.no og dev.data.utdanning.no i /etc/hosts

<pre>
127.0.0.1	dev.utdanning.no
127.0.0.1	dev.min.utdanning.no
127.0.0.1	dev.data.utdanning.no
</pre>

### Installer Docker 

- installer Docker. Se [https://docs.docker.com/install/linux/docker-ce/ubuntu/](https://docs.docker.com/install/linux/docker-ce/ubuntu/) for instruksjoner for Ubuntu
- Installer docker-compose: `sudo apt install docker-compose`
- Meld deg inn i docker-gruppa. Etter dette har det vist seg nødvendig å restarte pc-en.
- For Mac: Installer Docker Desktop
- For Mac: Kildekoden må være sjekket ut i en mappe som ligger inne i 'File sharing'-innstillingene til Docker. Hvis ikke så får ikke Mac mountet filsystemet inne i Docker.

### Forbered eventuell lokal Apache

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

Hvis du vil gjøre dette og det skaper problemer så er det bare å stoppe Apache, kjøre gjennom installasjonen og så komme tilbake til dette etterpå.


### Sjekke ut dev-prosjektet

- `git clone --recurse-submodules git@github.com:utdanningno/dev.utdanning.no.git --branch development`
- `cd dev.utdanning.no`
- `git submodule foreach --recursive git checkout development`
- `git submodule foreach --recursive git pull`

Vi bruker `development` som utviklingsbranch så arbeid bør gjøres i den eller i feature-brancher utledet fra den.


### Installere kontainere


- `./robo.phar install:utd`
- `./robo.phar install:data`
- `./robo.phar install:ms`

Etter dette så har du databaseoppsett i filene `dbpwfile.txt`, `data_dbpwfile.txt` og `minside_dbpwfile.txt`.


### Eksportere databaser fra beta

- ssh til beta-serveren.
- `sudo -u uno -i`
- `cd /srv/beta.utdanning.no`
- utdanning.no: `mysqldump --column-statistics=0 -h utdanning-dbaas01-beta.iktsenteret.c.bitbit.net uno_beta -u uno -p | gzip > /srv/tmp/beta.utdanning.no.yyyymmdd.sql.gz`
- utdanning.no-datakollektivet: `mysql -h utdanning-dbaas01-beta.iktsenteret.c.bitbit.net uno_data_beta -u uno_data -p -N -e 'show tables like "mv\_%"' | xargs mysqldump --column-statistics=0 -h utdanning-dbaas01-beta.iktsenteret.c.bitbit.net uno_data_beta -u uno_data -p  red_korrigering x_geonorge_kommuner z_undervisningssteder z_fagskole_tilbud z_folkehogskole_tilbud z_so_kravkoder z_organisasjoner z_ssb_nus z_ssb_styrk98 z_so_uh z_uh_tilbud z_vgs_tilbud z_annen_utdanning z_bedrifter z_so_fs z_privat_vgs_tilbud | gzip > /srv/tmp/beta.utd-dk.utdanning.yyymmdd.sql.gz`
- `cd /srv/beta.min.utdanning.no`
- minside: `mysqldump --column-statistics=0 -h utdanning-dbaas01-beta.iktsenteret.c.bitbit.net minside_beta -u uno -p | gzip > /srv/tmp/beta.min.utdanning.no.yyyymmdd.sql.gz`
- `cd /srv/beta.data.utdanning.no`
- data: `mysqldump --column-statistics=0 -h utdanning-dbaas01-beta.iktsenteret.c.bitbit.net beta_data_utdanning_no -u beta_data_utdanning_no -p | gzip > /srv/tmp/beta.min.utdanning.no.yyyymmdd.sql.gz`

### Last ned databasene

Legg dem i rot-katalogen til prosjektet. Det er der importskriptene forventer å finne dem.

For alle 3 database-dumpene:

- `scp uno-beta:/srv/tmp/filnavn.sql.gz .`

### Forberede import av databaser

#### Alternativ 1: Via nettleseren

Dette kan gjøres ved å gå til dev-nettstedene dine:

- http://dev.utdanning.no
- http://dev.min.utdanning.no
- http://dev.data.utdanning.no (ser ikke ut som om det er nødvendig)

Databasekonfigurasjon som installasjonen ber om ligger i filene `dbpwfile.txt`, `minside_dbpwfile.txt` og `data_dbpwfile.txt`.

#### Alternativ 2: Manuelle endringer i konfigurasjonsfiler


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

Endringer skal gjøres i `source/data.utdanning.no/web/sites/default/settings.php` og du behøver verdier fra fila `data_dbpwfile.txt`.

Legg inn en tilfeldig tekst i `$settings['hash_salt']`, f.eks.

> $settings['hash_salt'] = 'skldzfhnlcahegmhrmdlckggmaudhrcghugl84y8gae4cgmloz8glmo8';

Legg inn database-adapteren fra `source/utdanning.no/web/sites/default/settings.php` som har navn `datakollektivet`

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

Legg inn dette nederst i settings-fila:

<pre>
$databases['default']['default'] = [
  'database' => 'Database fra data_dbpwfile.txt',
  'username' => 'User fra data_dbpwfile.txt',
  'password' => 'Password fra data_dbpwfile.txt',
  'host' => 'Host fra data_dbpwfile.txt',
  'port' => '3306',
  'driver' => 'mysql',
  'prefix' => '',
];

$config['config_split.config_split.dev']['status'] = 1;
$config['config_split.config_split.stathis_local']['status'] = 0;
$config['config_split.config_split.alfa']['status'] = 0;
$config['config_split.config_split.beta']['status'] = 0;
$config['config_split.config_split.prod']['status'] = 0;
</pre>


### Importere databaser

#### Datakollektivet (dev.utdanning.no)

> ./robo.phar dbcustom datakollektivet uno_data beta.utd-dk.utdanning.no.yyyymmdd.sql.gz

Denne kommandoen ber deg om et passord. Bruk passordet du la inn i oppsettet til datakollektivet-databasen til dev.utdanning.no.

#### dev.utdanning.no

> ./robo.phar db:restore-utd beta.utdanning.no.yyyymmdd.sql.gz

#### min.utdanning.no

> ./robo.phar db:restore-ms beta.min.utdanning.no.YYYYMMDD.sql.gz

#### data.utdanning.no

> ./robo.phar db:restore-data beta.data.utdanning.no.yyyymmdd.sql.gz


### Avslutning

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


Hvis `drush cim` foreslår omfattende endringer så tyder det på at noe har blitt feil. Endringene du skal få er i hovedsak forskjeller mellom beta- og dev-miljø.

