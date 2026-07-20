# 📧 Email Services (SMTP / POP3 / IMAP)

Emailové služby jsou častý vstupní bod během externího i interního pentestu. Cílem je zjistit:

- jaký mail server organizace používá,
    
- zda lze enumerovat uživatele,
    
- zda existují slabé účty,
    
- zda není server špatně nakonfigurovaný,
    
- zda neobsahuje citlivé informace.
    

---

# 🔎 1. Enumeration

## Zjištění mail serveru přes DNS

Nejdříve zjistím MX záznamy domény.

```bash
host -t MX domain.com
```

nebo:

```bash
dig MX domain.com
```

Sleduji:

|Výsledek|Význam|
|---|---|
|google.com|Google Workspace|
|outlook.com|Microsoft 365|
|mail.domain.com|Vlastní mail server|

---

# 🔍 2. Port Scanning

Standardní porty:

|Port|Služba|
|---|---|
|25|SMTP|
|587|SMTP Submission|
|110|POP3|
|143|IMAP|
|993|IMAPS|
|995|POP3S|

Nmap:

```bash
nmap -sV -sC -p 25,110,143,587,993,995 <IP>
```

Zajímá mě:

- verze SMTP serveru,
    
- banner,
    
- podporované funkce,
    
- TLS konfigurace.
    

---

# 🏷️ 3. SMTP Banner Grabbing

SMTP často prozradí software a verzi.

```bash
nc -nv <IP> 25
```

Příklad:

```
220 mail.domain.com ESMTP Postfix
```

Informace:

- Postfix
    
- Exchange
    
- Sendmail
    
- Exim
    

Následně hledám známé zranitelnosti podle verze.

---

# 👤 4. User Enumeration

Cíl:

Zjistit platné uživatele před password sprayingem.

---

## VRFY

Ověření existence uživatele:

```smtp
VRFY administrator
```

Pokud server odpoví:

```
250 administrator
```

uživatel existuje.

---

## EXPN

Rozbalení distribučních seznamů:

```smtp
EXPN support
```

---

## RCPT TO

Testování příjemce:

```smtp
MAIL FROM:test@test.com
RCPT TO:admin@domain.com
```

---

## Automatizace

```bash
smtp-user-enum \
-M VRFY \
-U users.txt \
-t <IP>
```

---

# ☁️ 5. Microsoft 365 Enumeration

U cloudových mailů nefunguje klasické SMTP enum.

Použití:

```bash
o365spray
```

Použití:

- validace uživatelů,
    
- password spraying.
    

---

# 🔑 6. Password Attacks

Pokud mám validní uživatele:

```
admin@domain.com
john@domain.com
```

zkouším slabá hesla.

---

## Password Spraying

Preferovaná metoda:

```
Users:
john
admin
backup

Password:
Winter2026!
```

Důvod:

- méně lockoutů,
    
- realističtější firemní scénář.
    

---

## Hydra

POP3:

```bash
hydra \
-L users.txt \
-p Password123 \
<IP> pop3
```

IMAP:

```bash
hydra \
-L users.txt \
-P passwords.txt \
<IP> imap
```

---

# 📤 7. SMTP Misconfiguration

## Open Relay

Kontroluji, zda server dovoluje posílat emaily bez autentizace.

Nmap:

```bash
nmap --script smtp-open-relay -p25 <IP>
```

---

## swaks test

```bash
swaks \
--from test@test.com \
--to user@domain.com \
--server <IP>
```

Pokud odejde mail bez autentizace:

→ Open Relay nalezen.

---

# 📥 8. Post-Compromise Email Hunting

Pokud získám účet:

Kontroluji:

- uložená hesla,
    
- VPN konfigurace,
    
- interní dokumenty,
    
- reset linky,
    
- přístupové údaje.
    

Hledaná slova:

```
password
passwd
secret
vpn
config
backup
credentials
```

---

# 🛠️ Nástroje

|Nástroj|Použití|
|---|---|
|Nmap|Service detection, SMTP checks|
|dig|MX enumeration|
|host|DNS lookup|
|smtp-user-enum|User enumeration|
|Hydra|Brute force / spraying|
|o365spray|Microsoft 365 enum|
|swaks|SMTP testing|
|MailSniper|Exchange/O365 hunting|

---

# 🧠 Pentester Checklist

✅ MX záznamy nalezeny  
✅ Mail server identifikován  
✅ Porty otestovány  
✅ Banner získán  
✅ SMTP user enumeration vyzkoušena  
✅ Password spraying proveden  
✅ Open Relay ověřen  
✅ Po získání účtu proveden email hunting