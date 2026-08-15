## 📊 Interpretace protokolů (Log Messages) v SQLMapu

Při provádění testů nástrojem SQLMap generuje detekční engine řadu informativních hlášek. Porozumění těmto zprávám je klíčové pro pochopení, jaký typ zranitelnosti byl odhalen, jak fungují interní mechanizmy nástroje a jak následně zranitelnost správně zreportovat nebo ručně zneužít.

### 🔍 1. Inicializace a vlastnosti cílů

|**Hláška v protokolu**|**Význam a dopad na testování**|
|---|---|
|**`target URL content is stable`**|Odpovědi serveru na identické požadavky jsou stabilní. Nástroji to usnadňuje rozponávání odchylek způsobených injektovaným kódem.|
|**`GET parameter 'id' appears to be dynamic`**|Změna hodnoty parametru mění odpověď serveru, což potvrzuje, že parametr je dynamicky zpracováván (pravděpodobně spojen s databází).|
|**`heuristic (basic) test shows that ... might be injectable`**|Heuristický test (např. vložení úmyslně neplatných znaků `?id=1"'),)`) vyvolal databázovou chybu. Jde o indikátor (nikoli důkaz), který spouští hlubší testy.|
|**`heuristic (XSS) test shows that ... might be vulnerable`**|Rychlý vedlejší test odhalil, že parametr může být zranitelný vůči Cross-Site Scripting (XSS).|

### 🛠️ 2. Nastavení a optimalizace testování

- **`it looks like the back-end DBMS is 'MySQL' ... skip test payloads specific for other DBMSes?`**
    
    SQLMap rozpozná konkrétní databázový systém a nabízí přeskočení testů určených pro jiné databáze (např. PostgreSQL, Oracle), čímž zkracuje čas skenování.
    
- **`extending provided level (1) and risk (1) values?`**
    
    Pokud je detekována konkrétní databáze, nabídne rozšíření testovacích payloadů pro tuto databázi nad rámec výchozí úrovně testu.
    
- **`reflective value(s) found and filtering out`**
    
    Upozornění, že se části zadaného vstupu vrací zpět v odpovědi (junk/šum). SQLMap tyto odražené hodnoty automaticky filtruje, aby neovlivnily logiku vyhodnocování.
    

### ⏱️ 3. Speciální detekční mechanizmy

- **`GET parameter 'id' appears to be 'AND boolean-based blind ...' injectable (with --string="luther")`**
    
    Oznamuje nález zranitelnosti typu _Boolean-based blind_. Řetězec `luther` slouží SQLMapu jako konstantní příznak v HTML kódu k rozlišení stavu `TRUE` od `FALSE`.
    
- **`time-based comparison requires a larger statistical model...`**
    
    U zranitelností typu _Time-based blind_ SQLMap vytváří statistický model běžných odezev serveru, aby dokázal odlišit úmyslné zpoždění (`SLEEP`) od běžného síťového zpoždění (latency).
    
- **`automatically extending ranges for UNION query injection...`**
    
    Pokud je odhalena jiná funkční technika (např. Error-based), SQLMap automaticky zvýší limit testovaných parametrů pro náročnější _UNION query_ testy, protože šance na úspěch je vysoká.
    
- **`ORDER BY technique appears to be usable...`**
    
    Před odesláním `UNION` payloadů ověří funkčnost klauzule `ORDER BY`, což pomocí binárního vyhledávání drasticky zrychlí zjištění správného počtu sloupců.
    

### 🎯 4. Výsledky a ukládání dat

- **`GET parameter 'id' is vulnerable. Do you want to keep testing the others?`**
    
    Potvrzení, že parametr je prokazatelně zranitelný.
    
- **`sqlmap identified the following injection point(s)...`**
    
    Závěrečný souhrn všech ověřených a zneužitelných injekčních bodů (včetně typu, názvu a přesného použitého payloadu).
    
- **`fetched data logged to text files under '/home/user/.sqlmap/output/...'`**
    
    Všechna data, relace a výsledky se ukládají do lokálního adresáře. Při dalším spuštění nad stejným cílem SQLMap načte existující relaci a neopakuje již provedené testy.