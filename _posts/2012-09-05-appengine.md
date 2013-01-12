---
title: AppEgine na localhost a subdomény
layout: post
perex: V projektu, na kterém momentálně pracuji, jsem narazil na problém - spustit devel server AppEngine se subdoménami, kde se aplikace rozhoduje na základě subdomény. Zprovoznění není zrovna přímočaré, je nutné nainstalovat a nastavit virtualenv s AppEngine, pohrát si s dnsmasq a povolit pythonu poslouchat na portu 80. O tom všem dále.
---
Virtualenv
----------

Virtualenv je nástroj k vytvoření izolovaného Python prostředí, které uživatel může bezpečně demolovat a znečistit svým odpadem. Ještě nedávno AppEngine s virtualenv nespolupracoval tak, jak by si vývojář přál - viz [bug](http://code.google.com/p/googleappengine/issues/detail?id=4339). Dnes již funguje, tak hurá pro něj. Pro vytvoření prostředí stačí pustit příkaz:

```bash
virtualenv --distribute --no-site-packages dev_env
```

Tento příkaz vytvoří adresář 'dev_env' s čerstvým Python prostředím, budou zde hnít vlastní binárky a vlastní balíky. Volba --distribute místo 'easy_install' použije 'distribute', o současnému stavu věcí [zde](http://guide.python-distribute.org/introduction.html#current-state-of-packaging). Volba --no-site-packages provede to, že nové prostředí nebude s tím hlavním v operačním systému sdílet balíky.

Já si tato prostředí tvořím v mém 'home'. Jmenovitě tam budou minimálně 'python', 'easy_install' a 'pip'. Mělo by se zjevit něco velmi podobného:

```bash
prost@prost-VirtualBox:~$ virtualenv --distribute --no-site-packages dev_env
New python executable in dev_env/bin/python
Installing distribute...............done.
Installing pip...............done.
```

Prostředí se aktivuje následovně:

```
source dev_env/bin/activate
```

Na mém prostředí se prakticky stane toto:

```
prost@prost-VirtualBox:~$ source dev_env/bin/activate
(dev_env)prost@prost-VirtualBox:~$ 
```

Příkaz source je standardní věc shellu, spustí v rámci současného shell tento soubor, který si na první pohled pohraje s prompt (na začátek v závorce vrazí název prostředí, užitečné), lehce pohraje s proměnnou PATH (první položkou bude absolutní cesta k 'dev_env/bin') a upraví všechny proměnné prostředí, které se váží k Pythonu. Není v tom žádná magie, stačí se podívat do souboru 'activate' a je jasno. Je tam i 'activate.csh', varianta pro atypický (diplomaticky řečeno nepoužívaný) shell 'csh' a kdo ví co ještě. Jak je libo.

Google App Engine
-----------------

Prvním krokem je bezesporu stáhnout 'Google Application Engine SDK for Python' [odsud](http://code.google.com/appengine/downloads.html#Google_App_Engine_SDK_for_Python). Poté rozbalit třebas do adresáře '/opt', jehož účelem je uchovávat takovéto mrchy. Mně v systému srmdí v '/opt/google_appengine'.

Nyní je třeba vecpat adresář '/opt/google_appengine' do PATH proměnné prostředí, které vytvořil virtualenv o sekci výše. Stačí na konci souboru 'dev_env/bin/activate' trošku upravit PATH, a to přidat do ni cestu k appengine. Pro líné vše by mělo stačit:

```bash
# copy & paste lazy dudes
echo -e 'PATH=/opt/google_appengine:$PATH\nexport PATH' >> dev_env/bin/activate
source dev_env/bin/activate
```

Google Application Engine pro lokální vývoj má hromadu balíků jako webapp2, jinja2, webob a spousty dalších.

Nastavení pro použití subdomén lokálně
--------------------------------------

Pokusy s '/etc/hosts' mne dovedli k nepodpoře divočin jako *.localhost (wildcards) a dle všech mých poznatků prakticky nic nepodporuje divočejší věc s portem, to jest *.localhost:8080. Můj projekt se dynamicky měl rozhodovat na úrovni subdomén a protože nastavení několika testovacích subdomén natrvdo - typu 'test1.localhost' - nebylo úplně košér, zvolil jsem trochu obtížnější cestu.

DNS server dnsmasq a jeho nastavení
-----------------------------------

Nejdříve je třeba nainstalovat dnsmasq, takový lehčí dns server. Tahle věc by neměla nikoho trápit, je to jednoduše vypnutelný démon. Na Debianu stačí do konzole vystřelit:

```bash
sudo apt-get install dnsmasq
```

Tento server používá pro svou konfiguraci '/etc/dnsmasq.conf' a obsah '/etc/dnsmasq.d/'. Pro konfiguraci jsem přidal soubor 'localhost.conf', ale prakticky je to úplně jedno, hlavně ať ta následující kofigurace někde hnije:

```bash
# restrict dnsmasq to requests from localhost
listen-address=127.0.0.1
# 127.0.0.1 responds to requests coming from *.localhost
address=/localhost/127.0.0.1
```

Pro líné zadky by mělo stačit spustit toto:

```
sudo echo -e \
'listen-address=127.0.0.1\naddress=/localhost/127.0.0.1' > /etc/dnsmasq.d/localhost.conf
```

Samozřejmě, pro jakoukoliv změnu by to chtělo restartovat (nebo nastartovat) server:

```
sudo /etc/init.d/dnsmasq restart
```

Nyní, aby to všechno fungovalo, musí se donutit dhcp klient k tomu, aby to vůbec umožnil. V souboru '/etc/dhcp/dhclient.conf' je řádek, který je třeba odkomentovat (tj. umazat první znak '#' na tomto řádku):

```
#prepend domain-name-servers 127.0.0.1;
```

No a nebo jen spustit:
```
sudo echo 'prepend domain-name-servers 127.0.0.1;' >> /etc/dhcp/dhclient.conf
```

Posledním krokem je trochu aktualizovat informace dhclient:

```bash
sudo dhclient
```

Povolit aplikaci běžet na portu 80
----------------------------------

Zde už je trošku problém. Na Unix-like OS typicky jen privilegovaní uživatelé mohou nechat aplikaci naslouchat na portu 80 (přesněji na portech nižších než 1024). Při pokusu o puštění projektu tak vyskočí traceback s jasnou informací, 'Permission denied'.

```
(dev_env)prost@prost-VirtualBox:~/projects/secret_project$ dev_appserver.py --port 80 .
...
socket.error: [Errno 13] Permission denied
```

Je tak nutné pro nějakou binárku povolit tento port. Tohle v Debianu jde poměrně jednoduše, ale příchází problém druhý - pouze binárky. Skriptům to povolovat nelze, protože ve skutečnosti je pouštěna binárka, tj. typicky '/usr/bin/python'. To je potenciální bezpečnostní díra. Proč? Všechno, co použije python, může použít nejen port 80, ale všechny privilegované. To mi bylo celkem u zad, můj operační systém pro vývoj si kýchá ve virtuálním stroji, nebylo třeba se strachovat. Navíc pokud bylo vytvořeno Python prostředí pomocí virtualenv, povolení portu proběhne pro relativně izolované prostředí. Každopádně zakázat port 80 pro použití zvenčí na fyzickém stroji není zrovna špatný nápad.

Otevírat porty pro aplikace umí utilita set capabilities naleznutelnou pod 'libcap2-bin' v Debianu, copy & paste instalace tu:

```
sudo apt-get install libcap2-bin
```

Povolení se provádí způsobem níže, prakticky se nastaví v ACL cosi, co umožní pro tento soubor používat porty nižší než 1024.

```bash
sudo setcap 'cap_net_bind_service=+ep' /path/to/binary
```

Následující příkaz pusťte jen v prostředí pythonu, který chcete používat pro běh aplikace na portu 80:

```bash
sudo setcap 'cap_net_bind_service=+ep' $(readlink -f $(which python))
```

Nyní je možné pustit vysněnou aplikaci se subdoménami na portu 80:

```bash
dev_appserver.py --port 80 .
```
