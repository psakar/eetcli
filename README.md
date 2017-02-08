
# EETCLI - Commandline klient pro EET (etrzby.cz)

Tento klient slouží pro odesílání tržeb pomocí skriptu nebo externího programu. Základním zadáním bylo udělat co nejjednodušší implementaci. 
Je mi jasné, že čisté "C" by asi bylo mnohem čistější, ale PHP bylo jednodušší. 
Snažil jsem se o maximální multiplatformnost. Vzhledem k použití phar je jediná závislost na PHP, které je dostupné skoro na každém systému.
Klient sám o sobě neřeší žádnou evicenci dokladů a neřeší ani generování pořadových čísel dokladů. Pouze odesílá tržby na základě jasných požadavků.
V případě, že dojde k chybě, vrátí návratový kód. V opačném případě vrací FIK na standardním výstupu.
Klient by měl běžet i na relativně slabých systémech jako je raspberry PI. Může být spouštěn i jako externí program například z účetního software při vytvoření paragonu.

# Licence

Tento projekt je licencován pod GPL3. Zříkám se jakékoliv zodpovědnosti při používání tohoto programu.
I když se snažím vše odzkoušet, používání pro odesílání datových zpráv do EET je jen na Vás a Vaší zodpovědnosti.
Použil jsem komponenty třetích stran, které jsou rovněž šířeny pod otevřenou licencí, zejména
* [ondrejnov/eet](https://github.com/ondrejnov/eet)
* dealnews/console
* kherge/box

# Použití
Malý návod k použití je i součástí samotného příkazu.
```
./eetcli -h
This is commandline interface for Czech EET (etrzby.cz)
USAGE:
  eetcli.php  -h | --trzba celk_trzba [--cas dat_trzby] [--crt crt] [--dic dic] [--key key] [-n] [--output soubor] [-p] [--pc porad_cis] [--pokladna id_pokl] [--provozovna id_provoz] [-q] [--timeout mS] [--uuid uuid] [-v]

OPTIONS:
  --cas         dat_trzby   Datum a cas trzby
  --crt         crt         Certificate public key (pem format)
  --dic         dic         DIC
   -h                       Shows this help
  --key         key         Certificate private key (pem format)
   -n                       Overovaci rezim
  --output      soubor      Zapsat fik do souboru 
   -p                       Neprodukcni prostredi (playground)
  --pc          porad_cis   Poradove cislo
  --pokladna    id_pokl     ID pokladny
  --provozovna  id_provoz   ID provozovny
   -q                       Be quiet. Will override -v
  --timeout     mS          Timeout v milisekundach
  --trzba       celk_trzba  Celkova trzba v Kc
  --uuid        uuid        UUID
   -v                       Be verbose. Additional v will increase verbosity.
                            e.g. -vvv

Copyright Lukas Macura  2017-2017
```

## Konfigurace
Všechny parametry mohou být zadány přímo přes příkazovou řádku, nicméně pokud si vytvoříte ini soubor, můžete některé věci přednastavit.
Můžete vyjít z eetcli.ini.dist. Zkopírujte ho do složky eetcli a upravte pro Vaše použití. Nezapomeňte, že dokud nenakonfigurujete své údaje,
klient funguje v ověřovacím režimu s testovacími certifikáty!
```
[global]
;verbose=1
overovaci=1
;neprodukcni=1

[cert]
crt=./keys/EET_CA1_Playground-CZ1212121218.crt
key=./keys/EET_CA1_Playground-CZ1212121218.pem

[firma]
dic=CZ1212121218
pokladna=1
provozovna=181
```

## Příklady
Odešli tržbu 500,-Kč v ověřovacím režimu, použij klíč abcd.pem a certifikat abcd.crt. Použij pořadové číslo 1, pokladnu 1 a provozovnu 11.
```
eetcli --crt abcd.crt --key abcd.pem --pc 1 --pokladna 1 --provozovna 11 --trzba 500 -n
```
nebo v ostrém režimu
```
eetcli --crt abcd.crt --key abcd.pem --pc 1 --pokladna 1 --provozovna 11 --trzba 500
```

# Instalace

Teoreticky by mělo stačit stáhnout PHP a pak spouštět přímo eetcli. Návod pro instalaci pro jednotlivé systémy nebudu psát, kdo chce tento SW používat, jistě si to najde:)
Případně mi pošlete info a já můžu návod upravit.
Instalace na debian a podobných systémech:

```
sudo apt-get update
sudo apt-get install php-cli php5-curl
wget https://raw.githubusercontent.com/limosek/eetcli/0.1/bin/eetcli
chmod +x eetcli
./eetcli
```
Pokud chcete, můžete klienta přidat i do spustitelné cesty, takže bude zavolatelný z jakéhokoliv místa. 

Certifikáty pro EET jsou distribuovány jako .p12 soubory. Tento klient vyžaduje .pem a .crt soubory, které je potřeba extrahovar z .p12.
Pokud máte nainstalovaný program make a openssl, můžete si certifikát převést takto:
```
make pem P12=keys/muj_klic.p12 PASS=heslo_ke_klici 
```
V adresáři keys pak vzniknou nové soubory .pem a .crt, které můžete použít a odkázat se na ně například z ini souboru.

# Vývoj a kustomizace phar

Pokud chcete pomoci s vývojem, určitě neodmítnu :) 
Teoreticky si můžete vytvořit svůj vlastní phar archív a uložit do něj své klíče i ini soubor.
Stačí na to pustit make a vytvoří se eetcli.phar který je modifikován pro
vaše použití. *Součástí takového balíku jsou pak všechny klíče* z adresáře
keys tak  *eetcli.ini*. S tím jsou zase samozřejmě spojeny bezpečnostní věci, tedy že byste pak neměli phar archív nikdy dát z ruky, ale to už je mimo rámec tohoto dokumentu.
Pokud chcete vytvořit vývojové prostředí, potřebujete mít k dispozici php, phar, composer a make. Pro vytvoření phar archivu můžete použít:
```
git clone git@github.com:limosek/eetcli.git
cd eetcli
make clean
make
```

Pokud chcete změnit parametry pharu, můžete použít například:
```
make P12=cesta_ke_klici.p12 PASS=heslo_ke_klici 
```
Nezapomeňte, že součástí vytvořeného archivu jsou  *hesla, klíče ale i ini soubory*!

Pro vyčištění do původního stavu použijte
```
make distclean
```

Pro vytvoření čistého eetcli (v adresáři bin) pro účely další distribuce (bez klíčů a
osobních informací), použijte 
```
make distphar
```
