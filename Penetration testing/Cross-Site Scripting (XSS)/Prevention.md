Ochrana proti zranitelnostem typu XSS spočívá v zabezpečení celého toku dat od vstupu uživatele (_Source_) až po jeho vykreslení v prohlížeči (_Sink_). Efektivní obrana vyžaduje kombinaci opatření na straně klienta, serveru i v konfiguraci infrastruktury.

**Přehled obranných mechanismů proti XSS**

|**Vrstva**|**Mechanismus**|**Použité metody a nástroje**|**Účel a princip**|
|---|---|---|---|
|**Front-end**|Validace & Sanitizace|DOMPurify, Regex, `textContent`|Kontrola formátu a ošetření nebezpečných znaků přímo v prohlížeči.|
|**Back-end**|Sanitizace & HTML Encoding|`htmlentities()`, `filter_var()`, `html-entities`|Převod speciálních znaků (`<`, `>`, `&`) na bezpečné HTML entity před vykreslením.|
|**Server**|Bezpečnostní hlavičky|`Content-Security-Policy`, `HttpOnly`, `Secure`|Omezení spouštění skriptů a zamezení přístupu k relačním cookies.|

**1. Ochrana na klientské straně (Front-end)**

- **Validace vstupu:** Kontrola správného formátu (např. e-mailu nebo telefonního čísla) pomocí regulárních výrazů ještě před odesláním formuláře.
    
- **Sanitizace vstupu:** Použití knihoven jako **DOMPurify** pro odstranění škodlivého kódu ze zadaného textu (`DOMPurify.sanitize(dirty)`).
    
- **Bezpečné zacházení s DOM:**
    
    - **Eliminace zranitelných Sinks:** Vyhýbat se vlastnostem a metodám zápisu jako `innerHTML`, `outerHTML`, `document.write()` nebo `.html()` v jQuery.
        
    - **Používání bezpečných vlastností:** Pro zápis textu preferovat `textContent` nebo `innerText`, které automaticky neutralizují HTML znaky.
        

**2. Ochrana na straně serveru (Back-end)**

Jelikož front-endové kontroly lze snadno obejít přímým úpravou HTTP požadavku, serverová logika musí provádět vlastní validaci a kódování.

- **Validace vstupu:** Ověření dat pomocí vestavěných funkcí (např. `filter_var($_GET['email'], FILTER_VALIDATE_EMAIL)` v PHP).
    
- **Output HTML Encoding (Kódování výstupu):**
    
    - Nejvýznamnější prvek ochrany před Stored a Reflected XSS.
        
    - Převádí speciální znaky na neškodné HTML entity (např. `<` se změní na `&lt;`, `"` na `&quot;`).
        
    - V PHP se používají funkce `htmlentities()` nebo `htmlspecialchars()`, v Node.js balíček `html-entities`.
        

**3. Konfigurace serveru a bezpečnostní hlavičky**

Doplňková vrstva zabezpečení (Defense in Depth), která eliminuje riziko zneužití i v případě, že v kódování aplikace zůstane chyba:

- **Příznaky cookies (`HttpOnly` a `Secure`):** Zabraňují klientskému JavaScriptu přistupovat k relačním cookies (`document.cookie`) a vynucují přenos výhradně přes HTTPS.
    
- **Content Security Policy (CSP):** Hlavička (např. `Content-Security-Policy: script-src 'self'`) striktně definuje, z jakých zdrojů smí prohlížeč spouštět skripty.
    
- **`X-Content-Type-Options: nosniff`:** Zamezuje prohlížeči v interpretaci nahraných souborů jako jiného MIME typu (např. spustitelný JS maskovaný jako obrázek).
    
- **Web Application Firewall (WAF):** Automaticky detekuje a blokuje neobvyklé injektážní vzory v příchozím HTTP provozu.