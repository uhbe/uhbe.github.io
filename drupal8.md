---
title: Oppsett av Drupal 8
---

## {{ page.title }}

## Sjekke ut prosjektet

- `git clone \-\-recurse-submodules git@github.com:Utdanningsdirektoratet/dev.utdanning.no.git`
- gå inn i sourcemappa og så inn i utdanning.no og min.utdanning.no og ta en `git checkout development`

`git checkout development` bør alltid gjøres før man endrer kode i submodulene


## Docker-oppsett

- installer Docker. Se https://docs.docker.com/install/linux/docker-ce/ubuntu/ for instruksjoner for Ubuntu
- Installer docker-compose: `sudo apt install docker-compose`
- Meld deg inn i docker-gruppa. Etter dette har det vist seg nødvendig å restarte pc-en.
- For Mac: Installer Docker Desktop
- For Mac: Kildekoden må være sjekket ut i en mappe som ligger inne i 'File sharing'-innstillingene til Docker. Hvis ikke så får ikke Mac mountet filsystemet inne i Docker.
- `./robo.phar install`
- NB! Hvis du har Apache eller noe annet kjørende på port 80 så vil robo-installasjonen feile. Stopp Apache eller endre varnish.ports i docker-compose.yml til f.eks. 81:80.


## Apache-oppsett

Det avhenger også av eksisterende oppsett og portkonfigurasjon. Docker vil i utgangspunktet ta port 80 så det kræsjer fort med eksisterende oppsett.

- Legg inn dev8.utdanning.no i /etc/hosts

## Alternativt Apache-oppsett

For å unngå kollisjon med eventuelle andre ting på port 80 så kan man endre varnish.ports i docker-compose.yml til 81:80 og bruke det følgende i en vhost-fil til Apache:
<pre>
<VirtualHost *:80>
    ServerName dev8.utdanning.no
    ProxyPass "/"  "http://dev8.utdanning.no:81/"
    ProxyPassReverse "/"  "http://dev8.utdanning.no:81/"
</VirtualHost>
</pre>

For at dette skal fungere så må proxy og proxy_http-modulene aktiveres:

> sudo a2enmod proxy proxy_http

Aktiver deretter vhost-en. (`sudo a2ensite ....`)

Etter dette så kan dev8.utdanning.no brukes uten at man behøver å tenkte på at den egentlig kjøres på port 81. Men der Drupal bruker absolutte lenker så kan :81 dukke opp.



## Drupal 8

Åpne dev8.utdanning.no i en nettleser og følg instruksjonene.

- Bruk db-oppsett fra dbpwfile.txt


## Legge inn database og filer

Kort sagt: Dump database fra beta og last ned. Installer med `./robo.phar dbrestore`. Kopier fil-katalogen.
Hvis du har installert Drupal 8 for utdanning.no tidligere, men ikke lasted ned filene så bør du også oppdatere databasen.

- Lokalt (ta backup av evt. gammel db): `./robo.phar drush "sql-dump > /srv/tmp/dev8.uno.dump.sql"`
- På beta2: `drush sql-dump | gzip > /srv/tmp/dump.uno8_beta.sql.gz`
- Lokalt: Last ned fila fra beta og legg den i prosjektkatalogen, altså på samme sted som robo.phar.
- Lokalt: `./robo.phar dbrestore dump.uno8_beta.sql.gz`
- På beta2: `tar -zcvf /srv/tmp/beta.d8.files.tar.gz files` (hvis du står i web/sites/default-katalogen)
- Lokalt: last ned fil-dumpen (f.eks. med scp) og legg den et sted du får tak i den
- Lokalt: `cd source/utdanning.no/web/sites/default`
- Lokalt: `mv files filesold` (så du har de gamle filene i tilfelle noe skjærer seg)
- Lokalt: `tar -xzvf /srv/tmp/beta.d8.files.tar.gz` (juster sti til fildump hvis nødvendig)

