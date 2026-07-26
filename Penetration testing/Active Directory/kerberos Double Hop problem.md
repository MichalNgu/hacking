## 1. Co je „Double Hop“ problém a proč vzniká?

Problém „Double Hop“ (dvou skoků) nastává, když se pokoušíte použít Kerberos ověřování **přes dva nebo více síťových uzlů** (např. _Útočný stroj $\rightarrow$ Hostitel A (DEV01) $\rightarrow$ Doménový řetězec (DC01)_).

### Podstata problému:

- **Kerberos tikety nejsou hesla:** Kerberos tiket je podepsaný blok dat od KDC (Key Distribution Center), který opravňuje k přístupu k jednomu konkrétnímu zdroji (stroji).
    
- **Network Logon vs. Interactive Logon:** Při přihlášení přes WinRM (např. pomocí `Enter-PSSession` nebo nástroje `evil-winrm`) se provádí síťové ověření.
    
- **Absence hesel v paměti:** Při tomto typu ověření se na cílovém stroji (Hostitel A) **neukládá do paměti LSASS heslo ani NTLM hash uživatele** (všechna pole hesel jsou v Mimikatzu prázdná).
    
- **Chybějící TGT:** Cílovému stroji je předán pouze TGS (Ticket Granting Service) tiket pro přístup k němu samotnému. Stroji **není předán TGT (Ticket Granting Ticket)** uživatele.
    
- **Následek:** Když na Hostiteli A spustíte nástroj (např. _PowerView_) pro dotazování Active Directory, stroj nemá k dispozici uživatelovo TGT, nemůže požádat DC o nový TGS tiket pro LDAP a dotaz selže s chybou (např. `An operations error occurred`).
    

_(Poznámka: Pokud by se pro první skok použil např. PsExec, autentizace proběhne přes SMB a NTLM hash se do paměti uloží, takže k problému nedojde)._

## 2. Výjimka: Unconstrained Delegation

Pokud má Hostitel A v Active Directory povolenu konfiguraci **Unconstrained Delegation** (neomezenou delegaci), problém se neprojeví. V takovém případě se s požadavkem na Hostitele A automaticky posílá i TGT uživatele, které se uloží do paměti a Hostitel A jej může použít k žádosti o TGS tikety pro další stroje.

## 3. Praktická řešení (Workarounds)

Pokud při penetračním testu narazíte na omezení způsobené double hopem, existují dva hlavní způsoby, jak ho obejít:

### Řešení #1: Explicitní objekt `PSCredential`

Nejjednodušší metoda, kterou lze použít i v neinteraktivním shellu typu **evil-winrm** z Linuxu. Spočívá v tom, že v rámci vzdálené relace znovu vytvoříte objekt s přihlašovacími údaji a ten předáváte spouštěným příkazům.

1. **Vytvoření hesla v zabezpečeném řetězci:**
    
    PowerShell
    
    ```
    $SecPassword = ConvertTo-SecureString 'MojeHeslo123!' -AsPlainText -Force
    ```
    
2. **Vytvoření PSCredential objektu:**
    
    PowerShell
    
    ```
    $Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\backupadm', $SecPassword)
    ```
    
3. **Spuštění příkazu s parametrem `-Credential`:**
    
    PowerShell
    
    ```
    Get-DomainUser -spn -Credential $Cred | select samaccountname
    ```
    
    _Tímto způsobem explicitně poskytnete heslo pro ověření vůči DC a příkaz proběhne úspěšně._
    

### Řešení #2: Registrace konfigurace PSSession (`Register-PSSessionConfiguration`)

Tato pokročilejší metoda umožňuje přistupovat k dalším doménovým zdrojům bez nutnosti neustále předávat parametr `-Credential`. Funguje však pouze tehdy, pokud máte plnohodnotný PowerShell terminál (např. z Windows útočného stroje nebo přes RDP jump host) a administrátorská práva pro registraci relace na cílovém stroji.

1. **Registrace nové konfigurace relace s parametry `RunAs`:**
    
    PowerShell
    
    ```
    Register-PSSessionConfiguration -Name backupadmsess -RunAsCredential inlanefreight\backupadm
    ```
    
2. **Restartování služby WinRM na cílovém stroji:**
    
    PowerShell
    
    ```
    Restart-Service WinRM
    ```
    
3. **Připojení k nové relaci s definovanou konfigurací:**
    
    PowerShell
    
    ```
    Enter-PSSession -ComputerName DEV01 -Credential INLANEFREIGHT\backupadm -ConfigurationName backupadmsess
    ```
    

- **Jak to funguje:** Místní stroj nyní bude impersonovat vzdálený stroj v kontextu definovaného uživatele a všechny další požadavky budou korektně odesílány přímo na Domain Controller (což ověříte příkazem `klist`, který již ukáže načtené tikety pro krbtgt).