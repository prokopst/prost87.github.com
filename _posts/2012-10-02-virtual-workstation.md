---
title: Virtual workstation
layout: post
perex: Virtual workstation je prakticky pracovní operační systém ve virtuálním stroji na operačním systému, který se k vývoji moc nehodí. Původně jsem s tím začal kvůli problémům s hardware - zprovoznit audio vypadalo jako několikadenní laborování, NVIDIA Optimus nespolupracoval a větráček na GPU byl monumentálně slyšet. Po čase používání jsem poznal určité výhody, se kterými se podělím.
---
Oddělené prostředí od hlavního
------------------------------

První a z mého pohledu největší výhodou je oddělení zájmů, přesněji oddělení pracovního prostředí od hlavního operačního systému. Zapnete virtuální mašinu a programujete. Prohlížeč může zůstat v hlavním systému, copy & paste funguje docela dobře. Třeba můj uživatelský případ - editor/IDE, terminál a... prohlížeč? Nic víc typicky nepotřebuji.

Díky tomu se vám nikdy nestane, že si poděláte konfiguraci čehosi v hlavním systému. Stalo se mi, že mi nenaběhli Xka nebo update nestabilní distribuce nebyl bez problémů. Oba operační systémy jsou oddělené, cokoliv nekalého provedete ve virtuálním stroji, nikterak neovlivní fyzický systém. Jak by také mohl?

A pak - hlavní systém je stabilní, virtual workstation může být technologicky stále o obrovský kus vpřed než hlavní. A nestane se vám, že hardware předběhne software - jako se to stalo v případě NVIDIA a jejího Optimus. Virtual manager by měl tvořit určitou stabilní vrstvu, které se nemění tak často.

Můj hlavní operační systém - a teď si někdo ublinkne - je Windows 7. Nechme problémy vidlí stranou - patří dle mého k těm stabilnějším systémům, pokud se v nich nehrabete příliš, a hlavně - dají se na nich hrát všechny použitelné hry. To je jádro pudla, na vývoj mám unstable Debian běžící ve VirtualBox a můj hlavní operační systém je prostředí, které považuji za stabilní. Nemusíte se omezit na jeden virtuální stroj, můžete mít na každý projekt virtuální mašinu. Samozřejmě - je nutné počítat s určitou režií na nějakou údržbu systému a podobně.

Rychlost a škálování
--------------------

Na notebooku, jež je poháněn procesorem s čtyřmi virtuálními jádry (dvěma fyzickými) s podporou pro virtualizaci, 8 GiB pamětí a 640 GiB diskem si zavirtualizujete bez velkých problémů. Tohle není high-end, tohle je dnešní mainstream až dokonce low-end s přikoupenou RAMkou. Virtuální stroj jsem si dokonce nastavil na využívání CPU pouze z 75 % - děláno skrze nějakou magii podpory pro virtualizaci v procesoru. Tím pádem nehučel větráček 

Rychlost HDD je trochu jiná, nepoužívá se fyzické HDD, zápis a čtení je zprostředkováno pomocí souboru. Tím Nějaký soubor s příponou .vdfs funguje jako nafugovací hard disk. Nastavte mu, ať si ukousne 100 GiB, ale reálně bude zabírat pouze to, co využije.

Protože je HDD jako soubor, není práce s diskem nejsvižnější, ale nikdy jsem nepocítil něco jako bídné líné nabíhání aplikací. Samozřejmě - pokud je projekt náročný na čtení a zápis na HDD a je to celé líné už na fyzickém stroji, zrovna dvakrát si asi nepomůžete. Konkrétní čísla nemám, záleží na hodně okolnostech, je třeba vyzkoušet.

Prostředí čistě zaměřené na vývoj
---------------------------------

