# POP3 / POP3S (Post Office Protocol) – Porty 110, 995

POP3 je **TCP protokol pro stahování e-mailů ze serveru ke klientovi**.  
Na rozdíl od IMAP funguje jednoduše:

```
Server
  |
  | stáhneš e-maily
  ↓
Client
```

Po stažení se spojení většinou ukončí.

---

# POP3 Porty

|Port|Účel|
|---|---|
|110|POP3 bez šifrování|
|995|POP3 přes SSL/TLS|

---

# Enumerace

Cíl:

- zjistit verzi serveru
- otestovat přístup
- získat e-maily
- najít citlivé informace

---

# Banner Grabbing

## POP3 (port 110)

```
nc -nv TARGET 110
```

Příklad:

```
+OK Dovecot ready
```

---

## POP3S (port 995)

```
openssl s_client \
-connect TARGET:995 \
-crlf
```

---

# POP3 Commands

Přihlášení:

```
USER username
PASS password
```

---

|Příkaz|Funkce|
|---|---|
|`STAT`|Počet zpráv a velikost|
|`LIST`|Seznam e-mailů|
|`RETR 1`|Stažení zprávy|
|`TOP 1 10`|Hlavička + prvních 10 řádků|
|`CAPA`|Podporované funkce|
|`DELE 1`|Smazání zprávy|
|`QUIT`|Ukončení|

---

# Získání e-mailů

Po přihlášení:

```
STAT
```

Výpis:

```
+OK 5 20480
```

Seznam:

```
LIST
```

Stažení:

```
RETR 1
```

Hledat:

- hesla
- VPN údaje
- interní informace
- konfigurace

---

# Útoky

---

# 1. Sniffing

Port 110 neposílá data šifrovaně.

Útočník může vidět:

```
USER admin
PASS password123
```

Wireshark filtr:

```
pop
```

nebo:

```
tcp.port == 110
```

---

# 2. Brute Force

POP3 používá pouze:

```
USER
PASS
```

Hydra:

```
hydra -l user \
-P passwords.txt \
pop3://TARGET
```

---

# 3. User Enumeration

Některé servery mohou prozradit existenci uživatele podle odpovědi nebo času.

Moderní servery většinou vrací stejnou odpověď:

```
+OK
```

---

# POP3 Pentest Checklist

☐ Port 110/995 otevřený  
☐ Banner grabbing  
☐ Verze serveru  
☐ TLS podpora (`CAPA`)  
☐ Slabá hesla  
☐ Přístup k mailboxům  
☐ Citlivé informace v e-mailech  
☐ Recyklované heslo pro SSH/SMB

---

# HTB / CPTS POP3 Workflow

```
Nmap
 ↓
Port 110/995
 ↓
Banner Grab
 ↓
Test Login
 ↓
Download Emails
 ↓
Find Credentials
 ↓
Reuse Password
 ↓
SSH/SMB/Web Access
```

---

# Nejčastější POP3 nálezy

|Finding|Dopad|
|---|---|
|Plaintext POP3|Únik hesel|
|Weak passwords|Převzetí účtu|
|Exposed emails|Únik informací|
|Password reuse|Přístup k dalším službám|
|Starý mail server|Možné CVE|