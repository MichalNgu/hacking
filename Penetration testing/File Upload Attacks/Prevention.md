Zabezpečení nahrávání souborů vyžaduje vícevrstvou obranu (**Defense in Depth**). Pouhá kontrola přípon nestačí – účinná ochrana kombinuje striktní validaci obsahu, izolované úložiště mimo webroot a bezpečné konfigurace serveru.

**Přehled bezpečnostních opatření (Developer Checklist)**

|**Oblast**|**Opatření**|**Implementační detail**|
|---|---|---|
|**Přípona**|Kombinovaný Whitelist + Blacklist|Whitelist kontrolovaný na konci názvu (`/^.*\.(jpg\|jpeg\|png)$/i`) + pojistný Blacklist hledající zakázané vzory kdekoliv v názvu (`/\.ph(p\|ps\|ar\|tml)/i`).|
|**Obsah**|Magic Bytes & Content-Type|Ověření hlavičky `Content-Type` i reálných Magic Bytes (`mime_content_type()`). Obě hodnoty musí odpovídat povolenému MIME typu.|
|**Architektura**|Randomizace & Databáze|Soubory ukládejte pod náhodně vygenerovaným názvem (např. UUID). Původní sanovaný název ukládejte do databáze.|
|**Přístup**|Zákaz přímého přístupu & Webroot|Složku s uploady umístěte mimo `webroot` (nebo nastavte stav `403 Forbidden`). Ke stažení používejte dedikovaný skript `download.php`.|
|**Hlavičky**|Vynucení stažení|Používejte HTTP hlavičky `Content-Disposition: attachment` a `X-Content-Type-Options: nosniff` k zamezení renderingu skriptů v prohlížeči.|
|**Hardening**|Omezení PHP & Serveru|Vypněte nebezpečné funkce v `php.ini` (`disable_functions = system, exec, shell_exec, passthru`) a omezte přístup k adresářům přes `open_basedir`.|

**Bezpečná architektura stahování souborů (`download.php`)**

Přímý přístup k nahraným souborům v adresáři `/uploads/` je hlavní prerekvizitou pro spuštění webshellu. Namísto přímých odkazů servírujte soubory přes kontrolní skript:

1. **Autorizace & Kontrola cest (Anti-IDOR & Anti-LFI):** Skript musí ověřit, zda má přihlášený uživatel právo k danému souboru, a sanovat vstup pomocí `basename()`.
    
2. **Bezpečné předání souboru:**
    
    PHP
    
    ```
    // Příkaz pro prohlížeč ke stažení (nikoliv ke spuštění/vykreslení)
    header('Content-Type: ' . $mimeTypeFromDB);
    header('Content-Disposition: attachment; filename="' . $originalFilenameFromDB . '"');
    header('X-Content-Type-Options: nosniff');
    readfile($secretPathOnDisk);
    ```
    

**Doplňková bezpečnostní opatření**

- **Izolované úložiště:** Ukládejte uživatelské soubory na oddělený server (např. AWS S3) nebo do izolovaného Docker kontejneru. Případné RCE pak nekompromituje hlavní aplikační server.
    
- **Omezení velikosti a skenování:** Nastavte striktní limit pro velikost nahrávaných dat (`upload_max_filesize` v `php.ini`) a prodejejte nahrávané soubory antivirovým skenerem (např. ClamAV).
    
- **Potlačení chybových hlášení:** Vypněte zobrazování systémových chyb (`display_errors = Off`), aby útočník nemohl vyvolat výjimku k odhalení absolutní cesty k adresáři.