Ve virtuálním prostředí můžete mít kompilátor pro XY, Apache, MySQL - a všechny ty krámy pro vývoj jsou připraveny v případě, kdy je zapnut virtuální systém. Neběží vám na pozadí v systému a nečekají, až si vzpomenete na nějaký vývoj. Když vypnete mašinu - uložíte její momentální stav. Stane se něco podobného, jako když dáte hibernovat fyzický stroj, ale operační systém o tom neví - obsah paměti se nasype na disk. Po spuštění virtuálního stroje máte všechno tak, jak jste ho měli při opuštění. Nezapínáte služby, nezapínáte IDE a nečekáte, až se rozehřeje. Up & running, krása.

V tomto prostředí není nutné mít eye candy vizuální efekty, hlavní směr toho všeho je vývoj - that's all.

Snapshotování
-------------

Uložit si momentální stav, totálně si rozkopat nevhodným způsobem systém, načíst předchozí stav - proč ne. Navíc je systém načten přesně v takovém stavu, v jakém ho člověk zanechal, to jest i s těmi svými otevřenými aplikacemi. Proč se strachovat o zdevastovanou infrastrukturu projektu? Devastujte ji bezpečně.

V praxi se celkem kašle na nějaký automatický deploy a jeho verzování - můžete sice používat puppet, chef nebo fabrik, ale patlejte se s tím pro malý tým.

Migrace na jiný stroj
---------------------

Další výhodou je migrace na jiný stroj. Izolované prostředí má nespornou výhodu v tom, že ho jednoduše vrznete na médium a hodíte jinam. Není nutné se omezit na svůj nový stroj, změňte hesla, username a dejte ho kolegovy. Bude mít ihned běžící prostředí.

A také nevýhody
---------------

Netbeans (alespoň ve verzi 6.8) je se sdílenou schránkou retard - schránka pro kopírování je se systémem synchronizována jen při focus na okno (okno se dostane do popředí z třeba minimalizovaného stavu). Strašlivě debilní řešení, pokud používáte prohlížeč v hlavním systému a Netbeans ve virtuálu. Ale čert je vem, PyCharm funguje dost dobře.

Může se stát, že aktuální kernel předežene možnosti virtuálního stroje a spolupracovat nebude. Výsledkem bude nestabilní nebo méně příjemně použitelné prostředí. Setkal jsem se s tím jen jednou, a to u distribuce Fedora a VMWare Player, kdy nešel nainstaloval VMWare ovladač, a to ani jejich open-source z repozitáře. Výsledkem bylo, že nešlo sdílet schránku pro kopírování a měnit velikost okna virtuálního počítače.

Stalo se mi, že virtuální počítač zdechnul. Přesněji přestal komunikovat, bylo nutné jej resetovat. Problém se týká hlavně VirtualBox. Ne náhodou jsou pády kernelu ve VirtualBox brány jako **CRAP** - to jest malá priorita.

Pokud jste ve virtuálním stroji, typicky klávesové zkratky pro ovládání hlasitosti asi nebudou moc dávat smysl pro onen virtuál. Možná lze nějak přenastavit ve virtual machine manager, netuším, nepotřeboval jsem.

VirtualBox a dost možná další nespolupracují s více obrazovkami tak, jak by si mohl někdo představovat. Konkrétně přesunout jakékoliv okno z virtual workstation není přirozeně možné, to samé se bude dít i ve fullscreen a seamless režimu. Já jsem byl každopádně spokojen, na jedné obrazovce ve fullscreen GNU/Linux, v druhé Windows.

Při novějším kernelu je potřeba aktualizovat "additions", jinak je dost možné, že nepůjde měnit velikost okna a podobně. Pokud používáte ty, které jsou součástí systému, asi nic řešit nemusíte, já jsem si zvykl po větší aktualizaci provést instalaci VBoxAdditions.

Závěrem
-------

Používám to v práci i doma a nestěžuji si. Koncept je dobrý, záleží jen na tom, co člověk od toho očekává a co vlastně doopravdy potřebuje.