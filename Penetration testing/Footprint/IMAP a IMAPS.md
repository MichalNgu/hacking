# IMAP / IMAPS (Internet Message Access Protocol) – Porty 143, 993

IMAP je **TCP protokol pro práci s e-maily přímo na serveru**.  
Na rozdíl od POP3:

- e-maily zůstávají na serveru
- podporuje složky
- synchronizuje stav zpráv mezi zařízeními

Používá **tagy** (`A1`, `A2`), které spojují příkaz s odpovědí serveru.

---

# IMAP Porty

|Port|Účel|
|---|---|
|143|IMAP bez šifrování / STARTTLS|
|993|IMAPS přes SSL/TLS|

---

# Enumerace

Cíl:

- zjistit mail server
- zjistit uživatele
- získat e-maily
- najít citlivé informace

---

# Banner Grabbing

## IMAP (143)

```
nc -nv TARGET 143
```

Příklad:

```
* OK Dovecot ready
```

---

## IMAPS (993)

```
openssl s_client \
-connect TARGET:993
```

---

# IMAP Commands

IMAP vyžaduje před každým příkazem tag:

```
A1 LOGIN user password
```

---

|Příkaz|Funkce|
|---|---|
|`A1 LOGIN user pass`|Přihlášení|
|`A2 LIST "" "*"`|Výpis složek|
|`A3 SELECT INBOX`|Otevření schránky|
|`A4 EXAMINE INBOX`|Pouze čtení|
|`A5 FETCH 1 BODY[]`|Čtení e-mailu|
|`A6 LOGOUT`|Odhlášení|

---

# Práce s e-maily

Výpis složek:

```
A1 LIST "" "*"
```

Hledat:

```
Backup
Passwords
Shared
Archive
```

Výběr schránky:

```
A2 SELECT INBOX
```

Čtení zpráv:

```
A3 FETCH 1 BODY[]
```

Hledat:

- hesla
- VPN konfigurace
- interní informace
- přístupové údaje

---

# IMAP Útoky

---

# 1. Exploit starých verzí

Nejdříve zjisti verzi:

```
nmap -sV -p143,993 TARGET
```

Hledání exploitů:

```
searchsploit imap
```

Možné dopady:

- RCE
- buffer overflow
- získání shellu

---

# 2. IMAP Injection

Chyba ve webové aplikaci komunikující s IMAP serverem.

Princip:

```
Web aplikace
      |
      |
 IMAP Server
```

Špatná validace vstupu umožní vložit vlastní IMAP příkazy.

Dopad:

- obejití autentizace
- přístup k účtům

---

# 3. Log Poisoning (LFI → RCE)

Scénář:

```
IMAP Login
    |
    |
Malicious Input
    |
    |
Mail Log
    |
    |
LFI
    |
    |
RCE
```

Princip:

1. Vložíš kód do logu.
2. Server ho uloží.
3. LFI načte log.
4. Web vykoná kód.

---

# 4. Password Spraying

Místo:

```
1000 hesel → 1 účet
```

zkusíš:

```
1 heslo → 100 účtů
```

Příklad:

```
crackmapexec imap TARGET \
-u users.txt \
-p 'Password123!'
```

Hledáš:

- recyklovaná hesla
- slabé účty

---

# 5. MITM / SSL Stripping

Problém:

- port 143
- STARTTLS není vynuceno

Princip:

```
Client
 |
 | STARTTLS odstraněno
 |
Attacker
 |
 |
IMAP Server
```

Dopad:

- plaintext hesla
- čtení e-mailů

---

# 6. ACL Abuse

IMAP podporuje oprávnění na složky.

Špatná konfigurace může umožnit:

- přístup k cizím mailboxům
- čtení administrátorských e-mailů

---

# IMAP Pentest Checklist

☐ Port 143/993 otevřený  
☐ Banner grabbing  
☐ Verze serveru  
☐ TLS konfigurace  
☐ Login test  
☐ Výpis složek  
☐ Citlivé e-maily  
☐ Slabá hesla  
☐ Password reuse  
☐ ACL oprávnění

---

# HTB / CPTS IMAP Workflow

```
Nmap
 ↓
Port 143/993
 ↓
Banner
 ↓
IMAP Login
 ↓
Enumerace mailboxů
 ↓
Search Emails
 ↓
Find Credentials
 ↓
Reuse Password
 ↓
SSH / SMB / Web Access
```

---

# Nejčastější IMAP nálezy

|Finding|Dopad|
|---|---|
|Weak credentials|Převzetí účtu|
|Plaintext IMAP|Únik hesel|
|Exposed emails|Únik informací|
|Starý IMAP server|Exploit/RCE|
|Špatné ACL|Přístup k cizí poště|