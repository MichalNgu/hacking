Detekce toho, **co přesně** filtr nebo WAF (Web Application Firewall) blokuje, je klíčovým krokem k úspěšnému obcházení (Bypassing). Filtrování nejčastěji probíhá na dvou úrovních: přímo v kódování aplikace (PHP/NodeJS) nebo na síťové/aplikační bráně (WAF).

### 🔍 Aplikace vs. WAF: Jak poznat rozdíl

|**Vlastnost**|**Aplikační filtr (Backend kód)**|**Web Application Firewall (WAF)**|
|---|---|---|
|**Umístění**|Přímo v kódu webové aplikace (např. PHP `strpos()`, `preg_match()`).|Před aplikací (Cloudflare, ModSecurity, AWS WAF).|
|**Typická odpověď**|Vlastní chybová zpráva vykreslená v rámci šablony (např. _"Invalid input"_).|Standartní HTTP kódy (`403 Forbidden`, `406 Not Acceptable`) nebo generická blokovací stránka WAFu.|
|**Chování**|Blokuje velmi specifické řetězce nebo znaky definované vývojářem.|Detekuje širší vzory útoků (signatures, heuristika).|

### 🧪 Metodika izolace zakázaných znaků (Systematic Character Fuzzing)

Při identifikaci zakázaných znaků postupujte **izolací po jednom znaku**. Pokud posíláte kompletní payload typu `127.0.0.1; whoami`, nemůžete vědět, zda selhal kvůli středníku `;`, mezeře, nebo samotnému příkazu `whoami`.

1. **Základní validní vstup:**
    
    - `127.0.0.1` $\rightarrow$ **OK** (Potvrzení, že čístá IP projde)
        
2. **Izolované testování operátorů:**
    
    - `127.0.0.1;` $\rightarrow$ **Invalid input** _(Středník `;` je blokován)_
        
    - `127.0.0.1&` $\rightarrow$ **Invalid input** _(Ampersand `&` je blokován)_
        
    - `127.0.0.1|` $\rightarrow$ **Invalid input** _(Pipe `|` je blokován)_
        
    - `127.0.0.1%0a` $\rightarrow$ **200 OK / Vykoná se** _(Nová řádka `\n` předaná jako `%0a` často v blacklistech chybí!)_
        
3. **Testování mezer a speciálních znaků:**
    
    - `127.0.0.1%0awhoami` $\rightarrow$ Zjištění, zda je blokován samotný název příkazu nebo mezera.