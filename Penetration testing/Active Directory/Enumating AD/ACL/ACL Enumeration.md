# Enumerace ACL (Access Control List)

Pojďme se ponořit do enumerace ACL pomocí nástroje PowerView a projít si grafické znázornění v BloodHoundu. Poté si ukážeme několik scénářů a útoků, kde lze nalezené položky ACE (Access Control Entry) využít k získání dalšího přístupu do interního prostředí.

## Enumerace ACL pomocí PowerView

K enumeraci ACL můžeme použít PowerView, ale prohrabávání se všemi výsledky by bylo extrémně časově náročné a pravděpodobně i nepřesné. Pokud například spustíme funkci `Find-InterestingDomainAcl`, obdržíme obrovské množství informací, ve kterých je těžké se zorientovat:

### Použití Find-InterestingDomainAcl

PowerShell

```
PS C:\htb> Find-InterestingDomainAcl
ObjectDN                : DC=INLANEFREIGHT,DC=LOCAL
AceQualifier            : AccessAllowed
ActiveDirectoryRights   : ExtendedRight
ObjectAceType           : ab721a53-1e2f-11d0-9819-00aa0040529b
AceFlags                : ContainerInherit
AceType                 : AccessAllowed
ObjectInheritanceFlags        : ContainerInherit
SecurityIdentifier      : S-1-5-21-3842939050-3880317879-2865463114-5189
IdentityReferenceName   : Exchange Windows Permissions
IdentityReferenceDomain : INLANEFREIGHT.LOCAL
IdentityReferenceDN     : CN=Exchange Windows Permissions,OU=Microsoft Exchange Security Groups,DC=INLANEFREIGHT,DC=LOCAL
IdentityReferenceClass  : group

ObjectDN                : DC=INLANEFREIGHT,DC=LOCAL
AceQualifier            : AccessAllowed
ActiveDirectoryRights   : ExtendedRight
ObjectAceType           : 00299570-246d-11d0-a768-00aa006e0529
AceFlags                : ContainerInherit
AceType                 : AccessAllowed
ObjectInheritanceFlags        : ContainerInherit
SecurityIdentifier      : S-1-5-21-3842939050-3880317879-2865463114-5189
IdentityReferenceName   : Exchange Windows Permissions
IdentityReferenceDomain : INLANEFREIGHT.LOCAL
IdentityReferenceDN     : CN=Exchange Windows Permissions,OU=Microsoft Exchange Security Groups,DC=INLANEFREIGHT,DC=LOCAL
IdentityReferenceClass  : group
<ZKRÁCENO>
```

Pokud se pokusíme analyzovat všechna tato data během časově omezeného testu, pravděpodobně je nestihneme projít celá nebo nenajdeme nic zajímavého před koncem assessmentu. Existuje však způsob, jak PowerView používat efektivněji – provádět **cílenou enumeraci** počínaje uživatelem, kterého již máme pod kontrolou.

Zaměřme se na uživatele `wley`, kterého jsme získali dříve. Zjistíme, zda má tento uživatel nějaká zajímavá ACL práva, která bychom mohli využít. Nejprve musíme získat SID našeho cílového uživatele, abychom mohli efektivně vyhledávat.

PowerShell

```
PS C:\htb> Import-Module .\PowerView.ps1
PS C:\htb> $sid = Convert-NameToSid wley
```

K provedení cíleného vyhledávání pak můžeme použít funkci `Get-DomainObjectACL`. V níže uvedeném příkladu používáme tuto funkci k vyhledání všech doménových objektů, nad kterými má náš uživatel práva, a to mapováním uživatelského SID (pomocí proměnné `$sid`) na vlastnost `SecurityIdentifier`.

> [!NOTE]
> 
> Pokud vyhledáváme bez parametru `-ResolveGUIDs`, uvidíme výstup jako níže, kde právo `ExtendedRight` neposkytuje jasný obraz o tom, jaké konkrétní ACE právo má uživatel `wley` nad uživatelem `damundsen`. Je to proto, že vlastnost `ObjectAceType` vrací hodnotu GUID, která není pro člověka čitelná. Tento příkaz může v závislosti na velikosti prostředí běžet 1–2 minuty.

