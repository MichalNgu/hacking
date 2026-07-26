## 1. Úvod do laterálního pohybu (Lateral Movement)

- **Cíl:** Po získání prvotního přístupu (foothold) v doméně je cílem posunout se horizontálně (laterálně) či vertikálně na další stroje a dosáhnout úplného ovládnutí domény.
    
- **Tradiční přístup:** Pokud získáme účet s právy lokálního administrátora, lze použít útok **Pass-the-Hash (PtH)** přes protokol SMB.
    
- **Alternativní přístup:** Pokud lokální admin práva nemáme, hledáme uživatelská oprávnění k alternativním protokolům a službám pro vzdálenou správu.
    

V nástroji **BloodHound** se tyto vzdálené přístupy projevují jako specifické vazby (Edges):

- `CanRDP` – přístup přes vzdálenou plochu.
    
- `CanPSRemote` – přístup přes PowerShell Remoting (WinRM).
    
- `SQLAdmin` – administrátorská práva k MSSQL instanci.
    

## 2. Remote Desktop Protocol (RDP)

I bez administrátorských práv může mít uživatel možnost přihlásit se na vybrané servery (např. jump hosty nebo RDS / terminálové servery) prostřednictvím RDP.

- **Přínos pro útočníka:** Možnost eskalace práv přímo na hostiteli, sběr přihlašovacích údajů z paměti/registrů, hledání citlivých dat v souborech.
    
- **Enumerace (PowerView):** Zjištění členů lokální skupiny _Remote Desktop Users_:
    
    PowerShell
    
    ```
    Get-NetLocalGroupMember -ComputerName <NAZEV_PC> -GroupName "Remote Desktop Users"
    ```
    
    _(Často lze zjistit, že přístup má celá skupina `Domain Users`, což představuje vážné bezpečnostní riziko)._
    
- **Obranný audit:** Blue Team by měl pomocí BloodHoundu pravidelně spouštět předpřipravené dotazy jako _"Find Workstations/Servers where Domain Users can RDP"_.
    

## 3. Windows Remote Management (WinRM / PSRemoting)

Umožňuje spouštět příkazy nebo otevírat interaktivní textové relace na vzdáleném hostiteli pomocí PowerShellu. Od Windows 8 / Server 2012 existuje dedikovaná skupina **Remote Management Users**, která umožňuje WinRM přístup i ne-administrátorům.

### Postup z Windows (PowerShell)

Vytvoření objektu s přihlašovacími údaji a navázání relace:

PowerShell

```
$password = ConvertTo-SecureString "Heslo123" -AsPlainText -Force
$cred = new-object System.Management.Automation.PSCredential ("DOMÉNA\uzivatel", $password)
Enter-PSSession -ComputerName <NAZEV_PC> -Credential $cred
```

### Postup z Linuxu (`evil-winrm`)

Nástroj ze sady Ruby, který slouží jako WinRM klient:

Bash

```
# Instalace
gem install evil-winrm

# Připojení
evil-winrm -i <IP_ADRESA> -u <UZIVATEL> -p <HESLO>
```

## 4. Správa SQL Serveru (MSSQL Server Admin)

Účty s privilegii `sysadmin` na databázovém serveru MSSQL lze často odhalit pomocí Kerberoastingu, podvrhávání LLMNR/NBT-NS, password sprayingu nebo analýzou konfiguračních souborů (např. `web.config` pomocí nástroje **Snaffler**).

### Postup z Windows (`PowerUpSQL`)

1. Vyhledání MSSQL instancí v doméně:
    
    PowerShell
    
    ```
    Import-Module .\PowerUpSQL.ps1
    Get-SQLInstanceDomain
    ```
    
2. Spuštění testovacího dotazu proti instanci:
    
    PowerShell
    
    ```
    Get-SQLQuery -Instance "<IP>,1433" -username "DOMÉNA\uzivatel" -password "Heslo" -query 'Select @@version'
    ```
    

### Postup z Linuxu (`mssqlclient.py`)

Nástroj z toolkitu Impacket umožňující interaktivní přístup k MSSQL s podporou Windows autentizace (`-windows-auth`):

Bash

```
mssqlclient.py DOMÉNA/uzivatel@<IP_ADRESA> -windows-auth
```

### Eskalace na úroveň OS přes MSSQL

Pokud má kompromitovaný účet dostatečná práva (sysadmin), lze v SQL shellu povolit proceduru `xp_cmdshell`, která umožňuje vykonávat příkazy přímo v operačním systému Windows:

Plaintext

```
SQL> enable_xp_cmdshell
SQL> xp_cmdshell whoami /priv
```

- **Klíčový poznatek:** Příkazy vykonávané přes `xp_cmdshell` běží pod servisním účtem SQL serveru. Tento účet má téměř vždy aktivní oprávnění **`SeImpersonatePrivilege`**. To lze následně zneužít pomocí exploitů (např. _PrintSpoofer_, _JuicyPotato_, _RoguePotato_) k okamžité eskalaci práv na nejvyšší systémovou úroveň **SYSTEM**.
    

## 5. Shrnutí taktiky

- Útok a enumerace je **iterativní proces**. S každým nově ovládnutým účtem je nutné znovu provést skenování BloodHound / PowerView a zjistit, jaká nová přístupová práva (RDP, WinRM, SQL) tento účet získal.
    
- Servisní SQL přihlašovací údaje nalezené kdekoliv v síti představují kritické riziko, protože téměř garantují plné ovládnutí daného serveru (úroveň SYSTEM) skrze zneužití tokenů.