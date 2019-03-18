---
title: Jekyll
---

## {{ page.title }}

Jekyll bygger dette nettstedet fra markdown-filer. Etter en 'git push' så bygger github automatisk og publiserer på uhbe.github.io, men det er mye mer 
praktisk å byggge nettstedet lokalt når man gjør mange endringer. For å gjøre det så må Jekyll installeres.

### Klargjøre for å installere Jekyll på Ubuntu

Følgende må gjøres:

Installer nødvendig programvare for Ubuntu:
<pre><code>sudo apt-get install ruby-full build-essential zlib1g-dev</code></pre>

Legg opp til at gems installeres på din brukerkonto:
<pre><code>echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
</code></pre>

Installer bundler:
<pre><code>gem install bundler</code></pre>

Dette er delvis hentet fra [Jekyll sine installasjonsinstruksjoner for Ubuntu](https://jekyllrb.com/docs/installation/ubuntu/) . NB! Ikke gjør mer derfra!

Edit 1.

### Installere Jekyll med github-oppsett

Installer Github-Jekyll (som spesifisert i Gemfile):

<pre><code>bundle install</code></pre>

Deretter kan nettstedet kjøres lokalt med:

<pre><code>bundle exec jekyll serve</code></pre>

Og nettstedet er nå tilgjengelig på [http://localhost:4000](http://localhost:4000)

Oppdater gem "Github pages" ofte:
<pre><code>bundle update github-pages</code></pre>

Eventuelt <code>bundle update</code> for å oppdatere alle gems.

Dette er delvis hentet fra [Githubs instruksjoner for å sette opp Github Pages lokalt med Jekyll](https://help.github.com/en/articles/setting-up-your-github-pages-site-locally-with-jekyll)

Se også [Jekyll sin side om Github Pages](https://jekyllrb.com/docs/github-pages/)
