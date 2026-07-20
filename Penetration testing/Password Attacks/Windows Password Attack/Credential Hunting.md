# 🔍 Credential Hunting

---

# 1. Strategie vyhledávání

Před samotným hledáním je důležité zaměřit se na místa, kde uživatelé a administrátoři typicky ukládají přihlašovací údaje.

---

## Typické zdroje credentialů

|Aktivita|Hledat|
|---|---|
|Správa databází|`dbpassword`, `connectionstring`, databázové konfigurace|
|Skriptování|`.ps1`, `.bat`, `.vbs`|
|Konfigurace služeb|`.config`, `.xml`, `.ini`|
|SSH / VPN|SSH klíče, `.ovpn`, `.ppk`|
|Automatizace|uložené účty a tokeny|

---

# Klíčová slova

Při hledání v souborech:

```text
password
passphrase
login
creds
pwd
user
account
credential
dbcredential
config
secret
token
key
```

---

# 2. Nástroje pro Credential Hunting

---

# 🛠️ LaZagne

Automatizovaný nástroj pro získávání uložených credentialů z aplikací.

Podporuje například:

- Chrome / Edge
    
- Firefox
    
- Outlook
    
- Skype
    
- WinSCP
    
- OpenVPN
    
- další sysadmin nástroje
    

---

## Spuštění všech modulů

```cmd
LaZagne.exe all
```

---

Typické nálezy:

```text
Browser passwords
Saved VPN credentials
SSH credentials
Application passwords
```

---

# 🛠️ findstr (Windows Built-in)

Pokud není možné použít externí nástroje, lze použít vestavěné vyhledávání.

---

## Hledání hesel v souborech

```cmd
findstr /S /I /M /C:"password" *.txt *.ini *.config *.xml *.ps1
```

---

Parametry:

|Parametr|Funkce|
|---|---|
|`/S`|Prohledává podsložky|
|`/I`|Ignoruje velikost písmen|
|`/M`|Vypíše pouze název souboru|
|`/C`|Hledá přesný řetězec|

---

# 3. Nejčastější místa s credentialy

---

# 1️⃣ Webové prohlížeče

Uživatelé často ukládají:

- hesla
    
- cookies
    
- session data
    

Typicky:

```text
Chrome
Edge
Firefox
```

Credentialy jsou chráněné pomocí:

```text
DPAPI
```

---

# 2️⃣ Konfigurační soubory

Časté cíle:

```text
web.config
app.config
unattend.xml
*.ini
*.xml
```

---

## Unattend.xml

Při automatizované instalaci Windows může obsahovat:

- lokální administrátorský účet
    
- instalační hesla
    

---

# 3️⃣ SYSVOL a Group Policy

V Active Directory:

```text
\\DOMAIN\SYSVOL
```

Kontrolovat:

- logon skripty
    
- GPO konfigurace
    
- administrátorské skripty
    

Mohou obsahovat:

- servisní účty
    
- uložená hesla
    
- příkazy pro automatizaci
    

---

# 4️⃣ KeePass databáze

Soubory:

```text
.kdbx
```

Obsahují:

- hesla
    
- API klíče
    
- poznámky
    
- přístupy k serverům
    

Workflow:

```text
.kdbx soubor
      |
      ▼
Offline crack master hesla
      |
      ▼
Přístup k uloženým credentialům
```

---

# 5️⃣ Uživatelské soubory

Časté nálezy:

```text
password.txt
passwords.xlsx
accounts.xlsx
credentials.txt
private_key.pem
id_rsa
```

---

# 4. Užitečné příkazy

## Rekurzivní hledání souborů

```cmd
dir /S /B *password*
```

---

## Hledání konfigurací

```cmd
dir /S /B *.config *.xml *.ini
```

---

## Hledání SSH klíčů

```cmd
dir /S /B id_rsa *.pem *.ppk
```

---

# 🧰 Nástroje

|Nástroj|Použití|
|---|---|
|LaZagne|Automatický credential dump|
|findstr|Hledání v souborech|
|Mimikatz|Windows credential extraction|
|KeePass|Databáze hesel|
|PowerShell|Automatizované hledání|

---

# Credential Hunting Workflow

```text
Initial Access
      |
      ▼
Enumerace systému
      |
      ▼
Hledání konfigurací
      |
      ▼
Browser / App Credentials
      |
      ▼
SSH / VPN Keys
      |
      ▼
Další účty
      |
      ▼
Privilege Escalation
```