### Použití Get-DomainObjectACL

PowerShell

```
PS C:\htb> Get-DomainObjectACL -Identity * | ? {$_.SecurityIdentifier -eq $sid}

ObjectDN               : CN=Dana Amundsen,OU=DevOps,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
ObjectSID              : S-1-5-21-3842939050-3880317879-2865463114-1176
ActiveDirectoryRights  : ExtendedRight
ObjectAceFlags         : ObjectAceTypePresent
ObjectAceType          : 00299570-246d-11d0-a768-00aa006e0529
InheritedObjectAceType : 00000000-0000-0000-0000-000000000000
BinaryLength           : 56
AceQualifier           : AccessAllowed
IsCallback             : False
OpaqueLength           : 0
AccessMask             : 256
SecurityIdentifier     : S-1-5-21-3842939050-3880317879-2865463114-1181
AceType                : AccessAllowedObject
AceFlags               : ContainerInherit
IsInherited            : False
InheritanceFlags       : ContainerInherit
PropagationFlags       : None
AuditFlags             : None
```

Mohli bychom vyhledat GUID hodnotu `00299570-246d-11d0-a768-00aa006e0529` na internetu a zjistit, že uživatel má právo vynutit změnu hesla u druhého uživatele (`Force Change Password`). Alternativně můžeme použít reverzní vyhledávání přímo v PowerShellu a namapovat název práva zpět k danému GUID.

> [!WARNING]
> 
> Pokud již byl PowerView importován, níže uvedená rutina (cmdlet) může vyvolat chybu. V takovém případě může být nutné spustit ji v nové relaci PowerShellu.

### Reverzní vyhledávání a mapování GUID hodnoty

PowerShell

```
PS C:\htb> $guid= "00299570-246d-11d0-a768-00aa006e0529"
PS C:\htb> Get-ADObject -SearchBase "CN=Extended-Rights,$((Get-ADRootDSE).ConfigurationNamingContext)" -Filter {ObjectClass -like 'ControlAccessRight'} -Properties * | Select Name,DisplayName,DistinguishedName,rightsGuid | ?{$_.rightsGuid -eq $guid} | fl

Name              : User-Force-Change-Password
DisplayName       : Reset Password
DistinguishedName : CN=User-Force-Change-Password,CN=Extended-Rights,CN=Configuration,DC=INLANEFREIGHT,DC=LOCAL
rightsGuid        : 00299570-246d-11d0-a768-00aa006e0529
```

Tento postup nám sice dal odpověď, ale během reálného testu by byl neefektivní. PowerView obsahuje parametr **`-ResolveGUIDs`**, který tento převod provede automaticky za nás. Všimněte si, jak se změní výstup, pokud tento parametr zahrneme – vlastnost `ObjectAceType` se zobrazí v lidsky čitelném formátu jako `User-Force-Change-Password`.

### Použití parametru -ResolveGUIDs

PowerShell

```
PS C:\htb> Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid} 

AceQualifier           : AccessAllowed
ObjectDN               : CN=Dana Amundsen,OU=DevOps,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights  : ExtendedRight
ObjectAceType          : User-Force-Change-Password
ObjectSID              : S-1-5-21-3842939050-3880317879-2865463114-1176
InheritanceFlags       : ContainerInherit
BinaryLength           : 56
AceType                : AccessAllowedObject
ObjectAceFlags         : ObjectAceTypePresent
IsCallback             : False
PropagationFlags       : None
SecurityIdentifier     : S-1-5-21-3842939050-3880317879-2865463114-1181
AccessMask             : 256
AuditFlags             : None
IsInherited            : False
AceFlags               : ContainerInherit
InheritedObjectAceType : All
OpaqueLength           : 0
```

Proč jsme si procházeli tento příklad, když jsme mohli hned použít `-ResolveGUIDs`? Je nezbytné rozumět tomu, co naše nástroje na pozadí dělají, a mít v záloze alternativní metody pro případ, že nástroj selže nebo bude zablokován.

