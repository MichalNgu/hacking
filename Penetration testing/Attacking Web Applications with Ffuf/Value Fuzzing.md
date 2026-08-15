### Co je Value Fuzzing?

Jakmile pomocí fuzzingu odhalíte funkční parametr (např. POST parametr `id`), dalším krokem je zjistit **správnou hodnotu**, kterou backend očekává (např. uživatelské ID, číselný token, PIN nebo specifické uživatelské jméno).

### 1. Vytvoření vlastního slovníku (Custom Wordlist)

Pro hodnoty parametrů často neexistuje univerzální předpřipravený slovník, protože každý parametr očekává jiný typ dat. Pokud parametr přijímá sekvenční číselné ID (např. od 1 do 1000), můžete si slovník snadno vygenerovat v prostředí Bash:

Bash

```
for i in $(seq 1 1000); do echo $i >> ids.txt; done
```

_(Příkaz postupně zapíše čísla od 1 do 1000 do souboru `ids.txt`.)_

### 2. Spuštění Value Fuzzingu v `ffuf`

Zástupný symbol `FUZZ` se v příkazu umístí na pozici **hodnoty parametru** (`id=FUZZ`):

Bash

```
ffuf -w ids.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'id=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -fs <VELIKOST_DEFAULTNI_ODPOVEDI>
```

### 3. Získání výsledku pomocí `curl`

Jakmile `ffuf` odhalí platnou hodnotu (vrátí jinou velikost odpovědi než chybová hláška `Invalid id!`), odošlete finální POST požadavek přes `curl` pro získání obsahu (vlajky/dat):

Bash

```
curl http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'id=<NALEZENA_HODNOTA>' -
```