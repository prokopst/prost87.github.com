---
title: Jekyll a Github Pages
layout: post
perex: Github Pages je svým způsobem hosting statického webu v repozitáři. Kouzlo tohoto hostingu spočívá v podpoře Jekyll, generátoru statických stránek využitelným pro jednoduchý blog. Výzva přijata a zmigrováno!
---
Mým cílem bylo zmigrovat na tyto technologie blog. Hlavní lákalo pro mne bylo použít [Github flavored markdown](http://github.github.com/github-flavored-markdown/) a vůbec celý ten jednoduchý koncept, který bude popsán níže.

Jekyll
------

O [Jekyll](https://github.com/mojombo/jekyll) je toho na webu dost, takové hrubé vysvětlení je, že člověk si vytvoří nějaké šablony (nezdravá mixáž HTML a liquid šablon těžce inspirovaných Djangem), v adresáři ```_posts``` vytvoří pár souborů v markdown (nebo textile a jiných zvrhlostí) a po běhu z toho všeho vygeneruje **statický web**. Pro jistotu opakuji - **statický web**. Divné že, v dnešní době něco takového generovat... ale vlastně proč ne? Nakolik je dynamický takový uhratý IT blogísek?


Github Pages
------------

Github Pages je **branch 'gh-pages' v libovolném repozitáři** projektu. Pokud tam dostanete nějaký obsah (ať už stránka v Jekyll nebo statická, viz niže), dostanete se tam skrze URL ```username.github.com/projectname```, přesně jak [popisuje dokumentace](https://help.github.com/articles/user-organization-and-project-pages).

Ale Github Pages není jen pro dokumentaci a kecy projektu, člověk si tam může zařídit i **vlastní blog**. Třeba jako je tento. Narozdíl od stránek projektu je nutné použít místo branch 'gh-pages' **obsah v branch 'master'**. Svým způsobem logické, svým způsobem si někdo důsledně vytřel zadek konzistencí. Variantu vlastních stránek popisuje [ta samá stránka](https://help.github.com/articles/user-organization-and-project-pages) dokumentace. V tomto příspěvku se omezím pouze na osobní stránky - neuvedu-li explicitně jinak.

Použití pro blog
----------------

Po prolézání docela **strohé dokumentace** (v porovnání s dokumentací [Django](https://www.djangoproject.com/) je asi všechno strohé) jsem se rozhodl jít cestou prototypu - pokusů a omylů a slepování informací z různě zašitých míst.  Tohle hraní odhalilo pár vad celé té *krásy*. V první řadě pokud je provedeno něco špatně, jekyll mlčí. Když si nastavíte špatný layout, stane se, um... docela nic. Neexistující layout se totiž tiše nepoužije, jediným indikátorem je poněkud rozbitá stránka. Navíc někdo zapomněl dodat debug mode (nebo jej důsledně zamlčel). A pokud už se člověku podaří vytvořit všimnutelnou chybu při generování statického obsahu, Exception s **user unfriendly** hláškou většinou požaduje kontext místa vyhození. A honba za chybou stvořenou mezi klávesnicí a židlí může začít.

Předchozí se nakonec dá překousnout, ale bez tuctu sprostých slov se neobejde **Kocourkov na straně Githubu**. Github používá na celém webu Github flavored markdown, konečně použitelnou variantu [markdown](http://daringfireball.net/projects/markdown/). Github Pages jej... [nepoužívá](http://stackoverflow.com/questions/10759577/underscore-issues-jekyll-redcarpet-github-flavored-markdown)! Hmm, i Github evidentně poskytuje výkaly. Pluginy na Github Pages použít také nelze, takže ani touto cestou se není možné vydat. Ale toto není jediné omezení Github Pages, další jsou nepodpora kategorií a tagů. Jak by řekl opravář Lakatoše teď i o pár odstavců níže - [to je past vedle pasti](http://milujipraci.cz/sfx/past-vedle-pasti-pico.mp3).

Proč ale vytvořit Jekyll web lokálně a vybudovat jej vzdáleně? Proč všechno nemít lokálně a teprve push by dodal výslednou vygenerovanou stránku? Ano, taková možnost tu je, stačí mít v root ```master``` branch (nebo ```gh-pages```, viz výše) soubor ```.nojekyll``` a zbytek se tváří jako statický web.

**Jednoduché, že? Ne, není.**

První problém je, že dobrodruh, který se rozhodl pro tuto džungli, musí mít Jekyll web v branch X (dejme tomu ```source```) a statickou stránku v branch ```master```. Případně by šlo stránku generovat v rámci jedné branch přímo do root, **kdyby ovšem jekyll nemazal kompletně všechny soubory** v root před vygenerováním statické stránky. A to včetně ```.git```, které z lokálního repozitáře dělá lokální repozitář. Ha! Tohle naštěstí [bude jednoho dne částečně opraveno](https://github.com/mojombo/jekyll/issues/534). Zpět k řešení s dvěmi větvemi. Nejdříve vygenerovat obsah, pak přepnout branch a obsah dodat do root. Celé je to takové manuální a poblité. Nepoužitelné.

git-publish-pages
-----------------

Kvůli předchozím problémům jsem provedl **zářez do open-source pažby** a celé to [zautomatizoval nástrojem git-publish-pages](https://github.com/prost87/git-publish-pages). Až na pár věcí - tou zásadní je tvorba ```gh-pages``` branch pro použití mimo osobní web. Jelikož příkazy git vrací při úspěchu 0 a [při neúspěchu nejčastěji 1](http://stackoverflow.com/a/4918002), ošetřit neexistující branch nemusí být nutně korektním ošetření, protože chyba mohla nastat jiná. A parsovat chybové hlášky je **směšné. Debilní. Absurdní.** Takže git kromě sémanticky špatných příkazů (```git checkout``` pro přepnutí větve nebo pro zahození změn v adresáři nebo souboru) ještě vrací 1 při všelijakých různých chybách. ```<ironie>```**Ať žije git!**```</ironie>```

github-pages-jekyll-example
---------------------------

Dalším open-source projektem, který nutně musel vzniknout, byl vůbec celý prototyp statické stránky, na kterém jsem celou šaškárnu ověřoval. Je to takové primitivní a jednoduché:

```
.
├── _config.yml
├── README.md
└── _source
    ├── css
    │   ├── pygments.css
    │   └── style.css
    ├── index.html
    ├── _layouts
    │   ├── base.html
    │   └── post.html
    └── _posts
        ├── 2012-12-21-hello-world-nodejs.md
        ├── 2012-12-22-hello-world-c.md
        ├── 2012-12-23-hello-world-java.md
        ├── 2012-12-24-hello-world-lua.md
        └── 2012-12-25-hello-world-python.md
```

Tento prototyp k nalezení [tady](http://prost87.github.com/github-pages-jekyll-example) obsahuje:

* Jekyll web v master branch, statický web v gh-pages branch
* github flavored markdown
* stránkování (pagnation)
* css (jedno prázdné a druhé pro obarvení kódu)
* pár příspěvků
* mini návod, jak to vše zprovoznit lokálně

Porod nakonec snad k něčemu byl. Koncept Jekyllu a Github Pages se mi líbí, pokud **najdete chybu v článku, můžete provést fork, opravit a poté pull request**. Fork & go. Dokázal bych si přestavit toto řešení i pro nějaké fan pages, ale asi jsem příliš unavený.

Alternativy
-----------

**Alternativec** si může zvolit generátor statických stránek v Pythonu pojmenovaný [Hyde](http://ringce.com/hyde). A místo Github Pages? BitBucket podporuje něco podobného, více v [jejich dokumentaci](https://confluence.atlassian.com/display/BITBUCKET/Publishing+a+Website+on+Bitbucket). Případně Amazon [podporuje statické strány](http://docs.amazonwebservices.com/AmazonS3/latest/dev/WebsiteHosting.html) pro jejich červí díru zvanou s3.