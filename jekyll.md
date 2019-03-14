---
title: Jekyll
---

## {{ page.title }}

Jekyll bygger dette nettstedet fra markdown-filer. Etter en 'git push' så bygger github automatisk og publiserer på uhbe.github.io, men det er mye mer 
praktisk å byggge nettstedet lokalt når man gjør mange endringer. For å gjøre det så må Jekyll installeres 

### Klargjøre for å installere Jekyll på Ubuntu

Følgende må gjøres:

<code>sudo apt-get install ruby-full build-essential zlib1g-dev</code>
<pre><code>
echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
</code></pre>

Dette er hentet fra https://jekyllrb.com/docs/installation/ubuntu/ . NB! Ikke gjør mer derfra!


### Installere Jekyll med github-oppsett

Følg deler av instruksjonene på  https://help.github.com/en/articles/setting-up-your-github-pages-site-locally-with-jekyll
2.1-4 spesifiserer Gemfile. Denne skal være inkludert i repositoriet.

Mer spesifikt:

<pre><code>bundle install</code></pre>

Deretter kan nettstedet kjøres lokalt med

<pre><code>bundle exec jekyll serve</code></pre>

Og nettsteder er nå tilgjengelig på http://localhost:4000

Oppdater gem "Github pages" ofte: <pre><code>bundle update github-pages</code></pre>. Eventuelt <pre><code>bundle update</code></pre> for å oppdatere alle gems.
