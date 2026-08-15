Při obcházení filtrů u LFI zranitelností závisí úspěch na způsobu, jakým aplikace sanitizuje uživatelský vstup a jakou verzi PHP či backendu používá.

|**Typ ochrany / Filtru**|**Princip obcházení**|**Příklad payloadu**|
|---|---|---|
|**Nerekurzivní filtr (`str_replace`)**|Odstraní pouze jednu vrstvu `../`. Zdvojení řetězce vytvoří po smazání nový `../`.|`....//....//....//etc/passwd` nebo `...\/...\/`|
|**Filtrování teček a lomítek**|Převod znaků do URL kódování (případně Double Encoding).|`%2e%2e%2f%2e%2e%2fetc%2fpasswd`|
|**Povolené složky (Regex / Approved Path)**|Začátek payloadu musí odpovídat schválené cestě, následovaný Directory Traversal.|`./languages/../../../../etc/passwd`|
|**Vynucená přípona (Null Byte)** _(PHP < 5.5)_|Řetězec se ukončí nulovým bajtem, cokoliv za ním (vč. `.php`) se ignoruje.|`/etc/passwd%00`|
|**Vynucená přípona (Path Truncation)** _(PHP < 5.3/5.4)_|Překročením limitu cesty (4096 znaků) dojde k odříznutí přípony na konci.|`non_exist/../../../etc/passwd/././...` _(~2048x)_|

**Detailní rozbor technik**

- **Nerekurzivní mazání (`str_replace('../', '', $input)`)**
    
    - Pokud filtr proběhne pouze jednou, ze vstupu `....//` se vymaže středový řetězec `../`, čímž ze zbylých znaků vznikne opět platný `../`.
        
    - Další funkční varianty: `..././`, `....\/`, `....////`.
        
- **URL Encoding a Double Encoding**
    
    - Pokud aplikace přímo blokuje znaky `.` a `/`, kódování celé cesty (`.` = `%2e`, `/` = `%2f`) filtr obejde, pokud backend dekóduje vstup až v samotné funkci `include()`.
        
    - **Double Encoding:** `%252e%252e%252f` (využívá se, pokud aplikace provádí automatické dekódování ještě před průchodem filtrem).
        
- **Prefix schváleného adresáře (Approved Path / Regex)**
    
    - Aplikace ověřuje, zda cesta začíná např. `./languages/`.
        
    - **Obcházení:** Splní se podmínka na začátku cesty a následně se vyskočí zpět přes Traversal: `./languages/../../../../etc/passwd`.
        
- **Historické techniky obcházení přípony (`.php`)**
    
    - **Null Byte (`%00`):** Funguje v PHP < 5.5. Znak `%00` v paměti označuje konec C-řetězce, takže backend jakoukoliv příponu připojenou za nulový bajt zcela ignoruje.
        
    - **Path Truncation:** Funguje v PHP < 5.3/5.4. Linuxové systémy a starší verze PHP mají limit délky cesty 4096 bajtů. Generováním dlouhé cesty pomocí sekvence `/././.` se natvrdo připojená přípona `.php` na konci odřízne:
        
        Bash
        
        ```
        # Generování Path Truncation payloadu:
        echo -n "non_existing/../../../etc/passwd/" && for i in {1..2048}; do echo -n ".
        ```