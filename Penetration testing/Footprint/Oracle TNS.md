# Oracle TNS (Transparent Network Substrate) – Port 1521

Oracle TNS je **síťový protokol používaný Oracle databázemi** pro komunikaci mezi klientem a databázovým serverem.

Proces připojení:

```
Client
  |
  | TCP 1521
  ↓
Oracle Listener
  |
  ↓
Database Instance
```

Klient potřebuje znát:

- **SID (System ID)**
- nebo **Service Name**

Bez správného SID se k databázi nepřipojí.

---

# Enumerace

Cíl:

- zjistit verzi Oracle
- najít SID
- najít účty
- získat přístup

---

# Nmap Scan

Zjištění verze:

```
nmap -p1521 \
--script oracle-tns-version \
TARGET
```

---

# SID Enumeration

SID je název Oracle instance.

## ODAT

Hádání SID:

```
odat sidguesser \
-s TARGET
```

Příklady:

```
xe
orcl
prod
test
```

---

# Password Guessing

Pomocí ODAT:

```
odat passwordguesser \
-s TARGET \
-d SID \
--accounts-file accounts.txt
```

---

# Metasploit

```
use auxiliary/admin/oracle/oracle_login
```

---

# Připojení do Oracle

## SQL*Plus

Běžný uživatel:

```
sqlplus user/password@TARGET/SID
```

SYSDBA:

```
sqlplus sys/password@TARGET/SID as sysdba
```

---

# Základní Oracle příkazy

Verze:

```
SELECT * FROM v$version;
```

Aktuální uživatel:

```
SELECT user FROM dual;
```

Práva:

```
SELECT * FROM user_role_privs;
```

Uživatelé:

```
SELECT username FROM all_users;
```

---

# Oracle Útoky

---

# 1. RCE přes DBMS_SCHEDULER

Pokud máš administrátorská práva:

Oracle může spouštět plánované úlohy.

Princip:

```
SQL Access
    |
    ↓
DBMS_SCHEDULER
    |
    ↓
OS Command Execution
```

Možný dopad:

- spuštění příkazů
- reverse shell

---

# 2. Čtení souborů (UTL_FILE)

Oracle může pracovat se soubory systému.

Použití:

- konfigurace
- hesla
- citlivé soubory

Příklad:

```
odat utlfile \
-s TARGET \
-d SID \
-u user \
-p password
```

---

# 3. Extrakce Hashů

Oracle ukládá informace o uživatelích v systémových tabulkách.

Příklad:

```
SELECT name, spare4
FROM sys.user$;
```

Hashe:

- offline crackování
- Hashcat

---

# 4. TNS Poisoning

Chybná konfigurace Oracle Listeneru.

Princip:

```
Attacker
   |
   |
Fake Listener Information
   |
   |
Oracle Listener
```

Možný dopad:

- únik autentizačních údajů
- manipulace s připojením

Nástroj:

```
Metasploit:
auxiliary/admin/oracle/tns_poison
```

---

# Hledání citlivých dat

Tabulky:

```
SELECT table_name
FROM all_tables
WHERE table_name LIKE '%PASS%';
```

Hledat:

```
PASSWORD
USER
TOKEN
SECRET
KEY
```

---

# Oracle Privilege Check

Zajímavé role:

|Role|Význam|
|---|---|
|SYSDBA|Kompletní administrace|
|DBA|Správa databáze|
|CREATE SESSION|Přihlášení|
|EXECUTE|Spouštění funkcí|

---

# Oracle Pentest Checklist

☐ Port 1521 otevřený  
☐ Verze Oracle  
☐ SID enumeration  
☐ Default credentials  
☐ Slabá hesla  
☐ SYSDBA přístup  
☐ Uživatelé a role  
☐ Čtení souborů  
☐ DBMS_SCHEDULER práva  
☐ Listener konfigurace

---

# HTB / CPTS Oracle Workflow

```
Nmap
 ↓
Port 1521
 ↓
Oracle Version
 ↓
Find SID
 ↓
Login Testing
 ↓
Enumerate Users/Roles
 ↓
Privilege Check
 ↓
File Access / RCE
 ↓
Database Compromise
```

---

# Nejčastější Oracle nálezy

|Finding|Dopad|
|---|---|
|Default credentials|Přístup do DB|
|SYSDBA access|Kompletní kontrola|
|Weak SID security|Snadnější útok|
|File privileges|Únik dat|
|Starý Listener|Možné exploity|