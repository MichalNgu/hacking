## 🐍 Medusa

> **Co to je:** Medusa je rychlý, vysoce paralelní a modulární nástroj pro online brute-force útoky na přihlašovací systémy vzdálených služeb. Je navržen jako efektivní alternativa k nástrojům jako THC-Hydra nebo Ncrack.

### ⚙️ Základní syntaxe a klíčové parametry

Bash

```
medusa [možnosti_cíle] [přihlašovací_údaje] -M modul [možnosti_modulu]
```

|**Parametr**|**Význam**|**Příklad použití**|
|---|---|---|
|**`-h HOST` / `-H SOUBOR`**|Jeden cíl (`-h`) nebo seznam cílů ze souboru (`-H`).|`medusa -h 192.168.1.10`|
|**`-u USER` / `-U SOUBOR`**|Jedno jméno (`-u`) nebo seznam jmen ze souboru (`-U`).|`medusa -u admin` nebo `-U users.txt`|
|**`-p PASS` / `-P SOUBOR`**|Jedno heslo (`-p`) or seznam hesel ze slovníku (`-P`).|`medusa -P passwords.txt`|
|**`-M MODUL`**|Specifikuje modul/službu pro útok (`ssh`, `ftp`, `http`, `mysql`...).|`medusa -M ssh`|
|**`-m "OPCE"`**|Další parametry předávané konkrétnímu modulu.|`-m DIR:/login.php`|
|**`-t KROKY`**|Počet paralelně spuštěných úloh/vláken.|`-t 4`|
|**`-f` / `-F`**|Zastaví útok po prvním úspěšném přihlášení na hostiteli (`-f`) nebo celkově (`-F`).|`medusa -f`|
|**`-n PORT`**|Nastaví nestandardní port služby.|`-n 2222`|
|**`-e ns`**|Testuje prázdná hesla (`n`) a hesla shodná s uživatelským jménem (`s`).|`medusa -e ns`|
|**`-v ÚROVEŇ`**|Nastavení podrobnosti výstupu (1 až 6).|`-v 4`|

### 🔌 Podporované moduly a příklady použití

|**Služba**|**Modul**|**Přiklad použití**|
|---|---|---|
|**SSH**|`ssh`|`medusa -h 192.168.0.100 -U users.txt -P pass.txt -M ssh`|
|**HTTP Basic Auth**|`http`|`medusa -H web_servers.txt -U users.txt -P pass.txt -M http -m GET`|
|**Webové formuláře**|`web-form`|`medusa -M web-form -h example.com -U users.txt -P pass.txt -m FORM:"username=^USER^&password=^PASS^:F=Invalid"`|
|**FTP**|`ftp`|`medusa -M ftp -h 192.168.1.100 -u admin -P pass.txt`|
|**RDP / VNC**|`rdp` / `vnc`|`medusa -M rdp -h 192.168.1.100 -u admin -P pass.txt`|
|**Databáze (MySQL)**|`mysql`|`medusa -M mysql -h 192.168.1.100 -u root -P pass.txt`|

### 💡 Specifické funkce Medusy

- **Rychlé prověřování prázdných a továrních hesel (`-e ns`):**
    
    Umožňuje automaticky otestovat, zda účet nemá prázdné heslo (`n` = null) nebo heslo identické s uživatelským jménem (`s` = same):
    
    Bash
    
    ```
    medusa -h 10.0.0.5 -U usernames.txt -e ns -M ssh
    ```
    
- **Hromadný útok na více serverů současně (`-H`):**
    
    Medusa dokáže zefektivnit síťový útok tím, že zpracovává seznamIP adres paralelně napříč vlákny.