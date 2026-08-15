**Reflected (Odražený) XSS** je nepersistentní typ zranitelnosti, kdy backend server přijme neošetřený uživatelský vstup a ihned jej odrazí zpět v odpovědi (např. v chybové hlášce nebo výsledcích vyhledávání).

|**Vlastnost**|**Reflected XSS**|**Stored XSS**|
|---|---|---|
|**Ukládání v DB**|**Ne** (odpověď je dočasná)|**Ano** (uloženo v databázi)|
|**Rozsah dopadu**|Pouze uživatel, který otevře škodlivý odkaz|Všichni uživatelé navštěvující danou stránku|
|**Vektor útoku**|Útočné URL odkazy / GET parametry|Formulářové vstupy (komentáře, profily)|

📌 **Postup testování a zneužití Reflected XSS**

1. **Identifikace injekčního bodu**
    
    - Vlož do formuláře nebo parametru vyhledávání testovací řetězec (např. `test`).
        
    - Zkontroluj, zda se vrací v HTML (např. _„Úkol 'test' nebylo možné přidat“_).
        
2. **Ověření zranitelnosti (Testovací Payload)**
    
    - Vlož payload: `<script>alert(window.origin)</script>`
        
    - Pokud se zobrazí dialogové okno a v HTML kódu (`Ctrl+U`) je kód v nezměněné podobě, zranitelnost je potvrzena.
        
    - _Ověření nepersistentnosti:_ Po obnovení stránky (`F5`) se alert již nezobrazí.
        
3. **Kontrola HTTP metody (`Ctrl+Shift+I`)**
    
    - Otevři Vývojářské nástroje $\rightarrow$ záložka **Network**.
        
    - Odeslání payloadu zkontroluj v logu:
        
        - **GET metoda:** Data jsou vnořena přímo do URL (snadná konstrukce útočného odkazu).
            
        - **POST metoda:** Data jsou v těle požadavku (vyžaduje externí formulář útočníka).
            
4. **Sestavení útočného odkazu (pro GET požadavky)**
    
    - Zkopíruj finální URL adresu z adresního řádku nebo záložky Network.
        
    - **Finální tvar odkazu pro oběť:**
        
        `http://SERVER_IP:PORT/index.php?task=<script>alert(window.origin)</script>`
        
    - V momentě, kdy oběť na tento odkaz klikne, spustí se škodlivý JavaScript v kontextu její relace.