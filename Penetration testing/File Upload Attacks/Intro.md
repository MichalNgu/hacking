Funkcionalita nahrávání souborů patří k nejrizikovějším prvkům webových aplikací. Pokud aplikace nedostatečně validuje nahrané soubory, může útočník uložit a spustit kód přímo na backendovém serveru.

**Přehled rizik a typů útoků při nahrávání souborů**

|**Typ dopadu / Útoku**|**Princip zranitelnosti**|**Výsledný dopad**|
|---|---|---|
|**Arbitrary File Upload (RCE)**|Absence nebo selhání validace přípon a obsahu na serveru.|Nahrání web shellu a spuštění systémových příkazů na serveru.|
|**Stored XSS**|Nahrání HTML nebo SVG souboru s vloženým JavaScriptem.|Spuštění klientského skriptu v prohlížeči ostatních uživatelů při otevření souboru.|
|**XXE Injection**|Zpracování uživatelských XML, DOCX nebo PDF souborů nezabezpečeným parserem.|Čtení lokálních souborů serveru nebo provádění SSRF požadavků.|
|**Denial of Service (DoS)**|Chybějící limity na velikost souboru nebo počet nahraných položek.|Vyčerpání kapacity disku, paměti RAM nebo procesorového času serveru.|
|**Přepsání souborů (Overwriting)**|Zachování původního názvu a cesty bez unikátního přejmenování.|Přepsání kritických konfiguračních nebo aplikačních souborů.|

**Hlavní příčiny zranitelností**

- **Kontrola pouze na klientské straně (Client-side validation):** Spoléhání na JavaScriptové filtry v prohlížeči, které lze snadno obejít úpravou HTTP požadavku.
    
- **Neúčinná serverová validace:** Pouhá kontrola HTTP hlavičky `Content-Type` nebo kontrola přípony pomocí černé listiny (_blacklist_), která nepočítá s alternativními příponami.
    
- **Zastaralé závislosti:** Používání neaktualizovaných knihoven pro zpracování obrázků a dokumentů (např. historické zranitelnosti v ImageMagick).