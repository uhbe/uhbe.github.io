---
title : Twig-bruk på utdanning.no
---

## {{ page.title }}

### Filtere

Vi har en egen modul, twig_extensions, som definerer enkelte nyttige filtere. Her er et sammendrag av hvilke muligheter dette gir oss.


#### label_display

Bestemmer hvor/hvis feltets label skal vises. Tar samme verdier som brukes i GUI, altså "above", "inline", "hidden" og "visually_hidden"

Eksempel:

    {% raw %}{{ content.field_kort_tittel|label_display('hidden') }}{% endraw %}


#### theme_config

Bestemmer variabler som skal bli tilgjengelig i templates.

I node-template kan man ha dette:

    {% raw %}{{ content.field_name|theme_config({"separator": " | "}){% endraw %}

Og så lese det av i field-template med 

    {% raw %}{% set config_separator = get_theme_config(element, "separator", ", ") %}{% endraw %}

Deretter er verdien tilgjengelig med koden <code>{% raw %}{{ config_separator }}{% endraw %}</code>

Tredje parameter til get_theme_config er ønsket default-verdi.

#### theme_suggestion

Dette spesifiserer hvilken template vi ønsker at skal brukes til et felt. Med dette kan vi lage generelle templater og bruke de der vi vil uten
å behøve å opprette filer med filnavn som matcher feltene.

I node-template:

    {% raw %}{{ content.field_intvj_type|theme_suggestion('field__minimal')  }}{% endraw %}

Da vil field--minimal.html.twig bli brukt som template for feltet (hvis den finnes).


#### field_image_style

Brukes til å spesifisere ønsket bildestil. Eksempel på bruk i en node-template:

    {% raw %}{{ content.field_artik_illustrasjon_mid | field_image_style('bredformat_medium') | label_display('hidden') }}{% endraw %}

NB! Format for bildefeltet på innholdstypen sin "Administrer visning" må være "Thumbnail", ikke "Vist entitet".


### Kombinasjon av filtere

Alle filtrene kan kombineres så man kan bruke noe sånt som dette:

    {% raw %}{{ content.field_kort_tittel|label_display('hidden')|theme_config({"html-item-element": "p"}) }}{% endraw %}

### Funksjoner

#### field_label

Henter ut labelen til et felt. Eksempel:

    {% raw %}{{ field_label(content, "field_syff_kopi_sluttkompetanse") }}{% endraw %}


#### get_theme_config

Hente ut konfigurasjon som er satt med theme_config-filteret i en template: Eksempel:

    {% raw %}{% set config_separator = get_theme_config(element, "separator", ", ") %}
    {{ config_separator }}{% endraw %}

#### not_empty_field

Funksjon som returnerer true hvis feltet har en eller flere verdier og false hvis det er tomt. Eksempel:

    {% raw %}{% if not_empty_field(content.field_syff_kopi_programomrade) %}
        <section>
            <h4>{{ field_label(content, "field_syff_kopi_programomrade") }}:</h4>
            <p>{{ content.field_syff_kopi_programomrade }}</p>
        </section>
    {% endif %}{% endraw %}
