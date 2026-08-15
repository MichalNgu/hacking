## 🎯 Burp Intruder (Web Fuzzer & Brute-force)

> **K čemu to je:** Slouží k automatickému testování vstupů – např. hledání skrytých adresářů a souborů (fuzzing), brute-force hesel/jmen, enumeraci parametrů či password sprayingu. Funguje jako grafická alternativa k nástrojům jako `ffuf`, `gobuster` nebo `wfuzz`.
> 
> ⚠️ **Omezení Community verze:** Bezplatná verze je uměle zpomalena na cca **1 požadavek za sekundu**. Je tedy vhodná pro krátké slovníky (pro dlouhé slovníky je lepší použít CLI nástroje nebo ZAP).

### 1. Odeslání požadavku do Intruderu

1. V `Proxy` ➔ `HTTP history` vybereš požadavek.
    
2. Stiskneš **`CTRL + I`** _(nebo pravé tlačítko ➔ Send to Intruder)_.
    
3. Přepneš se do záložky Intruder přes **`CTRL + SHIFT + I`**.
    

### 2. Positions (Nastavení pozice payloadu)

- Vybereš místo v požadavku, které chceš měnit (např. `GET /DIRECTORY/ HTTP/1.1`).
    
- Označíš slovo `DIRECTORY` a klikneš na tlačítko **Add §** (vznikne `§DIRECTORY§`).
    
- **Typy útoků (Attack Type):**
    
    - **Sniper:** Používá jeden slovník a postupně jím nahrazuje jednotlivé pozice.
        
    - **Cluster Bomb / Pitchfork:** Pro testování více pozic současně s různými slovníky (např. jméno + heslo).
        

### 3. Payloads (Slovníky a zpracování)

- **Payload Type (Typ slovníku):**
    
    - **Simple List:** Klasické načtení celého slovníku do paměti.
        
    - **Runtime file:** Načítá slovník ze souboru po řádcích během útoku (vhodné pro **obří slovníky**, šetří RAM).
        
    - **Character Substitution:** Vygeneruje permutace podle zadaných znaků.
        
- **Payload Configuration:**
    
    - Tlačítkem **Load** načteš soubor se slovníkem (např. ze `SecLists`).
        
- **Payload Processing (Pravidla pro úpravu slovníku):**
    
    - Umožňuje přidávat předpony/přípony nebo řádky filtrovat.
        
    - _Příklad filtrování tečkových souborů (`.htaccess` atd.):_ **Add** ➔ **Skip if matches regex** ➔ vzor: `^\..*$`
        
- **Payload Encoding:**
    
    - Ponechat zaškrtnuté pro automatické URL kódování speciálních znaků v payloadu.
        

### 4. Settings / Options (Filtrování výsledků)

- **Grep - Match:** Slouží ke zvýraznění odpovědí podle obsahu.
    
    - _Příklad pro adresáře:_ Vyčisti seznam (`Clear`), zadej `200 OK` a klikni na **Add**. V tabulce výsledků se vytvoří nový sloupec, podle kterého lze výsledky seřadit.
        
- **Grep - Extract:** Vytáhne konkrétní část textu z odpovědi serveru (užitečné u dlouhých odpovědí).
    
- **Resource Pool:** Nastavení počtu vláken a prodlev mezi požadavky.
    

### 5. Spuštění a vyhodnocení

1. Klikni na tlačítko **Start Attack**.
    
2. V okně s průběhem seřaď výsledky podle sloupce **Status**, **Length** nebo vytvořeného **Grep příznaku** (`200 OK`).
    
3. Nalezené cesty (např. `/admin/`) následně ověř manuálně v prohlížeči.