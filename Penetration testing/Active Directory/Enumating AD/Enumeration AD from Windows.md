# Ověřená enumerace AD z Windows (Credentialed Enumeration)

Při post-exploitaci na operačním systému Windows máme k dispozici jak vestavěné mechanismy, tak specializované PowerShell/.NET nástroje. Cílem je zmapovat konfiguraci domény, odhalit slabá místa (Kerberoasting, nestandardní trusty), najít privilegované účty a prohledat sdílené soubory.

## 🛠️ 1. Modul ActiveDirectory pro PowerShell

Pokud získáme přístup k hostiteli, který využívá administrátor, je vysoce pravděpodobné, že zde bude přítomen oficiální modul pro správu AD. Použití tohoto modulu je stealth (nenápadné), protože splývá s běžnou aktivitou správců sítě.

### A) Příprava modulu

PowerShell

```
# Ověření, zda je modul již načten
Get-Module

# Pokud modul chybí, importujeme ho manuálně
Import-Module ActiveDirectory
```

(Modul obsahuje celkem 147 různých cmdlets.)

### B) Klíčové příkazy pro recon

PowerShell

```
# 1. Základní informace o doméně (SID, child domény, funkční úroveň)
Get-ADDomain

# 2. Vyhledání účtů pro Kerberoasting (uživatelé s nastaveným SPN atributem)
Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName

# 3. Kontrola doménových vztahů důvěry (Trust Relationships)
Get-ADTrust -Filter *

# 4. Výpis názvů všech skupin v doméně
Get-ADGroup -Filter * | select name

# 5. Detailní informace o konkrétní skupině
Get-ADGroup -Identity "Backup Operators"

# 6. Výpis členů konkrétní skupiny
Get-ADGroupMember -Identity "Backup Operators"
```

> 📌 **Cíl zájmu z příkladu:** Účet `backupagent` je členem skupiny `Backup Operators`. Pokud tento účet kompromitujeme, získáme možnost ovládnout celou doménu skrze práva zálohování.

## 🦅 2. PowerView

PowerView (součást repozitáře PowerSploit) je komplexní PowerShell nástroj určený k získání situačního povědomí v AD. Umožňuje pokročilé vyhledávání relací, ACL oprávnění a podrobnou enumeraci.

### A) Základní import a spuštění

PowerShell

```
cd C:\Tools\
Import-Module .\PowerView.ps1
```

### B) Přehled nejdůležitějších funkcí (Cheat Sheet)

|**Funkce**|**Popis**|
|---|---|
|**`Get-Domain`**|Vrátí objekt aktuální (nebo specifikované) domény.|
|**`Get-DomainController`**|Vypíše seznam doménových kontrolerů.|
|**`Get-DomainUser`**|Zobrazí všechny uživatele nebo detaily o konkrétním objektu.|
|**`Get-DomainGroupMember`**|Vypíše členy konkrétní doménové skupiny.|
|**`Get-DomainTrustMapping`**|Provede kompletní zmapování všech viditelných trustů.|
|**`Test-AdminAccess`**|Ověří, zda má aktuální uživatel admin práva na lokálním/vzdáleném stroji.|
|**`Find-InterestingDomainAcl`**|Najde nestandardní ACL modifikace nastavené pro běžné uživatele.|
|**`Find-DomainShare`**|Vyhledá dostupné sdílené složky na strojích v doméně.|

### C) Praktické příklady použití PowerView

PowerShell

```
# Detailní průzkum konkrétního uživatele (kdy měnil heslo, UAC příznaky, skupiny)
Get-DomainUser -Identity mmorgan -Domain inlanefreight.local | Select-Object -Property name,samaccountname,description,memberof,pwdlastset,useraccountcontrol

# Rekurzivní výpis členů skupiny (včetně vnořených skupin - Nested Membership)
# Odhalí uživatele, kteří dědí práva nepřímo (např. přes skupinu Secadmins)
Get-DomainGroupMember -Identity "Domain Admins" -Recurse

# Mapování vztahů důvěry mezi doménami
Get-DomainTrustMapping

# Test administrátorských práv na vzdáleném počítači
Test-AdminAccess -ComputerName ACADEMY-EA-MS01

# Vyhledání účtů s nastaveným SPN (Kerberoasting)
Get-DomainUser -SPN -Properties samaccountname,ServicePrincipalName
```

## ⚡ 3. SharpView

SharpView je .NET port nástroje PowerView. Podporuje většinu stejných metod a argumentů.

- **Hlavní využití (OPSEC):** Je ideální v situacích, kdy cílová organizace vynucuje striktní omezení na spouštění PowerShellu (např. Constrained Language Mode), nebo pokud se chceme vyhnout detekcím navázaným na monitorování PowerShell skriptů.
    

PowerShell

```
# Zobrazení nápovědy a argumentů pro vybranou metodu
.\SharpView.exe Get-DomainUser -Help

# Spuštění enumerace specifického uživatele bez použití PowerShellu
.\SharpView.exe Get-DomainUser -Identity forend
```

## 📂 4. Sběr informací ze sdílených složek (Shares & Snaffler)

Příliš benevolentně nastavená oprávnění ke sdíleným složkám (např. složky IT oddělení nebo vývoje) často vedou k úniku konfiguračních souborů, SSH klíčů či nezašifrovaných hesel.

### Snaffler

Snaffler je automatizovaný .NET nástroj, který prochází aktivní hostitele, vyhledává čitelné sdílené složky a rekurzivně v nich pátrá po citlivých datech na základě definovaných regulárních výrazů. Musí běžet v kontextu doménového uživatele.

Bash

```
# Spuštění nástroje Snaffler
Snaffler.exe -d INLANEFREIGHT.LOCAL -s -v data -o snaffler.log
```

- `-s` – Vypisuje nalezené výsledky přímo do konzole v reálném čase.
    
- `-v data` – Nastavení úspornější verbosity, zobrazuje pouze relevantní nálezy.
    
- `-o` – Zápis kompletního výstupu do logovacího souboru (vhodné pro sdílení klientovi jako příloha reportu).
    

#### 🔍 Co Snaffler typicky odhalí (podle závažnosti):

- **Zelená/Černá barva (`{Green}`, `{Black}`):** Nalezené dostupné sdílené složky (např. `\Department Shares`, `\User Shares`, `\ZZZ_archive`).
    
- **Červená barva (`{Red}`):** Vysoce kritické soubory odpovídající pravidlům. Hledej přípony jako:
    
    - `.kdb`, `.kwallet` (databáze hesel a klíčenky).
        
    - `.key`, `.ppk` (privátní klíče, SSH klíče).
        
    - `.sqldump` (databázové zálohy s potenciálními údaji).