## ⚡ ZAP Fuzzer (Web Fuzzer)

> **Hlavní výhoda oproti Burpu:** ZAP Fuzzer **NENÍ rychlostně omezen** (v bezplatné verzi neexistuje žádné umělé zpomalení). Je plně použitelný pro velké slovníky s podporou více vláken.
> 
> **Mínus oproti Burpu:** Má o něco méně pokročilé funkce úpravy payloadů než Burp Intruder Pro.

### 1. Odeslání požadavku do Fuzzeru

1. Zachyť nebo najdi požadavek v proxy historii (např. `GET /test/ HTTP/1.1`).
    
2. Klikni pravým tlačítkem ➔ **Attack** ➔ **Fuzz**.
    

### 2. Locations (Nastavení pozice fuzzingu)

- Funguje jako _Positions_ v Burpu.
    
- Vyber slovo/hodnotu, kterou chceš nahrazovat (např. `test`), a klikni na **Add**.
    
- ZAP označené místo zvýrazní **zelenou značkou** a automaticky otevře okno pro konfiguraci payloadů.
    

### 3. Payloads (Slovníky)

Po kliknutí na **Add** v okně Payloads zvolíš typ vstupu:

- **File:** Načtení vlastní wordlisty ze souboru.
    
- **File Fuzzers (Integrované slovníky):** ZAP obsahuje zabudované databáze slovníků (např. Dirbuster, Seclists). Další lze doinstalovat přes ZAP Marketplace.
    
- **Numberzz:** Automatické generování číselných řad s definovaným krokem.
    

### 4. Processors (Úprava a kódování payloadů)

Umožňují upravit každý prvek ze slovníku před odesláním:

- **URL Decode / Encode:** Doporučeno přidat pro správné kódování speciálních znaků.
    
- **Prefix / Postfix String:** Přidá text před nebo za payload.
    
- **Hashe / Dekodéry:** MD5, SHA-1/256/512, Base64 atd.
    
- **Script:** Spuštění vlastního skriptu na každý prvek.
    
- _Užitečná funkce:_ Tlačítko **Generate Preview** ukáže, jak bude vypadat finální tvar v požadavku.
    

### 5. Options (Nastavení výkonu)

- **Concurrent Scanning Threads per Scan:** Nastavení počtu paratelních vláken (např. **20** pro výrazné zrychlení).
    
- **Strategie procházení (při více pozicích):**
    
    - **Depth first (Do hloubky):** Projde celý slovník na 1. pozici, než se posune na další.
        
    - **Breadth first (Do šířky):** Vyzkouší jeden prvek na všech pozicích, než vezme další slovo ze slovníku.
        

### 6. Spuštění a vyhodnocení

1. Klikni na **Start Fuzzer**.
    
2. Výslednou tabulku seřaď podle **Response code** (hledáš kód `200`), **Size Resp. Body** (odchylky ve velikosti odpovědi) nebo **RTT** (doba odezvy – užitečné např. pro Time-Based SQLi).
    

## 📊 Rychlé srovnání: Burp Intruder vs. ZAP Fuzzer

|**Vlastnost**|**Burp Intruder (Community)**|**ZAP Fuzzer (Zdarma)**|
|---|---|---|
|**Rychlost**|🛑 Zpomaleno (~1 req/s)|🚀 Neomezená (multi-threading)|
|**Vestavěné slovníky**|Pouze v Pro verzi|Ano (File Fuzzers)|
|**Pokročilé funkce/Pravidla**|Extrémně bohaté|🟡 Základní až střední|
|**Použití pro praxi**|Krátké testy / Malé slovníky|Dlouhé fuzzování / Velké slovníky|