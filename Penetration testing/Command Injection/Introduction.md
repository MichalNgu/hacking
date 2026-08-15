**OS Command Injection** patří mezi nejkritičtější webové zranitelnosti. Vzniká v momentě, kdy aplikace předává neošetřený uživatelský vstup přímo do funkce operačního systému, která vykonává systémové příkazy.

### 📌 Přehled rizikových funkcí podle jazyků

Pokud vývojář použije dynamické spojování řetězců namísto bezpečné parametrizace, vzniká zranitelnost bez ohledu na použitou technologii:

|**Jazyk / Framework**|**Nebezpečné funkce pro spouštění příkazů**|**Vznik zranitelný kód (Anti-pattern)**|
|---|---|---|
|**PHP**|`system()`, `exec()`, `shell_exec()`, `passthru()`, `popen()`|`system("touch /tmp/" . $_GET['name'] . ".txt");`|
|**NodeJS**|`child_process.exec()`, `child_process.spawn()`|`exec(\`touch /tmp/${req.query.name}.txt`)`|
|**Python**|`os.system()`, `subprocess.Popen(..., shell=True)`|`os.system("ping -c 1 " + user_input)`|
|**Java**|`Runtime.getRuntime().exec()`, `ProcessBuilder`|`Runtime.getRuntime().exec("sh -c " + input)`|

### 💡 Jak dochází k manipulaci příkazu

Útočník využije speciální oddělovače příkazů v daném OS (např. `;`, `&&`, `|`, `\n`) k opuštění původního kontextu a připojení vlastního příkazu:

- **Původní zamýšlený příkaz:** `touch /tmp/soubor.txt`
    
- **Vstup útočníka:** `soubor.txt; id`
    
- **Výsledný vykonaný příkaz:** `touch /tmp/soubor.txt; id`
    

Dopadem úspěšné exploatace bývá kompletní převzetí kontroly nad backendovým serverem (Remote Code Execution - RCE) pod oprávněním uživatele, pod kterým běží webový server (např. `www-data`).