Zranitelnost **Local File Inclusion (LFI)** vzniká v momentě, kdy aplikace nekriticky předává uživatelský vstup do funkcí pro načítání souborů a umožní číst citlivá data z backendového systému.

**Přehled scenářů a technik obcházení LFI**

|**Scénář / Omezení**|**Příklad zranitelného kódu**|**Princip obcházení a Payload**|
|---|---|---|
|**Přímé LFI**|`include($_GET['page']);`|Absolutní cesta k souboru: `/etc/passwd` nebo `C:\Windows\boot.ini`|
|**Pevný adresář**|`include("./lang/" . $_GET['page']);`|**Path Traversal:** sekvence `../` vrátí cestu do kořene: `../../../../etc/passwd`|
|**Prefix v názvu**|`include("lang_" . $_GET['page']);`|Přidání `/` na začátek payloadu (ošetří prefix jako složku): `/../../../etc/passwd`|
|**Vynucená přípona**|`include($_GET['page'] . ".php");`|Načtení selže na `/etc/passwd.php`. Vyžaduje pokročilé techniky (PHP Wrappers, Null Byte).|

**Detailní rozbor klíčových principů**

- **Mechanika Path Traversal (`../`)**
    
    - Každé `../` posouvá strukturu o jednu složku výše (např. z `/var/www/html/lang/` do `/var/www/html/`).
        
    - Opakování `../` nad úroveň kořenového adresáře (`/`) nic nerozbije — systém zůstane v `/`. V praxi se proto bezpečně používá delší řetězec (např. `../../../../../../etc/passwd`).
        
- **Omezení přímého načítání `.php` souborů**
    
    - Pokud se pokusíte přes LFI načíst lokální `.php` skript (např. `index.php`), aplikace jej nepředá jako čistý text, ale **spustí jej a vykreslí pouze HTML výstup**.
        
    - Pro zobrazení samotného zdrojového kódu PHP je nutné využít obcházení pomocí PHP Wrapperů (`php://filter`).
        
- **Útoky druhé úrovně (Second-Order LFI)**
    
    - Útočník nevkládá payload přímo do GET/POST parametru zobrazené stránky, ale **otráví záznam v databázi** (např. uložení uživatelského jména jako `../../../../etc/passwd`).
        
    - K samotnému načtení souboru dojde až ve chvíli, kdy jiná funkce aplikace načte hodnotu z databáze a použiije ji v souborové operaci (např. při generování avatara `/profile/$username/avatar.png`).