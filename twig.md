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

Dette spesifiserer hvilken template vi ønsker at skal brukes til et felt

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