Pojďme se podívat, jak totéž provést pomocí vestavěných cmdletů `Get-Acl` a `Get-ADUser`, které mohou být k dispozici přímo na klientském systému. Schopnost provádět toto vyhledávání bez externích nástrojů (jako PowerView) je velkou výhodou v situacích, kdy jsme striktně omezeni na prostředky přítomné v systému.

Tento vestavěný příkaz sice není tak efektivní a v rozsáhlém prostředí může běžet mnohem déle než PowerView, ale funguje. Nejprve si vytvoříme seznam všech doménových uživatelů:

### Vytvoření seznamu doménových uživatelů

PowerShell

```
PS C:\htb> Get-ADUser -Filter * | Select-Object -ExpandProperty SamAccountName > ad_users.txt
```

Poté přečteme každý řádek souboru pomocí cyklu `foreach` a použijeme cmdlet `Get-Acl` k načtení informací o ACL pro každého doménového uživatele. Následně vybereme vlastnost `Access` (která obsahuje přístupová práva) a vyfiltrujeme vlastnost `IdentityReference` na uživatele, kterého kontrolujeme (`wley`).

### Foreach cyklus pro vestavěné vyhledávání

PowerShell

```
PS C:\htb> foreach($line in [System.IO.File]::ReadLines("C:\Users\htb-student\Desktop\ad_users.txt")) {get-acl "AD:\$(Get-ADUser $line)" | Select-Object Path -ExpandProperty Access | Where-Object {$_.IdentityReference -match 'INLANEFREIGHT\\wley'}}

Path                  : Microsoft.ActiveDirectory.Management.dll\ActiveDirectory:://RootDSE/CN=Dana Amundsen,OU=DevOps,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights : ExtendedRight
InheritanceType       : All
ObjectType            : 00299570-246d-11d0-a768-00aa006e0529
InheritedObjectType   : 00000000-0000-0000-0000-000000000000
ObjectFlags           : ObjectAceTypePresent
AccessControlType     : Allow
IdentityReference     : INLANEFREIGHT\wley
IsInherited           : False
InheritanceFlags      : ContainerInherit
PropagationFlags      : None
```

Jakmile máme tato data, můžeme použít stejné metody popsané výše k převodu GUID na lidsky čitelný formát.

Abychom si to shrnuli: začali jsme u uživatele `wley` a nyní máme kontrolu nad uživatelem `damundsen` prostřednictvím rozšířeného práva `User-Force-Change-Password`. Pojďme pomocí PowerView zjistit, kam nás může kontrola nad účtem `damundsen` posunout dále.

### Další enumerace práv pomocí účtu damundsen

PowerShell

```
PS C:\htb> $sid2 = Convert-NameToSid damundsen
PS C:\htb> Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid2} -Verbose

AceType               : AccessAllowed
ObjectDN              : CN=Help Desk Level 1,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights : ListChildren, ReadProperty, GenericWrite
OpaqueLength          : 0
ObjectSID             : S-1-5-21-3842939050-3880317879-2865463114-4022
InheritanceFlags      : ContainerInherit
BinaryLength          : 36
IsInherited           : False
IsCallback            : False
PropagationFlags      : None
SecurityIdentifier    : S-1-5-21-3842939050-3880317879-2865463114-1176
AccessMask            : 131132
AuditFlags            : None
AceFlags              : ContainerInherit
AceQualifier          : AccessAllowed
```

Nyní vidíme, že náš uživatel `damundsen` má **`GenericWrite`** privilegia nad skupinou `Help Desk Level 1`. To kromě jiného znamená, že do této skupiny můžeme přidat jakéhokoli uživatele (nebo sami sebe) a zdědit veškerá práva, která má tato skupina přidělena. Vyhledání přímých práv této skupiny sice nevrací nic zajímavého, ale podívejme se, zda není tato skupina vnořená do jiné skupiny.

Vnořené členství (nested group membership) znamená, že všichni uživatelé ve skupině A automaticky dědí všechna práva jakékoli skupiny, ve které je skupina A členem. Rychlé vyhledání nám ukazuje, že skupina `Help Desk Level 1` je vnořená do skupiny `Information Technology`.

