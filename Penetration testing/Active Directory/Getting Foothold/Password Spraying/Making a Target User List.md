## 🕵️‍♂️ 1. Anonymní sběr seznamu (Bez přihlašovacích údajů)

Pokud nemáš žádné uživatelské jméno ani heslo, spoléháš se na konfigurační chyby infrastruktury.

### A) SMB NULL Session

Umožňuje vytáhnout kompletní seznam uživatelů z doménového kontroleru bez autentizace. Výstupy je nutné pomocí Linux utilit (`grep`, `cut`) vyčistit, abychom získali čistá uživatelská jména.

- **Přes enum4linux:**
    
    Bash
    
    ```
    enum4linux -U 172.16.5.5 | grep "user:" | cut -f2 -d"[" | cut -f1 -d"]" > users.txt
    ```
    
- **Přes CrackMapExec / NetExec:** Výhoda CME spočívá v tom, že ti ukáže i hodnotu `badpwdcount` (počet špatně zadaných hesel). Pokud vidíš, že účet má již 4 špatné pokusy a limit je 5, **vymaž ho ze svého seznamu**, abys ho neuzamkl!
    
    Bash
    
    ```
    crackmapexec smb 172.16.5.5 --users
    ```
    

### B) LDAP Anonymous Bind

Pokud správce povolil anonymní dotazy do adresářových služeb LDAP, můžeš vytáhnout atribut `sAMAccountName` (přihlašovací jméno) všech objektů typu `user`.

- **Nástroj windapsearch:**
    
    Bash
    
    ```
    ./windapsearch.py --dc-ip 172.16.5.5 -u "" -U | grep "userPrincipalName:" | cut -f2 -d" " | cut -f1 -d"@" > users.txt
    ```
    

## ⚡ 2. Validace uživatelů přes Kerberos (Kerbrute)

Pokud jsou SMB NULL session i anonymní LDAP zakázány, nastupuje **Kerbrute**. Tento nástroj nepotřebuje heslo. Využívá mechanismus _Kerberos Pre-Authentication_.

Posílá dotazy na lístky (TGT) na doménový kontroler (port 88 UDP).

- Pokud KDC odpoví chybu `PRINCIPAL UNKNOWN`, uživatel neexistuje.
    
- Pokud si KDC vyžádá předběžné ověření (Pre-Authentication), **uživatel existuje** a Kerbrute ho označí za platného.
    

Bash

```
kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 /opt/jsmith.txt
```

> 🛡️ **OPSEC výhoda:** Fáze vyhledávání uživatelů (`userenum`) **negeneruje** v logách Windows selhání přihlášení (Event ID 4625) a **nikdy nezpůsobí uzamčení účtu**. Generuje však Event ID 4768 (žádost o TGT lístek), což mohou pozorní obránci odhalit, pokud zaznamenají tisíce těchto požadavků během několika sekund.

## 🔐 3. Sběr seznamu s platnými údaji (Credentialed)

Jakmile získáš jakýkoliv přístup do domény (např. skrze LLMNR/NBT-NS poisoning z předchozích kapitol), můžeš se jako platný uživatel oficiálně zeptat Active Directory na seznam všech ostatních kolegů:

Bash

```
crackmapexec smb 172.16.5.5 -u htb-student -p 'Academy_student_AD!' --users
```

## 📝 Zlatá pravidla administrace útoku (Logging)

Při provádění Password Spraying útoku si **vždy** veď striktní tabulku nebo logovací soubor. Pokud klient (nebo jeho SOC tým) detekuje anomálii, musíš být schopen okamžitě doložit, co přesně jsi dělal.

Zaznamenávej tyto hodnoty:

1. **Targeted accounts:** Která uživatelská jména jsi testoval.
    
2. **Domain Controller:** Proti které IP adrese / serveru útok směřoval.
    
3. **Date & Time:** Přesný čas spuštění (důležité pro ověření 30minutového okna pro reset čítače).
    
4. **Password attempted:** Jaké konkrétní heslo bylo testováno.