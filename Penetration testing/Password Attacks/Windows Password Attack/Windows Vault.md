# 🔐 Windows Vault / Credential Manager Cheat Sheet

---

# 1. Credential Manager (Windows Vault)

Windows Credential Manager ukládá uložené přihlašovací údaje do šifrovaných trezorů.

Každý uživatel má vlastní credential store.

---

## Umístění souborů

Lokální profil:

```text
%UserProfile%\AppData\Local\Microsoft\Credentials\
```

Roaming profil:

```text
%UserProfile%\AppData\Roaming\Microsoft\Credentials\
```

---

# Ochrana pomocí DPAPI

Windows používá:

```text
Data Protection API (DPAPI)
```

DPAPI chrání:

- uložená hesla
    
- síťové credentialy
    
- aplikační přístupy
    
- tokeny
    

---

# 2. Enumerace uložených credentialů

## cmdkey

Vestavěný Windows nástroj pro zobrazení uložených přihlašovacích údajů.

```cmd
cmdkey /list
```

---

Výstup:

```text
Target:
Type:
User:
```

---

# Co sledovat

|Položka|Význam|
|---|---|
|Target|Zdroj credentialu (server, IP, doména)|
|Type|Typ uloženého přístupu|
|Domain Password|Zajímavé doménové credentialy|
|Persistence|Platnost po restartu|

---

# 3. Použití uložených credentialů

Pokud systém obsahuje uložené přístupy, Windows je může použít automaticky.

---

## runas /savecred

Spuštění procesu pod uloženým účtem:

```cmd
runas /savecred /user:<DOMAIN>\<USER> cmd
```

---

Výsledek:

```text
Nové CMD pod uloženým uživatelem
```

---

# 4. Extrakce credentialů

---

# A) Mimikatz - Credential Manager

Pokud je uživatel aktivně přihlášen:

```cmd
mimikatz
```

---

Aktivace oprávnění:

```text
privilege::debug
```

---

Dump Credential Manager:

```text
sekurlsa::credman
```

---

Hledat:

```text
Password :
```

---

Možné nálezy:

- OneDrive hesla
    
- síťové disky
    
- uložené aplikace
    
- vzdálené služby
    

---

# B) DPAPI Offline Analysis

Pokud máš pouze soubory z disku:

```text
Credentials Folder
        |
        ▼
DPAPI Blob
        |
        ▼
Master Key
        |
        ▼
Decrypted Credential
```

Potřebné:

- DPAPI master keys
    
- uživatelský kontext
    
- registry data
    

---

# 5. Alternativní nástroje

---

## SharpDPAPI

C# nástroj pro práci s:

- DPAPI
    
- Credential Manager
    
- browser credentials
    

---

## LaZagne

Automatizovaný credential extractor.

Podporuje:

- Windows Vault
    
- browser passwords
    
- aplikace
    
- SSH
    
- VPN
    

---

## DonPAPI

Nástroj pro vzdálený sběr DPAPI credentialů v síti.

Použití:

- doménová prostředí
    
- více uživatelů
    
- vzdálená enumerace
    

---

# 🧰 Nástroje

|Nástroj|Použití|
|---|---|
|cmdkey|Enumerace uložených credentialů|
|Mimikatz|Credential Manager dump|
|SharpDPAPI|DPAPI analýza|
|LaZagne|Automatický credential hunting|
|DonPAPI|Remote DPAPI extraction|

---

# Credential Hunting Workflow

```text
User Access
      |
      ▼
cmdkey /list
      |
      ▼
Credential Store Discovery
      |
      ▼
DPAPI Analysis
      |
      ▼
Plaintext Credentials
      |
      ▼
Credential Reuse
```