### Prověření skupiny Help Desk Level 1 pomocí Get-DomainGroup

PowerShell

```
PS C:\htb> Get-DomainGroup -Identity "Help Desk Level 1" | select memberof

memberof                                                                      
--------                                                                      
CN=Information Technology,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
```

To je spousta informací ke zpracování! Zrekapitulujme si náš postup:

1. Máme pod kontrolou uživatele `wley`.
    
2. Zjistili jsme, že můžeme vynutit změnu hesla (`ForceChangePassword`) u uživatele `damundsen`.
    
3. Zjistili jsme, že uživatel `damundsen` může přidávat členy do skupiny `Help Desk Level 1` díky právu `GenericWrite`.
    
4. Skupina `Help Desk Level 1` je vnořená do skupiny `Information Technology`, což znamená, že její členové dědí veškerá práva této nadřazené skupiny.
    

Nyní se podívejme, zda členové skupiny `Information Technology` mohou dělat něco zajímavého. Další vyhledávání pomocí `Get-DomainObjectACL` nám ukazuje, že členové této skupiny mají práva **`GenericAll`** nad uživatelem `adunn`. To znamená, že bychom mohli:

- Modifikovat členství ve skupině.
    
- Vynutit změnu hesla.
    
- Provést cílený Kerberoasting a pokusit se cracknout heslo tohoto uživatele.
    

### Prověření skupiny Information Technology

PowerShell

```
PS C:\htb> $itgroupsid = Convert-NameToSid "Information Technology"
PS C:\htb> Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $itgroupsid} -Verbose

AceType               : AccessAllowed
ObjectDN              : CN=Angela Dunn,OU=Server Admin,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights : GenericAll
OpaqueLength          : 0
ObjectSID             : S-1-5-21-3842939050-3880317879-2865463114-1164
InheritanceFlags      : ContainerInherit
BinaryLength          : 36
IsInherited           : False
IsCallback            : False
PropagationFlags      : None
SecurityIdentifier    : S-1-5-21-3842939050-3880317879-2865463114-4016
AccessMask            : 983551
AuditFlags            : None
AceFlags              : ContainerInherit
AceQualifier          : AccessAllowed
```

Nakonec se podívejme, zda má uživatel `adunn` nějaký zajímavý přístup, který by nás mohl posunout k našemu hlavnímu cíli.

### Hledání zajímavých oprávnění uživatele adunn

PowerShell

```
PS C:\htb> $adunnsid = Convert-NameToSid adunn 
PS C:\htb> Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $adunnsid} -Verbose

AceQualifier           : AccessAllowed
ObjectDN               : DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights  : ExtendedRight
ObjectAceType          : DS-Replication-Get-Changes-In-Filtered-Set
...
AceQualifier           : AccessAllowed
ObjectDN               : DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights  : ExtendedRight
ObjectAceType          : DS-Replication-Get-Changes
...
<ZKRÁCENO>
```

Výše uvedený výstup ukazuje, že uživatel `adunn` má nad doménovým objektem práva **`DS-Replication-Get-Changes`** a **`DS-Replication-Get-Changes-In-Filtered-Set`**. To znamená, že tohoto uživatele lze zneužít k provedení útoku **DCSync**.

## Enumerace ACL pomocí BloodHoundu

Nyní, když jsme celou útočnou cestu (attack path) prošli manuálně pomocí PowerView a vestavěných cmdletů, podívejme se, o kolik snazší by bylo její identifikování pomocí vizuálního nástroje BloodHound.

Nahrajeme data shromážděná pomocí ingestoru SharpHound do BloodHoundu. Poté nastavíme uživatele `wley` jako náš výchozí bod (_Starting Node_), přejdeme na záložku _Node Info_ a sjedeme dolů k sekci _Outbound Control Rights_. Tato možnost nám ukazuje objekty, které ovládáme přímo, prostřednictvím členství ve skupinách, a také celkový počet objektů, které bychom mohli ovládnout přes tranzitivní cesty útoků ACL (_Transitive Object Control_).

