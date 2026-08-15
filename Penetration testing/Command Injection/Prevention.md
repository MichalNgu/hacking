Správná obrana proti zranitelnosti **Command Injection** se skládá ze tří hlavních pilířů: **architektonického návrhu** (nepoužívat systémový shell), **přísné validace a sanitizace vstupů na backendu** a **defenzivní konfigurace serveru**.

### 1. Architektonické pravidlo: Vyhněte se shellu

Nejúčinnější prevencí je **nepoužívat funkce, které předávají řetězce systémovému shellu** (např. v PHP `system()`, `exec()`, `passthru()`, `` ` `` nebo v Node.js `child_process.exec()`).

Místo spouštění externích příkazů OS používejte **rodné funkce daného programovacího jazyka**:

PHP

```
// ❌ ZRANITELNÝ PŘÍSTUP: Volání systémového příkazu ping
$output = system("ping -c 1 " . $_GET['ip']);

// 1. BEZPEČNÝ PŘÍSTUP: Použití rodných PHP funkcí
$ip = $_GET['ip'];
if (filter_var($ip, FILTER_VALIDATE_IP)) {
    // Pro ověření dostupnosti portu použijeme fsockopen místo ping
    $fp = @fsockopen($ip, 80, $errno, $errstr, 2);
    if ($fp) {
        echo "Host je dostupný";
        fclose($fp);
    }
}
```

Pokud _musíte_ spustit externí binární soubor, použijte API, které **nevolá příkazový interpret (shell)**, ale předává argumenty jako izolované pole (např. `execFile` nebo `spawn` v Node.js):

JavaScript

```
// ❌ ZRANITELNÉ: Spouští /bin/sh -c "ping -c 1 <input>"
const { exec } = require('child_process');
exec(`ping -c 1 ${userInput}`); 

// 1. BEZPEČNÉ: Obchází shell, argumenty jsou striktně odděleny
const { execFile } = require('child_process');
execFile('/bin/ping', ['-c', '1', userInput], (error, stdout) => {
    // userInput je předán přímo procesu ping jako 1 argument, nikoli shellu
});
```

### 2. Validace vs. Sanitizace vstupů

Pokud se uživatelskému vstupu nelze vyhnout, uplatňuje se koncept **Defense in Depth**:

- **Validace (Whitelisting):** Kontrola, zda vstup přesně odpovídá očekávanému formátu (např. platná IPv4/IPv6 adresa). Pokud neodpovídá, požadavek se okamžitě zamítne.
    
- **Sanitizace:** Odstranění všech neočekávaných a speciálních znaků, které by mohly sloužit jako řídící operátory shellu.
    

#### Ukázka v PHP:

PHP

```
// A) STRICT VALIDATION: Povolí pouze platné IP adresy
if (!filter_var($_POST['ip'], FILTER_VALIDATE_IP)) {
    die("Invalid IP address format.");
}

// B) SANITIZATION: Odstraní vše kromě alfanumerických znaků a tečky (Whitelist Regex)
$clean_ip = preg_replace('/[^A-Za-z0-9.]/', '', $_POST['ip']);
```

### 3. Srovnání: Zranitelný vs. Bezpečný vývoj

|**Oblast**|**Zranitelná praxe (Bad Practice)**|**Bezpečná praxe (Best Practice)**|
|---|---|---|
|**Volání procesů**|Předávání řetězců do `system()`, `exec()`|Použití rodného API / Knihoven nebo `execFile` bez shellu|
|**Filtrování**|**Blacklisting** zakázaných slov (`whoami`, `cat`) a znaků|**Whitelisting** (povolení pouze očekávaných znaků)|
|**Ošetření uvozovek**|Spoléhání na `escapeshellcmd()` nebo `escapeshellarg()`|Striktní Regex sanitizace (`preg_replace`)|
|**Validace**|Pouze na straně klienta (JavaScript v prohlížeči)|**Vždy na backendu** (PHP, Node.js, C#, Java)|

### 4. Zabezpečení serveru (Server Hardening)

I v případě, že se v aplikaci objeví chyba, správná konfigurace serveru výrazně omezí dopad případného útoku:

1. **Principle of Least Privilege (PoLP):**
    
    - Webový server (Apache/Nginx) i aplikační runtime (PHP-FPM, Node.js) musí běžet pod uživatelem s minimálními oprávněními (např. `www-data`).
        
    - Tento uživatel by neměl mít přístup k zapisování do kořenových adresářů webu ani do `/tmp` s právem spouštění (`noexec`).
        
2. **Zakázání nebezpečných funkcí v PHP (`php.ini`):**
    
    Ini, TOML
    
    ```
    disable_functions = exec, passthru, shell_exec, system, proc_open, popen, curl_exec, curl_multi_exec, parse_ini_file, show_source
    open_basedir = "/var/www/html:/tmp"
    ```
    
3. **Nasazení WAF (Web Application Firewall):**
    
    - Využití lokálních modulů (např. Apache `mod_security`) a síťových/cloudových WAFů (Cloudflare, Imperva) k detekci abnormalit v HTTP požadavcích.