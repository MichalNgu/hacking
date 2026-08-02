
## 1. Pre-Engagement (Příprava)

### Cíl:

Připravit test, pochopit zadání a definovat pravidla.

### Co řeším:

- Kdo je klient
    
- Co se bude testovat (scope)
    
- Co se nesmí testovat (out of scope)
    
- Jaký typ testu provádím:
    
    - Black box → nemám informace
        
    - Grey box → mám částečné informace
        
    - White box → mám kompletní informace
        

### Dokumenty:

- Rules of Engagement (RoE)
    
- Scope dokument
    
- NDA
    
- Testovací plán
    

### Otázky před testem:

- Jaké IP adresy/domény jsou v rozsahu?
    
- Je povolen exploit?
    
- Je povolen DoS?
    
- Jaké jsou kontaktní osoby?
    
- Kdy může test probíhat?
    

---

# 2. Information Gathering (Recon)

## Cíl:

Zjistit co nejvíce informací o cíli bez útoku.

---

# Network Enumeration

## Nmap

### Co zjišťuji:

- Otevřené porty
    
- Běžící služby
    
- Verze služeb
    
- Operační systém
    

### Postup:

1. Najdu živé hosty
    

```
nmap -sn target
```

2. Sken portů
    

```
nmap target
```

3. Kompletní sken
    

```
nmap -sC -sV -p- target
```

Výstup:

```
22/tcp SSH
80/tcp HTTP
445/tcp SMB
3389/tcp RDP
```

Pak řeším jednotlivé služby.

---

# Footprinting

## Cíl:

Zjistit informace o firmě a infrastruktuře.

Hledám:

- Domény
    
- Subdomény
    
- DNS záznamy
    
- Technologie
    
- Zaměstnance
    
- Email adresy
    

Nástroje:

- whois
    
- dig
    
- nslookup
    
- Google dorking
    
- Shodan
    

---

# Web Information Gathering

## Cíl:

Zjistit jak funguje webová aplikace.

Kontroluji:

- Technologie:
    
    - Apache
        
    - Nginx
        
    - PHP
        
    - WordPress
        
    - Laravel
        
- Složky:
    
    - /admin
        
    - /backup
        
    - /uploads
        
- Soubory:
    
    - robots.txt
        
    - sitemap.xml
        
    - .git
        

Nástroje:

- WhatWeb
    
- Wappalyzer
    
- ffuf
    
- Burp Suite
    

---

# 3. Vulnerability Assessment

## Cíl:

Najít možné zranitelnosti.

Postup:

1. Zjištění verze služby
    

Například:

```
Apache 2.4.49
```

2. Hledání CVE
    

```
searchsploit apache 2.4.49
```

3. Ověření zranitelnosti
    

Nejdříve:

- pochopit chybu
    
- zjistit dopad
    
- najít exploit
    

---

# 4. Exploitation

## Cíl:

Využít nalezenou chybu.

---

# Shells & Payloads

Po exploitu potřebuji shell.

Typy:

## Reverse shell

Cíl se připojí zpět ke mně.

```
Target ---> Attacker
```

## Bind shell

Já se připojuji k cíli.

```
Attacker ---> Target
```

---

# File Transfers

Používám při přenosu:

- exploitů
    
- nástrojů
    
- payloadů
    

Metody:

Linux:

- wget
    
- curl
    
- nc
    

Windows:

- certutil
    
- PowerShell
    
- SMB
    

---

# Password Attacks

## Cíl:

Získat přístup pomocí hesel.

Typy:

## Brute force

Zkoušení všech možností.

## Dictionary attack

Použití wordlistu.

Například:

```
rockyou.txt
```

## Password spraying

Jedno heslo proti hodně uživatelům.

Příklad:

```
Winter2025!
 |
 + user1
 + user2
 + user3
```

Nástroje:

- Hydra
    
- CrackMapExec
    
- John
    
- Hashcat
    

---

# Common Services Attacks

## SSH

Kontrola:

- slabé heslo
    
- špatná konfigurace
    

---

## SMB

Kontrola:

- anonymní přístup
    