## Alternativ uten x_-tabeller

Datakollektivet inneholder en rekke tabeller prefikset med 'x_' og disse brukes ikke i Drupal. For å lage en dump av databasen uten disse tabellene kan følgende kommando brukes

- `drush sqlq --database=datakollektivet 'SELECT table_name FROM information_schema.tables WHERE table_schema = "uno_data_beta" AND table_name NOT like "x_%"' | tr '\n' ','  | xargs -I % drush sql-dump --database=datakollektivet --tables-list='%' | gzip > /srv/tmp/beta.datakollektivet.dump.sql.gz`



## Theme-utvikling

- `cd web/themes/custom/utdanning`
- `npm install`
- `npx gulp`

Du kan nå se nettstedet på localhost:3000 (autoppdaterer ved css- og js-endringer) eller dev8.utdanning.no. Det er lurt å være innlogget for da er du mindre utsatt for Drupal-cache.


## Twig

Følg instruksjonene på https://www.drupal.org/node/2598914 inntil "Use Drupal Console development mode" for å deaktivere mest mulig cache. Hvis du ikke gjør dette så blir du snart lei av å flushe cachen.

I Drupal sin Twig-dokumentasjon anbefales det å opprette en services.yml (som kopi av sites/default/default.services.yml) og endre twig.config i denne. Dette fungerer bare delvis.


## Flushe cache

> ./robo.phar drush cr


## I tilfelle feil

Hvis Drupal kræsjer hardt og brutalt på grunn av php-feil så kan det være lurt å fjerne composer.lock og så kjøre en ny `composer install`.

## Datakollektivet

Datakollektivet settes opp i en egen database i web/sites/default/settings.php (NB! settings.php er ikke i git. Se tilsvarende fil på beta.). Det kan være lurt å hente ny versjon av Drupal-databasen fra beta og importere den før du fortsetter på oppsett av Datakollektivet.

Dump datakollektivet-databasen på beta:

> drush sql-dump \-\-database=datakollektivet \| gzip > /srv/tmp/beta.datakollektivet.dump.sql.gz

NB! Sjekk at det er nok ledig diskplass. Slett eventuelt noen gamle db-dumper først.

Deretter laster du ned fila til prosjektkatalogen på din maskin, (`/srv/dev8.utdanning.no/` eller lignende).

Gå til prosjektkatalogen og så kan du opprette databasen lokalt og importere dumpen fra beta:

> ./robo.phar dbcustom uno_data_beta uno_data beta.datakollektivet.dump.sql.gz

NB! Vi bruker `uno_data_beta` som lokalt db-navn inntil videre pga noen litt for hardkodete sql-views. Denne kommandoen vil be deg om å angi et passord for db-brukeren. Noter dette og legg det inn i settings.php.

Eksempel på kodeblokk for å definere datakollektivet-databasen i settings.php:

    $databases['datakollektivet']['default'] = array (
      'database' => 'uno_data_beta',
      'username' => 'uno_data',
      'password' => '**********',
      'prefix' => '',
      'host' =>  'db,
      'port' => '3306',
      'namespace' => 'Drupal\\Core\\Database\\Driver\\mysql',
      'driver' => 'mysql',
      'collation' => 'utf8mb4_danish_ci',
    );

Det kan også hende at databasebrukeren ikke har blitt opprettet ordentlig så du må opprette den selv. Logg inn i databasen i databasecontaineren

> ./robo.phar connect db
> mysql

og kjør følgende kommandoer:

> grant all on uno_data_beta.* to 'uno_data'@'localhost' identified by 'PASSORDHER';
> grant all on uno_data_beta.* to 'uno_data'@'utdanningno_app%' identified by 'PASSORDHER';


Hvis alt har gått som det skal så har du nå en oversikt over de eksterne entitetstypene på `/admin/structure/datakollektivet-entity-types`

