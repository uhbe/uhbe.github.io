---
title: Manual for frontend-utvikling
---


## Installasjon av miljø

Vi bruker et Docker-basert miljø for utvikling av utdanning.no. Se [https://uhbe.github.io/drupal8.html](https://uhbe.github.io/drupal8.html) for oppsett.

## Theme

Vi bruker et theme som er basert på Bootstrap og har støtte for SASS og React-komponenter i tillegg til standard CSS og JavaScript.
Det er plassert i `web/themes/custom/utdanning`. Vi bygger theme lokalt og sjekker inn resultatet (`npx gulp prod` før `git add/commit`). Kjør `npm install` før du begynner å utvikle i themet.

I hovedsak så plasserer vi JavaScript- og CSS-kode vi jobber med i src-mappa i themet og så bruker vi Gulp til å bygge og kopiere filer til dist-mappa. Koden gjøres tilgjengelig for Drupal ved å inkludere den i et såkalt "library" i `web/themes/custom/utdanning/utdanning.libraries.yml` og så inkludere den globalt i themet i `web/themes/custom/utdanning/utdanning.info.yml` eller på en spesifikk side med `attach_library`. Se `web/themes/custom/utdanning/templates/yrke/node--yrke--full.html.twig` for et eksempel. Se også Drupal-dokumentasjon for [Adding stylesheets (CSS) and JavaScript (JS) to a Drupal theme](https://www.drupal.org/node/2216195).

### Gulp

Vi bruker Gulp til å håndtere byggeprosessen til themet vårt. Se gjerne `web/themes/custom/utdanning/gulpfile.js`, det er ganske greit å lese seg fram til hva som skjer i de forskjellige kommandoene som er implementert der.

Kort beskrivelse av kommandoer

- `npx gulp` - Kjører hele dev-miljøet med live-reload. Live-reload kan være litt upålitelig pga Drupal og cache knyttet til innlogging i kombinasjon med at live-reload er basert på en proxy.
- `npx gulp prod` - Bygger theme som kan deployes til prod.
- `npx gulp js` - Kopierer javascript-filer til dist-mappe
- `npx gulp sass` - Kompilerer sass til css og flytter resultatet til dist-mappa.
- `npx gulp sass-prod` - Kompilerer sass til minimert css og flytter resultatet til dist-mappa.
- `npx gulp webpack` - Bygger React-komponenter.

Det kan være fordelaktig å bruke de "mindre" kommandoene når du jobber med bare css eller js fordi de er kjappere og man unngår diff i dist-filer man ikke har til hensikt å endre på pga avvikende versjoner ol. Men live-reload er bare implementert for hovedkommandoen.

### Twig

Template-filene våre er skrevet i [Twig](https://twig.symfony.com/) fordi det er det som brukes i Drupal. Template-filene i themet vårt er plassert i mappa `web/themes/custom/utdanning/templates/` og det kan være template-filer i enkelte moduler også. Som med andre template-løsninger så er det best å ha minst mulig logikk i template-fila. Enkle for-løkker og if/else er greit, men dataprosessering og omfattende logikk bør skilles ut i PHP-kode. Enten i themet (fila `utdanning.theme` inneholder en del sånne ting) eller i en egnet modul for problemet som skal løses.

I forhold til "Drupal best practices" så foretrekker vi å bruke Twig på en litt annen måte. "Drupal-standard" er å lage template-filer for individuelle felt slik at feltet selv vet hvordan det skal rendres i gitte tilfeller. Dette fører til relativt enkle template-filer, men det kan fort bli uoversiktlig mange av de. Vi foretrekker at en side-template skal si mest mulig om hva som skjer på siden og felt-template skal bare gi ut felt-verdien med minimal HTML-kode. Det blir da enklere å få et helhetlig bilde av hva som skjer på en side og vi får færre template-filer å forholde oss til.

Vi har en egen modul med Twig-utvidelser plassert i `web/modules/custom/twig_extensions/`. Denne modulen er delvis dokumentert på [https://uhbe.github.io/twig.html](https://uhbe.github.io/twig.html). Se spesielt hvordan funksjonene `theme_suggestion` og `theme_config`/`get_theme_config` fungerer. Vi kan altså ha generelle felt-templates som kan styres av parametere etter behov.

Vi bruker også contrib-modulen [Twig Tweak](https://www.drupal.org/project/twig_tweak) som inneholder en mengde nyttige funksjoner for å hente ut Drupal-data. 

En annen ting vi prøver å oppnå er å styre innstillinger via Twig-templatene i stedet for i GUI. Dette har noen utfordringer og mange ganger må ting uansett gjøres i GUI. Årsaken til dette er at Drupal-nettsteder typisk har en site-builder som er en teknisk rolle som vil jobbe mest mulig i Drupal-GUI og er plassert mellom utviklere og redaksjon. Vi har i stedet bare utviklere og redaksjon og vi gjør site-building på utviklervis, altså mer i kode enn i GUI.

Les mer om Twig i Drupal på [https://www.drupal.org/docs/theming-drupal/twig-in-drupal](https://www.drupal.org/docs/theming-drupal/twig-in-drupal).

### React

Som nevnt så har vi støtte for React-komponenter i themet. Dette kan brukes til egne komponenter eller til komponenter som hentes fra NPM via en liten shim som må skrives hos oss. React-komponenter brukes for eksempel på yrkesbeskrivelsene våre. Template `web/themes/custom/utdanning/templates/yrke/node--yrke--full.html.twig` inneholder blant annet
{% raw %}
```
<div id="yrke-tabs" data-react-component="true" data-props={{ bedrifter|json_encode }}></div>
```
{% endraw %}

Her sier `data-react-component="true"` at dette er en react-komponent. Deretter så vil komponenten `web/themes/custom/utdanning/src/react/components/yrke.component.js` koble seg på via id `yrke-tabs`. Hvordan dette oppsettet fungerer er dokumentert i [https://kompetansenorge.atlassian.net/browse/UTDTEK-655](https://kompetansenorge.atlassian.net/browse/UTDTEK-655).