- sdílené soubory
    
- NTLM útoky
    

Nástroje:

- smbclient
    
- enum4linux
    
- crackmapexec
    

---

## FTP

Kontrola:

- anonymous login
    
- špatná konfigurace
    

---

# Web Exploitation

## Burp Suite

Použití:

- zachycení requestů
    
- úprava parametrů
    
- testování vstupů
    

---

# SQL Injection

## Princip:

Aplikace vloží uživatelský vstup přímo do SQL dotazu.

Normální:

```
SELECT * FROM users WHERE username='admin'
```

Nebezpečné:

```
admin' OR 1=1--
```

Může vést k:

- bypass loginu
    
- čtení databáze
    
- změně dat
    

Nástroj:

- SQLMap
    

---

# XSS

## Cíl:

Spustit JavaScript u oběti.

Typy:

Stored:

- uložený v databázi
    

Reflected:

- v URL/requestu
    

DOM:

- přes JavaScript
    

Dopad:

- krádež session
    
- phishing
    
- změna stránky
    

---

# File Inclusion

## LFI

Local File Inclusion

Čtení lokálních souborů:

```
/etc/passwd
```

---

## RFI

Remote File Inclusion

Načtení vzdáleného souboru.

---

# File Upload Attacks

Kontroluji:

- zda jde nahrát škodlivý soubor
    
- kontrolu přípony
    
- MIME kontrolu
    

Riziko:

Upload webshellu.

---

# Command Injection

## Cíl:

Spustit příkazy systému přes aplikaci.

Například:

Aplikace:

```
ping IP
```

Špatně zabezpečené:

```
127.0.0.1; whoami
```

---

# 5. Post Exploitation

## Cíl:

Po získání shellu zjistit možnosti.

---

# Enumeration po získání přístupu

Zjišťuji:

- uživatele
    
- práva
    
- systém
    
- síť
    
- hesla
    

Linux:

```
whoami
id
uname -a
sudo -l
```

Windows:

```
whoami
systeminfo
net user
```

---

# Privilege Escalation

## Linux

Hledám:

- sudo práva
    
- SUID soubory
    
- cron joby
    
- špatná oprávnění
    
- hesla v souborech
    

## Windows

Hledám:

- služby
    
- registry
    
- scheduled tasks
    
- uložená hesla
    
- tokeny
    

Nástroje:

- LinPEAS
    
- WinPEAS
    

---

# 6. Lateral Movement

## Cíl:

Pohyb v interní síti.

Používá se:

- ukradené účty
    
- hesla
    
- hash
    
- Kerberos tikety
    

---

# Active Directory

## Enumeration

Zjistit:

- doménu
    
- uživatele
    
- skupiny
    
- počítače
    

Nástroje:

- BloodHound
    
- PowerView
    
- LDAP queries
    

---

# AD útoky

## Kerberoasting

Získání service hashů.

## AS-REP Roasting

Útok na účty bez Kerberos pre-auth.

## Pass The Hash

Použití NTLM hashe místo hesla.

## NTLM Relay

Přesměrování autentizace.

---

# 7. Proof of Concept

## Cíl:

Dokázat zranitelnost.

Obsah:

- Popis problému
    
- Kroky reprodukce
    
- Screenshoty
    
- Exploit
    
- Dopad
    
- Severity
    

---

# 8. Post Engagement (Report)

## Cíl:

Předat výsledky klientovi.

Report obsahuje:

## Executive Summary

Pro management:

- co bylo nalezeno
    
- jak velké je riziko
    

## Technical Findings

Každá chyba:

- Název
    
- Popis
    
- Impact
    
- Evidence
    
- Remediation
    

## CVSS

Hodnocení:

- Low
    
- Medium
    
- High
    
- Critical
    

# Kompletní postup pentestu:

Recon  
↓  
Enumeration  
↓  
Vulnerability Assessment  
↓  
Exploitation  
↓  
Shell  
↓  
Post Exploitation  
↓  
Privilege Escalation  
↓  
Lateral Movement  
↓  
Proof of Concept  
↓  
Report