Pokud klikneme na číslo `1` vedle položky _First Degree Object Control_, uvidíme první sadu práv, kterou jsme enumerovali: hranu `ForceChangePassword` směřující k uživateli `damundsen`. Pokud klikneme pravým tlačítkem na spojnici mezi dvěma objekty a vybereme možnost **Help**, zobrazí se nám podrobná nápověda pro zneužití tohoto konkrétního ACE pravidla, včetně:

- Bližších informací o konkrétním právu, nástrojích a příkazech pro realizaci útoku.
    
- Operačních bezpečnostních aspektů (Opsec).
    
- Externích referencí.
    

Pokud klikneme na číslo `16` vedle _Transitive Object Control_, BloodHound nám vykreslí kompletní graf cesty, kterou jsme předtím složitě vyhledávali ručně. Odtud pak můžeme jednoduše využít nápovědu pro každou hranu (edge) a zjistit ideální postup pro provedení jednotlivých kroků útoku. Nakonec můžeme pomocí předpřipravených dotazů (_Pre-built queries_) v BloodHoundu potvrdit, že uživatel `adunn` má skutečně DCSync práva.

# 📝 Rychlé poznámky a přehled příkazů

### 1. Cílené vyhledávání ACL (PowerView)

- **Získání SID uživatele:**
    
    PowerShell
    
    ```
    $sid = Convert-NameToSid wley
    ```
    
- **Vyhledání objektů, nad kterými má uživatel práva (s překladem GUID):**
    
    PowerShell
    
    ```
    Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid}
    ```
    
- **Kontrola vnořeného členství skupiny:**
    
    PowerShell
    
    ```
    Get-DomainGroup -Identity "JmenoSkupiny" | select memberof
    ```
    

### 2. Reverzní vyhledávání názvu práva podle GUID (Vestavěné AD moduly)

PowerShell

```
$guid = "00299570-246d-11d0-a768-00aa006e0529"
Get-ADObject -SearchBase "CN=Extended-Rights,$((Get-ADRootDSE).ConfigurationNamingContext)" -Filter {ObjectClass -like 'ControlAccessRight'} -Properties * | Select Name,DisplayName,rightsGuid | ?{$_.rightsGuid -eq $guid} | fl
```

### 3. Vyhledávání ACL bez externích nástrojů (Vestavěný PowerShell)

PowerShell

```
# 1. Vytvoření seznamu uživatelů
Get-ADUser -Filter * | Select-Object -ExpandProperty SamAccountName > ad_users.txt

# 2. Prohledání práv pro konkrétního útočného uživatele
foreach($line in [System.IO.File]::ReadLines("C:\Cesta\ad_users.txt")) {
    Get-Acl "AD:\$(Get-ADUser $line)" | Select-Object Path -ExpandProperty Access | Where-Object {$_.IdentityReference -match 'DOMENA\\uzivatel'}
}
```

### 4. Shrnutí nalezené útočné cesty (Attack Path)

$$\text{wley} \xrightarrow{\text{ForceChangePassword}} \text{damundsen} \xrightarrow{\text{GenericWrite}} \text{Help Desk Level 1} \xrightarrow{\text{Nested In}} \text{Information Technology} \xrightarrow{\text{GenericAll}} \text{adunn} \xrightarrow{\text{DCSync}} \text{Domain Controller}$$

- **`User-Force-Change-Password`** over `damundsen` umožní resetovat heslo tohoto uživatele.
    
- **`GenericWrite`** over `Help Desk Level 1` umožní uživateli `damundsen` přidat libovolný účet do této skupiny.
    
- **`Information Technology`** v sobě obsahuje skupinu `Help Desk Level 1`, čímž dochází k dědičnosti práv.
    
- **`GenericAll`** over `adunn` dává skupině plnou kontrolu nad tímto uživatelem (lze provést reset hesla či Kerberoasting).
    
- **`DS-Replication-Get-Changes`** (DCSync) opravňuje uživatele `adunn` k replikaci doménových tajemství a dumpování hashů přímo z Domain Controlleru.